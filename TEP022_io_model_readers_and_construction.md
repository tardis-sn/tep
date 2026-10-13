# TEP022: Model readers, construction paths, and the I/O dependency boundary

## Summary

This TEP restructures how TARDIS reads external model files and turns them
into a `SimulationState`. Every model reader shall return a passive
per-format dataclass with `metadata` and `data` fields that follow one column
and unit convention. Every reader shall be reachable from the configuration.
The code that builds geometry, density, composition and radiation field
objects shall move out of `tardis.io` and sit next to the domain it builds,
as one code path shared by YAML configurations and CSVY models. After this
change, `tardis.io` shall import nothing from `tardis.model`, `tardis.plasma`,
`tardis.transport` or `tardis.radiation_field`.

## Status

**Discussion**

## Responsible

Andrew Fullard

## Pull requests and Issues

None yet. The first implementation step (step 1 below) is ready on a local
branch and will be opened as a pull request against `tardis-sn/tardis`.

## Description

`tardis/io` currently mixes file reading with object construction, and that
mixing causes several concrete problems.

**Readers have no common interface.**
- Readers return 3-tuples, 5-tuples, or one of six unrelated dataclasses
  (`CSVYData`, `ArtisData`, `STELLAModel`, `CMFGENModel`, `SNECOutput`,
  `ArepoData`).
- Only `STELLAModel` and `CMFGENModel` use a clean `metadata` + `data` shape,
  and neither is connected to the configuration.

**Units are inconsistent.**
- CSVY strips units to bare arrays, while ARTIS returns astropy Quantities.
- Velocity from a CSVY YAML header is a Quantity, but velocity from the CSV
  body of the same file is a bare array.

**Several readers are unreachable.**
- STELLA, the newer CMFGEN reader, SNEC, AREPO and Blondin cannot be selected
  from a configuration.
- `filetype: cmfgen_model` still routes to a deprecated parser that assigns
  units by column position.

**Dependencies run the wrong way.**
- `tardis.io.model.parse_*` imports `tardis.model`, `tardis.plasma`,
  `tardis.radiation_field` and `tardis.transport`, and `tardis.model.base`
  imports those parsers back. The resulting cycle is worked around with lazy
  imports and `TYPE_CHECKING` blocks.
- `tardis/io/atom_data/base.py` also imports `tardis.plasma.array_util`, which
  pulls in `tardis.transport`.

**Construction logic is duplicated, and the copies have drifted.** Geometry,
density, mass fractions, composition and radiation field each exist as a
`*_from_config` and a `*_from_csvy` function. The two copies no longer agree:
- CSVY models with an analytic density section produce N+1 densities for N
  shells. `SimulationState.density` then slices the first N, so every active
  shell receives its inner neighbour's density. For a branch85 model the
  first shell is a factor of about 160 too dense. The existing comparison
  test misses this because it compares to six decimal places and the values
  are of order 1e-13.
- CSVY isotopes skip the `model_isotope_time_0` checks that configuration
  isotopes receive.
- A CSV velocity column combined with a YAML analytic density crashes.

**There is further duplication.**
- Relative-path resolution is copied into six places.
- There are two `ConfigurationError` classes with different base classes.
- `custom_abundance.py` holds a fourth copy of the CSVY velocity and density
  logic and a third CSVY writer.
- The ARTIS structure file is read four times per run.

The goal of this TEP is a single, testable path from any supported model file
to a `SimulationState`:
- readers are pure I/O with one data convention;
- one set of construction functions serves both configuration and CSVY input;
- a dependency boundary that a test enforces.

The goal is not a new model format or new physics. Scientific results shall
change only where this TEP names a defect.

## Implementation

### Reader output

Each format shall keep its own dataclass with the existing shape:

```python
@dataclass(frozen=True)
class CMFGENModel:
    metadata: dict
    data: pd.DataFrame
```

The same shape shall be used for `SimpleAsciiDensityModel`,
`SimpleAsciiAbundanceModel`, `ArtisDensityModel`, `ArtisAbundanceModel`,
`CSVYModel`, `STELLAModel` and `CustomCompositionModel`. A `ModelFile`
protocol in `tardis/io/model/model_file.py` shall type them all. Every reader
shall follow these conventions:

- **Rows.** `data` holds one row per file row, in file order, as bare floats.
  Row 0 is the inner-boundary or core row. Readers shall not drop it.
- **Column names.** Physical columns use standard names: `velocity` (the
  outer boundary of the row's cell), `radius`, `density`, `electron_density`,
  `temperature` and `dilution_factor`. Composition columns are named by
  element symbol (`O`) or nuclide (`Ni56`) in their original case.
- **Units.** pandas cannot hold astropy Quantities. Every dimensional column
  shall therefore have a `metadata[f"{column}_unit"]` entry holding an astropy
  unit, as the current CMFGEN and STELLA readers already do. Scalars in
  `metadata`, such as `time_of_model` and `isotope_time`, are Quantities.
- **Optional metadata.** CSVY headers may add `velocity_section`,
  `density_section` and `uniform_abundances`. The ARTIS abundance reader adds
  an `elemental_includes_isotopes` flag.

The only code that turns a `ModelFile` into Quantities (converted to cgs) and
into TARDIS mass-fraction frames shall be three accessors in `model_file.py`:
- `get_boundary_quantity` returns all rows;
- `get_cell_quantity` drops row 0;
- `get_mass_fractions` drops row 0 and transposes into an `atomic_number`
  index, or an `(atomic_number, mass_number)` index for isotopes.

Dropping row 0 once, in these accessors, matches what every current path does.

Two registries keyed by `filetype` shall replace the `if`/`elif` dispatch in
`readers/base.py`: `STRUCTURE_READERS` and `COMPOSITION_READERS`. Each maps a
filetype to a `Callable[[Path], ModelFile]`. `filetype: cmfgen_model` shall
use `read_cmfgen_model`, after fixing three problems in it:
- it lowercases element columns (`c`, `o`, `mg`), which are then silently
  dropped;
- its time-header pattern accepts only decimal days;
- it does not skip `#` comment lines.

### Resolving inputs

`load_model_input(config)` in `tardis/io/model/load.py` shall return a
`ModelInput` with these fields:
- the structure and composition model files;
- the velocity, density and uniform-abundance sections, taken from either the
  YAML configuration or the CSVY header;
- the isotope reference time.

Each file shall be read once. Velocity-boundary overrides stay in
`config.model`, where users and tests already set them. ARTIS isotopes from
the structure file shall be used only when the composition file is also
ARTIS, which preserves the current behavior.

### Construction next to each domain

The configuration and CSVY copies of each construction function shall be
replaced by one module function next to the domain it builds:

| Module | Contents |
|---|---|
| `tardis/model/geometry/parse.py` | `parse_velocity`, `parse_geometry` |
| `tardis/model/matter/density.py` | analytic density profiles, time scaling, `parse_density` |
| `tardis/model/matter/parse.py` | `parse_composition`: uniform abundances, one clean-up and normalization step, one decay step |
| `tardis/radiation_field/parse.py` | `parse_radiation_field` |
| `tardis/transport/montecarlo/packet_source/config.py` | packet-source construction from configuration |

`SimulationState.from_config` shall handle both input types through
`load_model_input`, and `SimulationState.from_csvy` shall be removed.

### Converters to CSVY

SNEC output is a time series of hydrodynamic snapshots, and AREPO output is a
3D particle snapshot. Both shall connect to TARDIS by conversion to CSVY, not
as native filetypes; Blondin toy models shall connect the same way. A single
`write_csvy` function shall replace the three existing writers, writing
values at full precision and without row subsampling.

### Steps

Each step is one pull request. Steps 3–5 depend on step 2, and step 6 depends
on step 5.

1. **Errors, paths and schema.**
   - Merge the two `ConfigurationError` classes.
   - Resolve paths at all six sites with `Path(config.config_dirname) /
     filename`. Joining an absolute filename returns it unchanged, so the
     resolved paths do not change.
   - Add a `filetype` enum to the abundance schema, so that an unknown
     abundance filetype fails at validation instead of later with a
     `KeyError`.
   - *Implemented locally, not yet opened.*
2. **Science fixes in the existing code.** Each fix lands before any code
   moves and comes with a failing test that uses relative tolerances:
   - give CSVY analytic density N entries;
   - apply the isotope-time checks to CSVY input, raising
     `ConfigurationError` instead of failing an `assert`;
   - support a CSV velocity column with a YAML analytic density;
   - drop zero-sum isotope rows in the configuration path;
   - validate radiative temperatures read from files.
3. **Reader dataclasses and the CMFGEN switch.**
   - Add `model_file.py`, the per-format dataclasses and the registries.
   - Temporarily keep `readers/base.py` as an adapter, so construction code
     does not change in this step.
   - Delete the deprecated CMFGEN parser.
4. **Construction next to domains.**
   - Add `load_model_input` and the domain `parse` modules.
   - Remove `from_csvy` and the `tardis/io/model/parse_*` modules.
   - Move the pure-pandas `get_ground_state_multi_index` into
     `tardis/io/atom_data`.
   - Add a test that walks `tardis/io` with `ast` and fails on any import of
     `tardis.model`, `tardis.plasma`, `tardis.transport` or
     `tardis.radiation_field`.
5. **The CSVY writer.** Route `custom_abundance` and the Blondin converter
   through `write_csvy`.
6. **STELLA, AREPO and SNEC.** Add a `stella` filetype and the AREPO and SNEC
   converters. This step must not start until the scientific assumptions
   below are confirmed.
7. **Documentation.** Update the configuration filetype documentation, the
   external-model how-to notebooks, the API pages and the model walkthrough.

### Scientific assumptions to confirm before step 6

**STELLA**
- The file gives cell centres and outer edges. Boundary velocities shall
  assume homologous expansion: `v_edge = v_centre * r_edge / r_centre`.
- The model time shall be `r_centre / v_centre`, which must agree across
  cells.
- The first zone becomes the core row, because its inner edge is unknown.
- The header's `t_max`, which is measured from peak bolometric luminosity, is
  not the time since explosion.

**AREPO**
- Profiles are binned in radius, and the mass-weighted mean bin velocities may
  be NaN or non-monotonic.
- Boundaries shall be `r_edge / t_snapshot`; alternatively a `radius` column
  shall be written.
- Empty bins shall raise an error.

**SNEC**
- Whether the `radius` output is a cell centre or an edge must be confirmed
  from the SNEC source.
- Early snapshots are not homologous, so a `radius` column shall be written.
- Isotope profiles shall be interpolated by enclosed mass onto the snapshot
  cells.
- The isotope reference time is the explosion time.

## Tests and validation

- **Step 2 behavior changes.** Each fix starts with a test that fails on the
  current code. Tests shall use relative tolerances, so they cannot repeat the
  absolute-tolerance blind spot that hid the CSVY density shift. The CSVY
  analytic-density fix shall require each shell's density to equal the
  density from the equivalent YAML configuration, for branch85 and power-law
  profiles.
- **Steps 1, 3, 4 and 5.** These are behavior-preserving. Before step 2, the
  density, radiative temperature, dilution factor, abundances and isotopic
  number densities of the `SimulationState` shall be recorded for these
  models:
  - the very simple configuration;
  - the ARTIS configurations;
  - the CMFGEN configuration;
  - the CSVY test configurations.

  After every later step, these records shall match with
  `assert_allclose(rtol=1e-12)` and `pandas.testing.assert_frame_equal`.
- **CMFGEN switch.** The existing expectations for the deprecated CMFGEN
  reader (array shapes and listed values) shall be applied unchanged to the
  new reader.
- **Unit convention.** A structural test shall check that every reader
  provides a unit entry for each dimensional column.
- **Dependency boundary.** The import-boundary test shall fail before step 4
  and pass after it.
- **Every step.** Each step shall run:
  - the full test suite;
  - Ruff;
  - the regression-data tests via `--tardis-regression-data`;
  - a documentation build.

  Outside steps 2 and 6, any change to a regression reference stops the work
  for review.

## Backward compatibility

TARDIS has no public Python API. Removed functions, classes and module paths
shall therefore get no aliases or deprecation shims, and all in-repository
callers, tests and notebooks shall be updated in the same step. The
user-visible changes are:

- **Configuration validation.** Unknown abundance filetypes are rejected when
  the configuration is validated (step 1). Every filetype currently used in
  the repository, the documentation and the regression data is in the allowed
  set.
- **`config_dirname`.** Absolute paths now need `config_dirname` as well
  (step 1). `from_yaml` and `from_config_dict` always set it; a
  `Configuration` built directly from a dict must set it before files are
  read.
- **Scientific results.**
  - CSVY models with analytic density change (step 2), because the one-shell
    offset is removed.
  - CSVY isotope inputs now fail when `model_isotope_time_0` is not earlier
    than `time_explosion`.
- **`SimulationState.from_csvy`.** It is removed (step 4).
  `SimulationState.from_config` accepts configurations with `csvy_model`, and
  its `atom_data` argument becomes optional.
- **`filetype: cmfgen_model`.** Its input format and results are unchanged.
- **AREPO CSVY exports.** These change (step 6): velocities become true
  boundaries and values are written at full precision. The reference CSVY
  files used in tests shall be regenerated only after review.

## Alternatives

- **Native SNEC and AREPO filetypes.** The configuration would select a
  snapshot time or a profile type directly. This was rejected because it
  would add time-series and 3D-reduction parameters to the model schema;
  conversion to CSVY keeps the schema one-dimensional and gives users a file
  they can inspect.
- **One shared reader dataclass.** All readers would return a single
  `ModelFile` class instead of per-format classes. This was not chosen; the
  per-format classes follow the established `STELLAModel` and `CMFGENModel`
  pattern, and the shared `ModelFile` protocol still gives them one interface.
- **Readers that return Quantities converted to cgs.** This would not fit
  pandas, which cannot hold Quantities. Keeping bare values with explicit unit
  metadata, and converting in one set of accessors, makes units consistent
  without that limitation.
- **Classmethod constructors on domain classes**, such as
  `HomologousRadial1DGeometry.from_model_file`. This would put file-format
  knowledge into the domain classes. Module functions next to each domain
  keep those classes independent of input formats.
- **A new `tardis/model/builders` package.** This would duplicate the
  existing domain layout of `tardis/model` (`geometry/`, `matter/`) and the
  existing `tardis/radiation_field/` package.
- **Keeping the deprecated CMFGEN parser beside the new one.** This would
  keep the position-based unit assignment and two readers for one format.
- **A single pull request.** This would be hard to review, and it would mix
  behavior changes with behavior-preserving moves. Landing the science fixes
  first lets each move be checked at `rtol=1e-12`.

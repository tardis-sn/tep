TEPs: TARDIS Enhancement Proposals
----------------------------------

TEPs are used to address non-trivial enhancements that require much discussion
and thought. This is intended to mirror the long-standing Python Enhancement
Proposal process, but generally not quite as formally. A proposal goes
through various phases of consideration. Discussion is expected to
take place using existing mechanisms (github PRs), and
eventually a decision is made regarding whether the proposal should be
accepted, rejected, or modified.

The general process of a TEP starts with a PR to the TEP repository.
At this stage, the TEP should have a title and some description of the
proposed changes. We encourage an accompanying PR to the main TARDIS
repository with some code or pseudo-code that quickly showcases the proposed 
changes. Once the group has iterated over the documented and agreed to implement
the proposed changes the TEP is merged. If things change during the 
implementation the TEP should be amended.

New TEPs should be created using the ``TEP_template.md`` file in this repository.
Just fork the repository, copy ``TEP_template.md`` to ``TEPXXX.md`` and issue a
PR with that file once you've written it up.  Be sure to look through the PRs in
this repo first so that you choose an TEPXXX that reflects both those that are in
the repository *and* those that are under discussion.

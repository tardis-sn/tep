==============
 TEP Template
==============


This TEP template is a guideline of the sections that a TEP should
contain.  Extra sections may be added if appropriate, and unnecessary
sections may be noted as such.

The TEP should start with a title:

TEPXXX: Meaningful title
========================

Status
======

 **Discussion**

Responsible
===========

@yeganer

Description
===========

Git is a very powerful tool for managing projects. Currently we are not taking full advantage
of it. To faciliate navigating the commit history in the future, the following
rules for commit messages (taken from http://chris.beams.io/posts/git-commit/) should be followed:


* Separate subject from body with a blank line
* Limit the subject line to 50 characters
* Capitalize the subject line
* Do not end the subject line with a period
* Use the imperative mood in the subject line
* Wrap the body at 72 characters
* Use the body to explain what and why vs. how
* Reference related issues in the body

When following these guidelines it should be much easier to follow the changes especially as
someone who isn't familiar with the code.

In addition to the rules above the following should be avoided:

* Don't commit code that is not working. Always run the basic test suit and fix bugs or modify tests, if neccessary, immediately.
  If there are crashing commits in your branch, consider rewriting your history
  ( see https://git-scm.com/book/en/v2/Git-Tools-Rewriting-History for examples)
* Don't commit code with changes related to different issues/features. Use git add -p to selectively stage changes
  for commiting

Further best practice for git commit messages can be found here:
* http://chris.beams.io/posts/git-commit/
* http://who-t.blogspot.de/2009/12/on-commit-messages.html

Information about selectively commiting only some changes to a file can be found here:
* http://johnkary.net/blog/git-add-p-the-most-powerful-git-feature-youre-not-using-yet/
* http://nuclearsquid.com/writings/git-add/

Implementation
==============

Link to this TEP in the documentation for Contributors.

Backward compatibility
======================

We can't rewrite history, we can only improve the future.

Alternatives
============

The alternative to having rules for commit messages is the current state.

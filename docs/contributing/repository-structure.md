# Repository Structure
The structure of the JDA repository is heavily inspired from [A Successful Git Branching Model](http://nvie.com/posts/a-successful-git-branching-model/)


## Release Branch

This branch represents the latest stable release. If a bug is found that requires immediate handling, a [hotfix branch](#hotfix-branch) is to be based off of this branch
and merged back into it.

## Release Candidate Branch

The release candidate branches are forks of the [development branch](#development-branch) and are mainly used to prepare a release. The branch name is formatted as `release-x` where `x` would be the expected version.

**Based off:**
- development

**Merged into:**
- development
- release

## Hotfix Branch

These branches are used to fix critical issues on the stable release and are to be merged right into master and development.

If a release is in the process of being established and clashes versions, the version of the unmerged release is bumped.

**Based off:**
- release

**Merged into:**
- development
- release

## Development Branch

This is the core of the repository where all changes start. All Pull Requests are based off of this branch and merged back into it.
[Feature Branches](#feature-branch) should be forked from development and merged back into it.

**Merged into:**
- release

## Documentation Branch

This is branch dedicated to updating the documentation. Since we build every change on `development`, we don't want to do a ton of "Fix typo" builds. For this reason we have this documentation branch.

**Merged into:**
- development

## Feature Branch

A feature branch should be used to develop a new feature for the library and should only deal with the core of the feature. Feature branches should not
introduce breaking changes but may add deprecation.
The branch name is formatted as `feature/x` where `x` would be the feature topic.

**Based off:**
- development

**Merged into:**
- development

## Experimental Branch

An experimental branch changes core functionality of the library to a large extent and requires testing.
The branch name is formatted as `experimental/x` where `x` would be the experiment topic.

**Based off:**
- development

**Merged into:**
- development
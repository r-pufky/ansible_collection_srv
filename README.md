# Ansible Collection - r_pufky.srv

Documentation for the collection.

Pinned to `0.0.x` until all roles are migrated to ansible `2.18`, migrated from
private repository, and common functionality is pulled out to the collection
level; with documentation and template files fleshed out.

## Specific ignores until migration is done
* bump PATCH version `x` when committing a new migration. This will keep ALL
  unreleased incremental changes to `0.0.x`.
* NO PUBLISHING TO GALAXY until `1.0.0`; all roles with collection dependencies
  may experience fluctuating collection setup until `1.0.0` release.
* yamllint passes
* ansible-lint passes
* Export common role functionality to collection when importing role
* fill out and remove templated / skeleton files (all should be finished before
  `1.0.0`).
* Unit testing, integration testing required; with the ability to disable live
  API/file downloads for isolated testing.
* Settings are set to sane defaults.
* no .yamllint, .ansible-lint disable files; document accepted usage.
* [Documentation standardized](docs/README.md).

Those requiring the collection for `2.18` use should manually build and include
this collection until it is live on galaxy.

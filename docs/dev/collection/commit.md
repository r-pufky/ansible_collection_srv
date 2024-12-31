# Collection Commits
Prerequisite:
* [ansible environment](../environment/ansible.md)

## Ensure all TODOs are valid
``` bash
grep -ri todo
```

## Ensure linting follows [guidelines](../roles/linting.md)
``` bash
grep -ri yamllint  # yamllint
grep -ri noqa  # ansible-lint
```

## Lint returns clean
``` bash
yamllint .
ansible-lint .
```

[yamllint rules](https://yamllint.readthedocs.io/en/stable/)

[ansible-lint rules](https://ansible.readthedocs.io/projects/lint/rules/)

## Add collection ignore files
Add files that should not be included in the built collection; such as tests.

`0640 {USER}:{USER}` {COLLECTION}/galaxy.yml
``` yaml
build_ignore:
  - .gitignore
  - changelogs/.plugin-cache.yaml
```
[Reference](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections_distributing.html#ignoring-files-and-folders)

## Versioning
Per [Semantic Versioning v2](https://semver.org/)

* Bump version in [galaxy.yml](../../../galaxy.yml)
  * Major: new platform release (debian release) / breaking changes
  * Minor: added functionality / non-breaking changes
  * Point: bug fixes / updates
* Tag commit with galaxy version per [galaxy.yml](../../../galaxy.yml)

Until collection is migrated publically Major will always be `0` as this
prevent galaxy inclusion (`1.0.0` is the minimum version).

## Push Galaxy Release

``` bash
ansible-galaxy collection build
ansible-galaxy collection publish
```
[Reference](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_creating.html)


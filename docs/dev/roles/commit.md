# Role Commits
Assumes ansible and collection [environment setup](../collection/setup.md).

## Ensure all TODOs are valid
``` bash
grep -ri todo
```

## Ensure linting follows [guidelines](file_format.md)
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

{COLLECTION}/galaxy.yml
``` yaml
build_ignore:
  - .gitignore
  - changelogs/.plugin-cache.yaml
```
[Reference](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections_distributing.html#ignoring-files-and-folders)

## Track Project Updates
* Always link to the latest release (or release page).
* Set alerts to check for new release (or push notifications) for each role.

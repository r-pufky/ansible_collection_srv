# Role Creation
Prerequisite:
* [ansible environment](../environment/ansible.md)

All roles are added as [submodules](submodules.md) in the collection repo.

Creation will overwrite contents of the specified location, destroying existing
git checkouts. Create the role with a temporary name and move created items
into the git checkout location.

``` bash
ansible-galaxy role init --init-path roles example
```
[Reference](https://goetzrieger.github.io/ansible-collections/5-creating-collections/#adding-content-adding-a-custom-role)

Standards for directory layout:
```
├─┬── defaults
│ └┬─ main
│  ├─ main.yml  # ansible specific role configuration (users, locations, etc)
│  ├─ *.yml  # service specific configuration; named by service or type
│  └─ ports.yml  # firewall definitions even if not used.
├──── handlers
├──── molecule  # unit testing
├─┬── meta
│ └┬─ main.yml  # galaxy metadata
│  └─ requirements.yml  # collection and roles required
├──── tasks
├──── templates
├──── vars
├──── LICENSE
└──── README.md
```

## `0640 {USER}:{USER}` meta/main.yml
``` yaml
---
galaxy_info:
  author: 'r_pufky'
  description: 'Role configuration'
  company: 'N/A'
  license: 'AGPL-3.0 License'
  min_ansible_version: '2.18.0'

  platforms:
    - name: 'Debian'
      versions:
        - 'bookworm'

  galaxy_tags:
    - 'debian'
    - 'base'
    - 'os'

dependencies: []
```

## `0640 {USER}:{USER}` meta/requirements.yml
Any tasks **not** in `ansible.builtin` should appear with URL, collection, and
module used.
``` yaml
---
collections:
  # https://docs.ansible.com/ansible/latest/collections/community/general/
  # ufw, timezone, locale_gen
  - community.general

roles: []
```

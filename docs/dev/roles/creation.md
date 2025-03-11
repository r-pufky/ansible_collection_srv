# Role Creation
Prerequisite:
* [ansible environment](../environment/ansible.md)

All roles are added as [submodules](submodules.md) in the collection repo.

Always create roles explicitly designed for bare-metal installations. Any VM or
container workarounds must be applied by the consumer of the role, not the role
itself. Consumed data (e.g. user data directories) manipulation should be
toggled off by default.

Creation will overwrite contents of the specified location, destroying existing
git checkouts. Create the role with a temporary name and move created items
into the git checkout location.

``` bash
ansible-galaxy role init --init-path roles example
```
[Reference](https://goetzrieger.github.io/ansible-collections/5-creating-collections/#adding-content-adding-a-custom-role)

Standards for directory layout:
```
├──── .ansible  # transient molecule testing cache (very large); do not commit.
├─┬── defaults
│ └┬─ main
│  ├─ main.yml  # ansible specific role configuration (users, locations, etc).
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

## `0640 {USER}:{USER}` .gitignore
Ignore new `.ansible` caching directory.
``` yaml
.ansible/
.ansible
```

## `0640 {USER}:{USER}` meta/requirements.yml
Any tasks **not** in `ansible.builtin` should appear with URL, collection, and
module used.
``` yaml
---
collections:
  - 'community.general'  # https://docs.ansible.com/ansible/latest/collections/community/general/

roles: []
```

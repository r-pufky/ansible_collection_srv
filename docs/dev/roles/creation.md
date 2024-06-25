# Role Creation
Assumes ansible and collection [environment setup](../collection/setup.md).

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
├──── molecule
├─┬── meta
│ └┬─ main.yml  # galaxy metadata
│  └─ requirements.yml  # collection and roles required
├──── tasks
├──── templates
├──── vars
├──── LICENSE
└──── README.md
```

## meta/main.yml
``` yaml
---
galaxy_info:
  author: 'r_pufky'
  description: 'Role configuration'
  company: 'N/A'
  license: 'AGPL-3.0 License'
  min_ansible_version: '2.16.0'

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

## meta/requirements.yml
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

# Submodule Creation
Collection roles are git submodules.

Role creation will overwrite the default git files pulled for submodules; copy
created template files into the submodule.

From collection root:
``` bash
git submodule add https://github.com/{USER}/{REPO} roles/fonts
```
[Reference](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

Enable main repo [summaries & out of date checking.](#enable-submodule-summaries-and-out-of-date)

## Enable Submodule Summaries and Out of Date
Make changes from collection repo (stored in `.git/config`).

Enable submodule summary on main repo commits/status
``` bash
git config status.submodulesummary 1
```

Check for submodules out of date/uncommitted when pushing main repo
``` bash
git config push.recurseSubmodules check
```

# Update Submodules for Collection
This will sync the submodule to head (or specified version); these will be
committed tags to the collection repo.

Changes may be made in the submodule -- be sure to commit these then run this
command afterwards to ensure the main repo has the correct reference.
``` bash
git submodule update --init  # to pull latest submodule release to head.
```
[Reference](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

# Removing a Submodule
Usually after a bad initial creation or weird sync state.

From collection root:
``` bash
rm -rfv roles/{ROLE}
git rm -r roles/{ROLE}
rm -rf .git/modules/roles/{ROLE}
git config --remove-section submodule.roles/{ROLE}
```
Submodule is fully removed and ready to be re-added.

[Reference](https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule)

# Troubleshooting

## Multiple configurations found for 'submodule.roles/{ROLE}'
The local copy of submodules `.git/config` mis-matches the checked in
submodules file `.gitmodules`.

Ensure `.git/config` modules match `.gitmodules` or the following warning will
occur on push:
``` bash
warning: {HASH}:.gitmodules, multiple configurations found for 'submodule.roles/{ROLE}'. Skipping second one!
```
[Reference](https://stackoverflow.com/questions/51883100/git-submodule-warning-multiple-configurations-found)

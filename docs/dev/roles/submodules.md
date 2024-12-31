# Submodules
Prerequisite:
* [ansible environment](../environment/ansible.md)

Collection roles are git submodules.

## Creation
[Role creation](creation.md) will overwrite the default git files pulled for
submodules; copy created template files into the submodule.

From collection root:
``` bash
git submodule add https://github.com/{USER}/{REPO} roles/{ROLE}
```
[Reference](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

Enable Submodule Summaries and Out of Date Checking (main repo):
``` bash
git config status.submodulesummary 1  # enable submodule summary on main repo
git config push.recurseSubmodules check  # check out of date/uncommitted submodules on push
```
* Changes are stored in main repo git config (`git/config`).

# Update Submodules for Collection
This will sync the submodule to head (or specified version); these will be
committed tags to the collection repo.

Changes may be made in the submodule -- be sure to commit these then run this
command afterwards to ensure the main repo has the correct reference.
``` bash
git submodule update --init  # from roles/{ROLE}; pulls latest release to repo
git add roles/{ROLE}  # add updated submodule to main repo commit
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

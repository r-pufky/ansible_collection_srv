# One-time Collection Initialization
Only for new repositories without existing collections.

``` bash
git clone https://github.com/{USER}/{REPO} /var/git/{COLLECTION}
cd ~/git/{COLLECTION}
ansible-galaxy collection init --init-path . r_pufky.srv
mv -vn r_pufky/srv/* .
rm -rfv r_pufky
mkdir -p plugins/modules  # for code like python
touch plugins/modules/demo_hello.py  (paste from site)  # demo
```
[Collection Structure](https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collections-doc-dir)

[Collection Template](https://github.com/ansible-collections/collection_template/tree/main)

[Creating Collections](https://goetzrieger.github.io/ansible-collections/5-creating-collections/)

## Enable Submodule Summaries and Out of Date
Check out of date/uncommitted submodules on push and provide a better summary
of submodule status (stored in `.git/config`).

Enable submodule summary on main repo commits/status
``` bash
git config status.submodulesummary 1
```

Check for submodules out of date/uncommitted when pushing main repo
``` bash
git config push.recurseSubmodules check
```

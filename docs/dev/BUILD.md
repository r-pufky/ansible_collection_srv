# Setup ansible environment
#
# Reference:
# * https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html
# * https://docs.ansible.com/ansible/latest/community/collection_contributors/collection_requirements.html
# * https://docs.ansible.com/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix
# * https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collection-structure
# * https://goetzrieger.github.io/ansible-collections/5-creating-collections/
#
# * https://docs.ansible.com/ansible/latest/reference_appendices/special_variables.html#magic-variables
# * https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#default-groups
# * https://docs.ansible.com/ansible/2.8/user_guide/playbooks_best_practices.html
# * https://ansible.readthedocs.io/projects/molecule/getting-started/
# * https://ansible.readthedocs.io/projects/molecule/examples/podman/
# * https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/#systemd-container
# * https://ansible.readthedocs.io/projects/lint/rules/yaml/
# * https://docs.ansible.com/ansible/latest/dev_guide/migrating_roles.html

# TODO: maybe use some of these in role updates to ensure everything working?
# * https://docs.ansible.com/ansible/latest/reference_appendices/test_strategies.html

#
# $ python -m virtualenv ansible  # Create a virtualenv if one does not already exist
# $ source ansible/bin/activate   # Activate the virtual environment
# $ pip install ansible

python -m virtualenv ~/.config/venv/ansible
source ~/.config/venv/ansible/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install ansible
pip install argcomplete
pip install ansible-lint
pip install molecule
pip install molecule-plugins[podman]
pacman -Syu podman

# Unit testing role w/ molecule and podman (expanded for integration tests with collection)
#
# Reference:
# * https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-1/
# * https://wiki.archlinux.org/title/Podman

pacman -S podman (fuse-overlayfs is NOT needed anymore in latest releases)

verify rootless support:
  podman info | grep -i overlay

  107:  graphDriverName: overlay
  114:    Native Overlay Diff: "true"

overlay and diff true means supported

verify unpriviledged user namespace enabled:

  sysctl kernel.unprivileged_userns_clone

1 (enabled)

create sub uid/gid mappings:
  configuration entry must exist for each user that wants to use it. New users
  created using useradd(8) have these entries by default. If not add user
  defaults.

  cat /etc/subuid
  cat /etc/subgid

  usermod --add-subuids 100000-165535 --add-subgids 100000-165535 username

# set default container registry for podman
# * https://halukkarakaya.medium.com/how-to-configure-default-search-registries-in-podman-ea930289692
/etc/containers/registries.conf

  [registries.search]
  registries = ['ghcr.io']
  unqualified-search-registries=['ghcr.io']

# enable rootless use of ping
/etc/containers/containers.conf

  default_sysctls = [
    "net.ipv4.ping_group_range=0 2147483647",
  ]

# Critical for unprivileged podman to execute properly. Run as the current
# user.
#
# Reference:
# * https://github.com/containers/podman/issues/12715
podman system migrate




# Create collection development git managed location
git clone https://github.com/r-pufky/ansible_collection_srv ~/git/ansible_collection_srv
cd !$  (or alt + . or $_)

# one time -- init ansible collection on git directory
ansible-galaxy collection init --init-path . r_pufky.srv
mv -vn r_pufky/srv/* .
rm -rfv r_pufky
mkdir -p plugins/modules  # demo
touch plugins/modules/demo_hello.py  (paste from site)  #demo
# https://goetzrieger.github.io/ansible-collections/5-creating-collections/

# Create a new role in the collection (demo role)
ansible-galaxy init --init-path roles hello_motd

# Setup VScode
Install the 'Ansible' extension
ansible -> settings -> workspace
ansible path: ~/.config/venv/ansible/bin/ansible
always use FQCN: check
reuse terminal: check
python interpreter path: ~/.config/venv/ansible/bin/python
python activatioon script: ~/.config/venv/ansible/bin/activate
ansible navigator path: ansible-navigator
completion provide redirect modules: check
completino provide module option aliases: check
validation: enable
validation lint: enable
validation lint path: ansible-lint
redhat telemetry: uncheck
--> save workspace configuration

# Build ansible collection
ansible-galaxy collection build (--force to overwrite)
--> This will build a semantic versioned package for local install
ansible-galaxy collection install (--force to overwrite)
--> This will install the collection in the 'local' collections cache
~/.ansible/collections/ansible_collections/r_pufky/srv
--> collection will need to be rebuild if changes were made and tests changed.

# Testing collections
https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_testing.html
https://docs.ansible.com/ansible/latest/community/collection_contributors/test_index.html#testing-collections-guide
https://docs.ansible.com/ansible/latest/dev_guide/testing_running_locally.html#testing-running-locally

# https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_documenting.html
meta/runtime.yml
  requires_ansible: '>=2.16'

plugins/modules/demo_hello.py
  version_added: "1.0.0" --> This is the collection version it was added in NOT the ansible version

plugins/modules/demo_hello.py
  RETURN = '''
  fact:
    returned: success

# Initalize roles with molecule (uses ansible-galaxy with molecule templating)
#
# molecule init role mywebapp --driver-name=podman
#
# 'role' removed; just use scenario after creating the role. See reference.
#
# Use reference for templated file sources.
#
# Refernce:
# * https://github.com/geerlingguy/ansible-for-devops/issues/574
# * https://goetzrieger.github.io/ansible-collections/5-creating-collections/
ansible-galaxy role init hello_motd
cd hello_motd
molecule init scenario --driver-name=podman

roles/hello_motd/defaults/main.yml (ref)
roles/hello_motd/meta/main.yml (ref)
roles/hello_motd/tasks/main.yml (ref - use /tmp instead of /etc)

mkdir roles/hello_motd/tests/playbook
touch roles/hello_motd/tests/playbook/test_hello_motd.yml (ref - use YOUR collection name)

plugins/modules/demo_hello.py (add import futures, metaclass type)

  from __future__ import (absolute_import, division, print_function)
  __metaclass__ = type

# --ask-become-pass if sudo is needed or use the GPG key injection
ansible-playbook roles/hello_motd/tests/playbook/test_hello_motd.yml

  will still get the following warnings when running
  [WARNING]: No inventory was parsed, only implicit localhost is available
  [WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost does not match
  'all'

  This will remove warnings but fail because localhost does not accept ssh:
  ansible-playbook -i tests/inventory tests/playbook/test_hello_motd.yml

# Running collection tests
# Always be sure to build and install before running.
#
# This will generate warnings and errors, fixed below.
cd ~/.ansible/collections/ansible_collections/r_pufky/srv
ansible-test sanity
ansible-test integration
ansible-test units

# Fixing GPLv3 license missing
# community distributions assume GPLv3. Must explicitly ignore these.
#
# Manually create 'tests' directory in collection directory.
#
# An ignore file is required for each specific version of ansible.
#
# ORDER MATTERS - if ignores are throwing warnings, check ordering and make
# sure they are ordered correctly. Proper ignores will NOT generate warnings.
#
# Only certain ignores are explicitly allowed; others will always throw
# ansible-lint errors. See reference.
#
# Reference:
# * https://github.com/ansible/ansible/issues/67032
# * https://docs.ansible.com/ansible/latest/dev_guide/testing/sanity/ignores.html
# * https://docs.ansible.com/ansible/latest/dev_guide/testing/sanity/validate-modules.html
# * https://ansible.readthedocs.io/projects/lint/rules/sanity/

mkdir -p tests/sanity
touch tests/sanity/ignore-2.16.txt
plugins/modules/demo_hello.py compile-2.7!skip
plugins/modules/demo_hello.py import-2.7!skip
plugins/modules/demo_hello.py compile-3.6!skip
plugins/modules/demo_hello.py import-3.6!skip
plugins/modules/demo_hello.py compile-3.7!skip
plugins/modules/demo_hello.py import-3.7!skip
plugins/modules/demo_hello.py compile-3.8!skip
plugins/modules/demo_hello.py import-3.8!skip
plugins/modules/demo_hello.py compile-3.9!skip
plugins/modules/demo_hello.py import-3.9!skip
plugins/modules/demo_hello.py compile-3.10!skip
plugins/modules/demo_hello.py import-3.10!skip
plugins/modules/demo_hello.py compile-3.12!skip
plugins/modules/demo_hello.py import-3.12!skip
plugins/modules/demo_hello.py validate-modules:missing-gplv3-license

# Setup molecule containers for testing
#
# Use podman as it is a full systemd container; and doesnt have the network
# issues (and many issues) that docker does.
#
# * ghcr.io/hifis-net/debian-systemd:12
# * ghcr.io/hifis-net/debian-systemd:11
# * ghcr.io/hifis-net/debian-systemd:10
#
# Molecule removed the 'linting' option in updates. Linting is done through
# other tools. Molecule is ONLY used to test now. See reference 3802.
#
# Podman requires tmpfs to be a dictionary not a list. See reference 4140.
# This also states that tmpfs isn't needed anymore with 'systemd: always'.
#
# Reference:
# * https://www.ansible.com/blog/developihttps://goetzrieger.github.io/ansible-collections/5-creating-collections/
ng-and-testing-ansible-roles-with-molecule-and-podman-part-1/
# * https://www.ansible.com/blog/developing-and-testing-ansible-roles-with-molecule-and-podman-part-2/
# * https://ansible.readthedocs.io/projects/molecule/getting-started/#inspecting-the-moleculeyml
# * https://github.com/hifis-net/debian-systemd
# * https://github.com/hspaans/molecule-containers/pkgs/container/molecule-container-debian
# * https://github.com/ansible/molecule/pull/3802
# * https://github.com/ansible/molecule/issues/4140
# * https://ansible.readthedocs.io/projects/molecule/guides/systemd-container/

# potentially insufficient UIDs or GIDs available in yser namespace
# potentially this: https://github.com/systemd/systemd/issues/21952
#
# Run privileged ?
hello_motd/molecule/default/molecule.yml
  ---
  dependency:
    name: galaxy
  driver:
    name: podman
  provisioner:
    name: ansible
    config_options:
      defaults: #
        interpreter_python: auto_silent # suppress warnings
        callback_whitelist: profile_tasks, timer, yaml # output profiling info
      ssh_connection:
        pipelining: false # does not work with podman
  platforms:
    - name: instance
      image: ghcr.io/hifis-net/debian-systemd:12
      systemd: always
      # tmpfs:
      #   - /run
      #   - /tmp
      volumes:
        - /sys/fs/cgroup:/sys/fs/cgroup:ro
      capabilities:
        - SYS_ADMIN # required for systemd systems
      command: "/lib/systemd/systemd"
      pre_build_image: true
  verifier:
    name: ansible
  lint: |
    set -e
    yamllint .
    ansible-lint .

# create molecule testing instances
#
# GHCR.IO requires a github login to download images. Login to this!
#
# The warning message cannot be disabled. See reference.
#
#   WARNING Driver podman does not provide a schema.
#
# Reference:
# * https://github.com/ansible/molecule/discussions/4108
podman login ghcr.io
cd roles/hello_motd
molecule create
molecule test
molecule test -- -v

# multiple scenarios
molecule test --all
molecule test --scanario-name other_scenario

# Molecule overview and directory structure/notes
# * https://sbarnea.com/slides/molecule/#/13

# Releasing collection
# Reference:
# * https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_creating.html
ansible-galaxy collection build
ansible-galaxy collection publish


# Importing other git repositories as submodules (separate tracked repos)
# https://git-scm.com/book/en/v2/Git-Tools-Submodules
git submodule add https://github.com/r-pufky/ansible_fonts roles/fonts

# Adding documentation to collection / roles.
# https://docs.ansible.com/ansible/latest/dev_guide/developing_collections_structure.html#collections-doc-dir
# collection template
# https://github.com/ansible-collections/collection_template/tree/main

# Ignoring files for building (likely tests, etc)
# https://docs.ansible.com/ansible/devel/dev_guide/developing_collections_distributing.html#ignoring-files-and-folders
#
#   galaxy.yml
#     build_ignore:
#       - .gitignore
#       - changelogs/.plugin-cache.yaml


# linting
#
# always grep for existing disables and try to remove them. Explicit disables
# only.
#
#   grep -ri yamllint  # yamllint
#   grep -ri noqa      # ansible-lint
#   grep -ri todo      # any open todo's
#
# Only accept file level disables for yaml.
#
# Reference:
# * https://yamllint.readthedocs.io/en/stable/
# * https://ansible.readthedocs.io/projects/lint/rules/
yamllint .
ansible-lint .

# Ensure grep for ansible tasks and include in meta/requirements.yml for
# collections/roles that this role depends on.

# DEFAULT VALUES
# * https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_filters.html#providing-default-values
#
# default(VALUE, true) -- true is used to set the default value if false or ''; else raises undefined error.

Role compliant for galaxy collection deployment.


Using submodules:
* https://git-scm.com/book/en/v2/Git-Tools-Submodules
git submodule update --init  # to pull latest submodule release to head.

# Setup Ansible Environment

## Create Environment and Install Packages
Create python virtual environment, install Ansible and Molecule pip packages:
``` bash
python3 -m venv /var/venv/ansible_collection_srv
source /var/venv/ansible_collection_srv/bin/activate
pip install --upgrade pip
pip install --upgrade setuptools
pip install ansible
pip install argcomplete
pip install ansible-lint
pip install molecule
```
[Reference](https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html)

### Update Ansible Environment
A minimal (clean) environment is provided to test collection and roles against
a vanilla ansible installation; enforcing only settings which make testing
easier. Update environment file to use *your* specific settings.

`0644 {USER}:{USER}` [ansible.env](../../../ansible.env)
``` bash
# Ansible serving/caching directory for controller node (high read/writes).
export SRV_DEPLOY="${HOME}/.ansible"  # ${ANSIBLE_HOME}

# Ansible python virtual environment (high read/writes)
export SRV_VENV='/var/venv/ansible_collection_srv'

# SRV git repository root directory (high read only)
export SRV_GIT='/var/git/ansible_collection_srv'
```

## Set alternative container storage location (optional)
Developing ansible will thrash disk especially when running playbooks and
testing with molecule. Relocate high-use directories to a disk that can handle
high wear. Prefer to config change as this enables vanilla use in production.

``` bash
# delete or move existing cache data.
ln -s /mnt/cache/.ansible_async ${HOME}/.ansible_async
ln -s /mnt/cache/.ansible ${HOME}/.ansible
ln -s /mnt/cache/cache/ansible-compat ${HOME}/.cache/ansible-compat
ln -s /mnt/cache/cache/molecule ${HOME}/.cache/molecule
```
* Consider moving and linking entire `.cache` directory.

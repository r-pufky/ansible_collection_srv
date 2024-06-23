# Module Development
Loose module development notes from tutorial (and changes required for the
tutorial to work)

``` bash
ansible-galaxy role init hello_motd
cd hello_motd
molecule init scenario --driver-name=podman
```

roles/hello_motd/defaults/main.yml (ref)
roles/hello_motd/meta/main.yml (ref)
roles/hello_motd/tasks/main.yml (ref - use /tmp instead of /etc)

``` bash
mkdir roles/hello_motd/tests/playbook
touch roles/hello_motd/tests/playbook/test_hello_motd.yml  # ref - use YOUR collection name
```

plugins/modules/demo_hello.py (add import futures, metaclass type)

  from __future__ import (absolute_import, division, print_function)
  __metaclass__ = type


# https://docs.ansible.com/ansible/latest/dev_guide/developing_modules_documenting.html
meta/runtime.yml
  requires_ansible: '>=2.16'

plugins/modules/demo_hello.py
  version_added: "1.0.0" --> This is the collection version it was added in NOT the ansible version

plugins/modules/demo_hello.py
  RETURN = '''
  fact:
    returned: success

# Firmware
Firmware update management.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_firmware/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_firmware/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_firmware/blob/main/defaults/main)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
Add additional firmware repositories if needed. This will install firmware
packages and apply firmware updates to hardware.

group_vars/all/vars/debian.yml
```
firmware_packages:
  - 'intel-microcode'
firmware_packages_absent:
  - 'firmware-bnx2x'
firmware_timeout: 600
firmware_poll: 5
```

Apply the base role
``` yaml
- name: 'Update system firmware'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.firmware'
```

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_firmware/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)

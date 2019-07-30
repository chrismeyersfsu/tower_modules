# Ansible Tower Modules Collection
This repo is a proper Ansible collection of Ansible Tower modules that can be used to interact with the Tower and AWX API.

[Docs](https://docs.ansible.com/ansible/latest/modules/list_of_web_infrastructure_modules.html#ansible-tower)

### Get ansible.tower collection
The ability to call collections was added to Ansible 2.8. In Ansible 2.9 `ansible-galaxy` will be capable of installing collections from galaxy.

If you are using **Ansible 2.8** then you need the python package `mazer` and use it to install the `ansible.tower` collection.

```
pip install mazer
mazer install ansible.tower
```

If you are using **Ansible >= 2.9** then you will use `ansible-galaxy` to install collections.

```
ansible-galaxy collection install ansible.tower
```

By default, the collections will be installed in `~/.ansible/collections/ansible_collections/<namespace>/<collection_name>`

### Test call ansible.tower collection
```yaml
# main.yml
- hosts: localhost
  gather_facts: false
  tasks:
    - name: Create Tower Organization
      ansible.tower.tower_organization:
        name: "My Org"
```

```
ansible-playbook main.yml -vvv
```

```
PLAYBOOK: main.yml ***********************************************************************************************************************************************************************
1 plays in main.yml

PLAY [localhost] *************************************************************************************************************************************************************************
META: ran handlers

TASK [ansible.tower.tower_organization] ***********************************************************************************************************************************
task path: /tmp/testy/testy/main.yml:4
<127.0.0.1> ESTABLISH LOCAL CONNECTION FOR USER: meyers
<127.0.0.1> EXEC /bin/sh -c 'echo ~meyers && sleep 0'
<127.0.0.1> EXEC /bin/sh -c '( umask 77 && mkdir -p "` echo /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308 `" && echo ansible-tmp-1564681229.66-9483478929308="` echo /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308 `" ) && sleep 0'
Using module file /home/meyers/.ansible/collections/ansible_collections/ansible/tower/plugins/modules/tower_organization.py
<127.0.0.1> PUT /home/meyers/.ansible/tmp/ansible-local-434iJkaXt/tmp79F0mg TO /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308/AnsiballZ_tower_organization.py
<127.0.0.1> EXEC /bin/sh -c 'chmod u+x /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308/ /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308/AnsiballZ_tower_organization.py && sleep 0'
<127.0.0.1> EXEC /bin/sh -c '/home/meyers/ansible/virtualenv/ansible-dev/bin/python2 /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308/AnsiballZ_tower_organization.py && sleep 0'
<127.0.0.1> EXEC /bin/sh -c 'rm -f -r /home/meyers/.ansible/tmp/ansible-tmp-1564681229.66-9483478929308/ > /dev/null 2>&1 && sleep 0'

```

We know that the collection version is being used because of the following line in the output `Using module file /home/meyers/.ansible/collections/ansible_collections/ansible/tower/plugins/modules/tower_organization.py`


### Migrate playbooks to use ansible.tower collection

Old:
```yaml
- hosts: localhost
  gather_facts: false
  tasks:
    - name: Create Tower Organization
      tower_organization:
        name: "My Org"
```

How to migrate; easy mode:
```yaml
- hosts: localhost
  gather_facts: false
  collection:
    - ansible.tower
  tasks:
    - name: Create Tower Organization
      tower_organization:
        name: "My Org"
```
 
How to migrate; using fully qualified module path:
```yaml
- hosts: localhost
  gather_facts: false
  tasks:
    - name: Create Tower Organization
      ansible.tower.tower_organization:
        name: "My Org"
```

If you did the above and ran it you will notice a deprecation warning. With the advent of collections the `tower_` portion of the module names is redundant and we have dropped it in favor of just the resource name that it represents instead. For example `tower_organization` becomes `organization` and `tower_project` becomes `project`. Using the fully qualified module name for example `ansible.tower.tower_organization` becomes `ansible.tower.organization` and `ansible.tower.tower_project` becomes `ansible.tower.project`.

New, using fully qualified module path:
```yaml
- hosts: localhost
  gather_facts: false
    - name: Create Tower Organization
      ansible.tower.organization:
        name: "My Org"
 ```
 
 New, easy mode:
 ```yaml
- hosts: localhost
  collections:
    - ansible.tower
  gather_facts: false
    - name: Create Tower Organization
      organization:
        name: "My Org"
 ```
 
## FAQ's
_Why did tower modules change to using collections?_
<br>
Collections allow us to release fixes and features outside of the Ansible release cycle. This means features and fixes reach our users faster.

_How do I use the `ansible.tower` collection in Tower?_
<br>
By adding the collection `ansible.tower` to a project file `collections/requirements.yml`. Similarly to how Tower downloads roles in `roles/requirements.yml`, Tower will download collections listed in `collections/requirements.yml` i.e.

```yaml
---
collections:
  - ansible.tower
```

_I get the below failure. What's wrong?_

`fatal: [localhost]: FAILED! => {"changed": false, "msg": "Failed to import the required Python library (ansible-tower-cli) on thinkbeefy's Python /home/meyers/ansible/virtualenv/ansible-dev/bin/python2. Please read module documentation and install in the appropriate location"}`

The `ansible-tower-cli` python package is needed for the tower modules. Run `pip install ansible-tower-cli`. If you are _in_ Tower you will need to create a `virtualenv` and install the module inside. Instructions can be found [here](https://docs.ansible.com/ansible-tower/latest/html/upgrade-migration-guide/virtualenv.html#using-virtualenv-with-at).


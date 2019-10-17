# Blue Planet Integration Tests

Write integration and automation tests for Ciena's Blue Planet orchestration platform

[Ansible Galaxy](https://galaxy.ansible.com/jgroom33/blueplanet_integration_tests)

## Recommended Installation

```bash
ansible-galaxy install jgroom33.blueplanet_integration_tests
```

## Requirements

Blue Planet

## Role Variables

bp_integration_tests_server: http://localhost:8181

bp_integration_tests_username: admin

bp_integration_tests_password: adminpw

bp_integration_tests_password_token: none

## Dependencies

N/A

## Example Playbook

### Scenario 1 - Onboarding Validations

```yaml
###### main.yml #######
###### main.yml #######
###### main.yml #######
---
- hosts: localhost
  vars:
    bp_integration_tests_server: http://localhost:8181
  gather_facts: no
  roles:  ## This performs login via always tag from login import
  - jgroom33.blueplanet_integration_tests
  tasks:
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/validate_resource_provider.yml
    vars:
      validate_domain_urns:
      - urn:ciena:bp:domain:bpraciscoiosxr
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/validate_resourceTypes.yml
    vars:
      validate_resourceTypes:
      - bpraciscoiosxr.resourceTypes.NetworkFunction
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/validate_products.yml
    vars:
      validate_products:
      - tosca.resourceTypes.Monitor
```

```yaml
###### main.yml #######
###### main.yml #######
###### main.yml #######
---
- hosts: localhost
  vars:
    mdso:
      bp_integration_tests_server: http://localhost:8181
  gather_facts: no
  roles:
  - jgroom33.blueplanet_integration_tests
  tasks:
## ---------------- Create -----------------
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: create_domain
    vars:
      - domainType: bpraciscoiosxr
      - title: 'Cisco_IOS_XR_Domain_Integration_Test'
      - description: 'Integration Test'
      - accessUrl: 'http://localhost:9192'
      - address:
          country: Canada
          state: ""
          zip: ""
          street: ""
          longitude: -85.706546
          latitude: 38.344877
          city: Vancouver
      - properties: {}
    tags:
      - create_domain
      - create
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: create_resource_by_resourceTypeId
    tags:
      - create_sessionProfile
      - create
    vars:
      - resourceTypeId: bpraciscoiosxr.resourceTypes.SessionProfile
      - label: SessionProfile_{{ hostvars['host1']['label'] }}
      - properties:
          typeGroup: "/typeGroups/Cisco"
          authentication:
              netconf:
                  username: "{{ hostvars['host1']['username'] }}"
                  password: "{{ hostvars['host1']['password'] }}"
          connection:
              netconf:
                  hostport: "{{ hostvars['host1']['netconfHostPort'] }}" # Note: This requires https://stackoverflow.com/questions/52487396 in order to be an integer
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: create_resource_by_resourceTypeId
    tags:
      - create_networkFunction
      - create
    vars:
      - resourceTypeId: bpraciscoiosxr.resourceTypes.NetworkFunction
      - label: NetworkFunction_{{ hostvars['host1']['label'] }}
      - properties:
          sessionProfile: "{{ keyed_resources['SessionProfile_' + hostvars['host1']['label'] ] }}"
          ipAddress: "{{ hostvars['host1']['ansible_host'] }}"
  - import_tasks: get_named_explicit_paths.yml
    tags:
      - get_named_explicit_paths
      - create
  - import_tasks: get_tunnels.yml
    tags:
      - get_tunnels
      - create
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: create_resource_by_resourceTypeId
    tags:
      - create_namedExplicitPath
      - create
    vars:
      - resourceTypeId: bpraciscoiosxr.resourceTypes.NamedExplicitPath
      - label: "ansible_path_1"
      - properties:
          {
              "device": "{{ keyed_resources['NetworkFunction_' + hostvars['host1']['label'] ] }}",
              "config": {
              "name": "ansible_path_1"
              },
              "explicitRouteObjects": [
                  {
                      "config": {
                      "address": "1.1.1.1",
                      "hopType": "STRICT",
                      "index": 1
                      }
                  },
                  {
                      "config": {
                      "address": "2.2.2.2",
                      "hopType": "LOOSE",
                      "index": 2
                      }
                  },
                  {
                      "config": {
                      "address": "3.3.3.3",
                      "hopType": "STRICT",
                      "index": 3
                      }
                  }
              ]
          }
## ---------------- Patch -----------------
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: patch_resource
    tags:
      - patch_namedExplicitPath
      - patch
    vars:
      - resourceId: "{{ keyed_resources['ansible_path_1'] }}"
      - properties: {
          "explicitRouteObjects": [
            {
              "config": {
                "address": "4.4.4.4",
                "hopType": "STRICT",
                "index": 2
              }
            },
            {
              "config": {
                "address": "5.5.5.5",
                "hopType": "STRICT",
                "index": 3
              }
            },
            {
              "config": {
                "address": "6.6.6.6",
                "hopType": "LOOSE",
                "index": 4
              }
            }
          ]
        }
## ---------------- Delete -----------------
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: delete_resource
    tags:
      - delete_namedExplicitPath
      - delete
    vars:
    - resourceId: "{{ keyed_resources['ansible_path_1'] }}"
  - debug:
      msg: resourceId={{ keyed_resources['NetworkFunction_' + hostvars['host1']['label'] ] }}
    tags:
      - delete
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: delete_resource
    tags:
      - delete_networkFunction
      - delete
    vars:
    - resourceId: "{{ keyed_resources['NetworkFunction_' + hostvars['host1']['label'] ] }}"
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: delete_resource
    tags:
      - delete_sessionProfile
      - delete
    vars:
    - resourceId: "{{ keyed_resources['SessionProfile_' + hostvars['host1']['label'] ] }}"
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: delete_domain
    tags:
      - delete_domain
      - delete
    vars:
      - domainId: "{{ keyed_domains['Cisco_IOS_XR_Domain_Integration_Test'] }}"
```

```yaml
###### inventory.yml #######
###### inventory.yml #######
###### inventory.yml #######
cisco:
  hosts:
    host1:
      ansible_host: cisco-0.cisco-asr.ciena.cloud.tesuto.com
      label: Cisco_XR_0
      username: tesutocli
      password: bar
      netconfHostPort: 830
```

The above would be executed as follows:

```bash
ansible-galaxy install jgroom33.blueplanet_integration_tests
# Just create
ansible-playbook main.yml -t create
# Patch
ansible-playbook main.yml -t patch
# then delete
ansible-playbook main.yml -t delete
```

License
-------

Apache 2.0

Author Information
------------------

Jeff Groom (jgroom@ciena.com)

Table render broken
-------------------

Default fact storage:

| Market entity |             task                |                                       input                                        |  output fact (returns this global var)   |
| :-----------: | :-----------------------------: | :--------------------------------------------------------------------------------: | :--------------------------------------: |
|   resource    |         create_resource         |                     resourceTypeId <br> label <br> properties                      |           resourceId_`<label>`           |
|   resource    |   get_resource_one_by_filters   |                                                                                    | resource (Dict) <br> resourceId (String) |
|   resource    | get_resources_by_resourceTypeId |                                   resourceTypeId                                   | resources (Dict) <br> resourceId (String) |
|    domain     |          create_domain          | title <br> description <br> accessUrl <br> domainType <br> properties <br> address |            domainId_`<title>`            |
|    domain     |          delete_domain          |                                      domainId                                      |                                          |

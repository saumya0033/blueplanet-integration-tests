# Blue Planet Integration Tests

Write integration and automation tests for Ciena's Blue Planet orchestration platform

## Recommended Installation

```bash
ansible-galaxy install --roles-path . jgroom33.blueplanet_integration_tests
```

## Requirements

Blue Planet

## Role Variables

BP_SERVER (default http://localhost:8181)

BP_USERNAME (default admin)

BP_PASSWORD (default adminpw)

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
    mdso:
      BP_SERVER: http://localhost:8181
  gather_facts: no
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
      BP_SERVER: http://localhost:8181
  gather_facts: no
  tasks:
## ---------------- Create -----------------
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/create_domain.yml
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
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/create_resource_by_resourceTypeId.yml
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
    tags:
      - create_sessionProfile
      - create
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/create_resource_by_resourceTypeId.yml
    vars:
      - resourceTypeId: bpraciscoiosxr.resourceTypes.NetworkFunction
      - label: NetworkFunction_{{ hostvars['host1']['label'] }}
      - properties:
          sessionProfile: "{{ keyed_resources['SessionProfile_' + hostvars['host1']['label'] ] }}"
          ipAddress: "{{ hostvars['host1']['ansible_host'] }}"
    tags:
      - create_networkFunction
      - create
  - import_tasks: get_named_explicit_paths.yml
    tags:
      - get_named_explicit_paths
      - create
  - import_tasks: get_tunnels.yml
    tags:
      - get_tunnels
      - create
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/create_resource_by_resourceTypeId.yml
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
    tags:
      - create_namedExplicitPath
      - create
## ---------------- Patch -----------------
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/patch_resource.yml
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
    tags:
      - patch_namedExplicitPath
      - patch
## ---------------- Delete -----------------
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/delete_resource.yml
    vars:
    - resourceId: "{{ keyed_resources['ansible_path_1'] }}"
    tags:
      - delete
      - delete_namedExplicitPath
  - debug:
      msg: resourceId={{ keyed_resources['NetworkFunction_' + hostvars['host1']['label'] ] }}
    tags:
      - delete
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/delete_resource.yml
    vars:
    - resourceId: "{{ keyed_resources['NetworkFunction_' + hostvars['host1']['label'] ] }}"
    tags:
      - delete
      - delete_networkFunction
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/delete_resource.yml
    vars:
    - resourceId: "{{ keyed_resources['SessionProfile_' + hostvars['host1']['label'] ] }}"
    tags:
      - delete
      - delete_sessionProfile
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/delete_domain.yml
    vars:
      - domainId: "{{ keyed_domains['Cisco_IOS_XR_Domain_Integration_Test'] }}"
    tags:
      - delete
      - delete_domain
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

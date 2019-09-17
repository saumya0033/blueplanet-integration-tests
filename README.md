Role Name
=========

Write integration and automation tests to Ciena's Blue Planet orchestration platform

Requirements
------------

Blue Planet

Role Variables
--------------

BP_SERVER
BP_USERNAME
BP_PASSWORD

Dependencies
------------

N/A

Example Playbook
----------------

Default fact storage:

| Market entity |             task                |                                       input                                        |  output fact (returns this global var)   |
| :-----------: | :-----------------------------: | :--------------------------------------------------------------------------------: | :--------------------------------------: |
|   resource    |         create_resource         |                     resourceTypeId <br> label <br> properties                      |           resourceId_`<label>`           |
|   resource    |   get_resource_one_by_filters   |                                                                                    | resource (Dict) <br> resourceId (String) |
|   resource    | get_resources_by_resourceTypeId |                                   resourceTypeId                                   | resources (Dict) <br> resourceId (String) |
|    domain     |          create_domain          | title <br> description <br> accessUrl <br> domainType <br> properties <br> address |            domainId_`<title>`            |
|    domain     |          delete_domain          |                                      domainId                                      |                                          |

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
###### main.yml #######
###### main.yml #######
###### main.yml #######
---
- 
  tasks:
## ---------------- Create -----------------
  - import_tasks: create_domain.yml
    tags:
      - create_domain
      - create
    vars:
      - domainType: bpraciscoiosxr
  - import_tasks: create_sessionProfile.yml
    tags:
      - create_sessionProfile
      - create
## ---------------- Delete -----------------
  - import_tasks: delete_sessionProfile.yml
    tags:
      - delete
      - delete_sessionProfile
  - import_tasks: delete_domain.yml
    tags:
      - delete
      - delete_domain
```

```yaml
###### create_domain.yml #######
###### create_domain.yml #######
###### create_domain.yml #######
- import_tasks: jgroom33.blueplanet_integration_tests/tasks/create_domain.yml
  vars:
    - domainType: bpraciscoiosxr
    - title: 'Cisco IOS XR Domain: Integration Test'
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

```

```yaml
###### create_sessionProfile.yml #######
###### create_sessionProfile.yml #######
###### create_sessionProfile.yml #######
- import_tasks: jgroom33.blueplanet_integration_tests/tasks/create_resource.yml
  vars:
    - resourceTypeId: bpraciscoiosxr.resourceTypes.SessionProfile
    - label: "{{ hostvars['host1']['label'] }}"
    - properties:
        typeGroup: "/typeGroups/Cisco"
        authentication:
            netconf:
                username: "{{ hostvars['host1']['username'] }}"
                password: "{{ hostvars['host1']['password'] }}"
        connection:
            netconf:
                hostport: "{{ hostvars['host1']['netconfHostPort'] }}" # Note: This requires https://stackoverflow.com/questions/52487396 in order to be an integer
```

```yaml
###### delete_sessionProfile.yml #######
###### delete_sessionProfile.yml #######
###### delete_sessionProfile.yml #######
  - import_tasks: jgroom33.blueplanet_integration_tests/tasks/delete_resources_by_resourceTypeId.yml
    vars:
      resourceTypeId: bpraciscoiosxr.resourceTypes.SessionProfile
```

```yaml
###### delete_domain.yml #######
###### delete_domain.yml #######
###### delete_domain.yml #######
- import_tasks: jgroom33.blueplanet_integration_tests/tasks/delete_domains_by_domainType.yml
  vars:
    - domainType: bpraciscoiosxr
```

License
-------

Apache 2.0

Author Information
------------------

Jeff Groom (jgroom@ciena.com)

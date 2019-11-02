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

* bp_integration_tests_server: http://localhost:8181
* bp_integration_tests_username: admin
* bp_integration_tests_password: adminpw
* bp_integration_tests_password_token: none
* bp_integration_tests_multi_domain: false

## Dependencies

N/A

## Example Playbook

### create_resource_by_resourceTypeId

Input:

|    property    | type    |
|:--------------:|:-------:|
| resourceTypeId | string  |
|     label      | string  |
|   properties   |   {}    |

Example:

```yaml
  - import_role:
      name: jgroom33.blueplanet_integration_tests
      tasks_from: create_resource_by_resourceTypeId
    vars:
      - resourceTypeId: tosca.resourceTypes.NumberPool
      - label: create_resource_by_resourceTypeId_test
      - properties:
          highest: 100
          lowest: 1
```

Generated Fact storage:

```json
    "bp_keyed_resources": {
        "create_resource_by_resourceTypeId_test": "5db8a8fd-0d4c-49d3-9255-2e3b43645d86"
    }
```

### delete_resource

The fact storage from the previous execution can be used for the delete

```yaml
  - import_role:
      name: ../jgroom33.blueplanet_integration_tests
      tasks_from: delete_resource
    vars:
    - bp_resource_id: "{{ bp_keyed_resources['create_resource_by_resourceTypeId_test'] }}"
```

> Review the [tests](tests/) directory for more examples

## Tests

```bash
ansible-playbook tests/chained/create.yml
ansible-playbook tests/chained/get.yml
ansible-playbook tests/chained/delete.yml

ansible-playbook tests/create-delete_by_id/playbook.yml

ansible-playbook tests/create-delete_by_resourceTypeId/playbook.yml

ansible-playbook tests/create-delete_domains/playbook.yml

ansible-playbook tests/filter/create.yml
ansible-playbook tests/filter/get.yml
ansible-playbook tests/filter/delete.yml

# ansible-playbook tests/hacks/playbook.yml

ansible-playbook tests/multi_domain/1-create_domain.yml
ansible-playbook tests/multi_domain/2-get_domains.yml
ansible-playbook tests/multi_domain/3-get_products.yml
ansible-playbook tests/multi_domain/4-create_resource.yml
ansible-playbook tests/multi_domain/5-get_resource.yml
ansible-playbook tests/multi_domain/6-delete_resource.yml
ansible-playbook tests/multi_domain/7-delete_domain.yml

ansible-playbook tests/validations/playbook.yml

```

## License

Apache 2.0

## Author Information

Jeff Groom (jgroom@ciena.com)

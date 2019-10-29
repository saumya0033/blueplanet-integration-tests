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

Review the tests directory

## Tests

```bash
ansible-playbook tests/playbook.yml

ansible-playbook tests/chained/create.yml
ansible-playbook tests/chained/delete.yml

ansible-playbook tests/filter/create.yml
ansible-playbook tests/filter/get.yml
ansible-playbook tests/filter/delete.yml

ansible-playbook tests/multi_domain/playbook.yml
```

## License

Apache 2.0

## Author Information

Jeff Groom (jgroom@ciena.com)

## Default fact storage:

| Market entity |                  task                   |                                       input                                        |    output fact (returns this global var)     |
|:-------------:|:---------------------------------------:|:----------------------------------------------------------------------------------:|:--------------------------------------------:|
|   resource    |             create_resource             |                     resourceTypeId <br> label <br> properties                      |     bp_keyed_resources:{`<label>`:`id`}      |
|   resource    |       get_resource_one_by_filters       |                                                                                    | resource:(Dict) <br> bp_resource_id (String) |
|   resource    |     get_resources_by_resourceTypeId     |                                   resourceTypeId                                   |              resources:[(Dict)]              |
|   resource    | get_created_resources_by_resourceTypeId |                                   resourceTypeId                                   |              resources:[(Dict)]              |
|    domain     |              create_domain              | title <br> description <br> accessUrl <br> domainType <br> properties <br> address |        keyed_domains:{`<title>`:`id`}        |
|    domain     |              delete_domain              |                                    bp_domain_id                                    |                                              |
|   products    |              get_products               |                                                                                    |    keyed_products:{`resourceTypeId`:`id`}    |
|   products    |      get_products_by_resourceType       |                                    resourceType                                    |    keyed_products:{`resourceTypeId`:`id`}    |
|   products    |       get_products_by_domainType        |                          urn:ciena:bp:domain:`domainType`                          |    keyed_products:{`resourceTypeId`:`id`}    |
|   products    |      get_products_by_resourceType       |                                   resourceTypeId                                   |    keyed_products:{`resourceTypeId`:`id`}    |

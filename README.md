# Ansible Examples

Role-based Ansible examples for Ubuntu 22.04 Jammy hosts.

This repo demonstrates:

- Shared playbook composition with `import_playbook`.
- Package installation loops.
- Generic INI rendering from structured variables.
- PHP-FPM global and pool config templates.
- PHP module install/config loops from `[module_name, so_filename, package_name, optional_config]` tuples.
- Directory tree management.
- Service state management with handlers.
- Tags for targeted package, config, module, pool, and service runs.
- Fact gathering for platform assertions and rendered config values.

## Layout

```text
inventories/example/hosts.yml
inventories/example/group_vars/all/common.yml
inventories/example/group_vars/all/services.yml
inventories/example/group_vars/web/php.yml
inventories/example/group_vars/web/vars.yml
inventories/example/group_vars/web/vault.yml.example
inventories/example/group_vars/worker/php.yml
inventories/vagrant/hosts.yml
playbooks/base.yml
playbooks/web.yml
playbooks/worker.yml
roles/common/
roles/php/
roles/services/
```

## Commands

Start an Ubuntu 22.04 Jammy VM and apply the web playbook:

```sh
vagrant up --provision
```

Inspect task order:

```sh
ansible-playbook -i inventories/example/hosts.yml playbooks/web.yml --list-tasks
ansible-playbook -i inventories/example/hosts.yml playbooks/worker.yml --list-tasks
```

Run only tagged work:

```sh
ansible-playbook -i inventories/example/hosts.yml playbooks/web.yml --tags php-modules
ansible-playbook -i inventories/example/hosts.yml playbooks/web.yml --tags php-pools
ansible-playbook -i inventories/example/hosts.yml playbooks/worker.yml --tags services
```

Create the encrypted secret file:

```sh
ansible-vault create inventories/example/group_vars/web/vault.yml
```

Add this variable to the encrypted file:

```yaml
vault_dummy_api_key: real-secret-value
```

Concrete package lists, PHP module tuples, pool definitions, managed INI files, and service state live under `inventories/example/group_vars/`. The roles keep empty or derived defaults so the inventory owns the deployed data.

`base.yml` gathers facts explicitly. The common role asserts the host is Ubuntu Jammy, renders `/etc/example-app/facts.ini` from facts like `ansible_distribution_release` and `ansible_memtotal_mb`, and PHP memory limits are selected from `ansible_memtotal_mb`.

PHP module INI templates and module symlinks notify the `Restart php-fpm` handler. Pool changes notify `Reload php-fpm`, which is usually less disruptive.

The `curl` module example renders `dummy.api_key={{ dummy_api_key }}` from optional module config. `dummy_api_key` is assigned from `vault_dummy_api_key`, which should live in an encrypted `inventories/example/group_vars/web/vault.yml`.

The example inventory targets remote-looking hosts like `web.example.com` and `worker.example.com`. The Vagrantfile uses a separate `inventories/vagrant/hosts.yml` inventory with `ansible_connection: local`, installs Ansible inside the Jammy VM, and passes `dummy_api_key=vagrant-dummy-secret` as an extra var so the VM can provision without a vault password prompt.

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "web.example.com"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 1
  end

  config.vm.provision "ansible_local" do |ansible|
    ansible.install = true
    ansible.playbook = "playbooks/web.yml"
    ansible.inventory_path = "inventories/vagrant/hosts.yml"
    ansible.limit = "web"
    ansible.extra_vars = {
      "dummy_api_key" => "vagrant-dummy-secret",
      "vault_dummy_api_key" => "vagrant-dummy-secret"
    }
    ansible.raw_arguments = [
      "-e", "@/vagrant/inventories/example/group_vars/all/common.yml",
      "-e", "@/vagrant/inventories/example/group_vars/all/services.yml",
      "-e", "@/vagrant/inventories/example/group_vars/web/php.yml"
    ]
  end
end

# vi: set ft=ruby

VAGRANTFILE_API_VERSION = "2"

SSH_PORT = 2232

Vagrant.require_version ">= 1.5.0"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.box = "chef/ubuntu-14.04"

    # awkward fix for SSH reassignment issue (re-evaluate w/ Vagrant 1.5.4)
    config.vm.network :forwarded_port, guest: 22, host: 2222, id: "ssh", disabled: true
    config.vm.network :forwarded_port, guest: 22, host: SSH_PORT, auto_correct: true

    config.vm.define "site" do |site|
        site.vm.network "private_network", ip: "10.42.2.100"

        site.vm.synced_folder "../opencivicdata.org", "/projects/opencivicdata.org/src/opencivicdata.org"

        site.vm.provider "virtualbox" do |v|
            v.name = "opencivicdata.org"
        end

        site.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/site.yml"
            ansible.inventory_path = "ansible/hosts.vagrant"
            ansible.limit = "all"
            # needed for common tasks to avoid EBS & checkout over synced_folders
            ansible.extra_vars = { deploy_type: "vagrant" }
            # seems to avoid the delay with private IP not being available
            ansible.raw_arguments = ["-T 30"]
        end
    end

    config.vm.define "db" do |db|
        db.vm.network "private_network", ip: "10.42.2.101"

        db.vm.provider "virtualbox" do |v|
            v.memory = 1024
            v.name = "db.opencivicdata.org"
        end

        db.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/db.yml"
            ansible.inventory_path = "ansible/hosts.vagrant"
            ansible.limit = "all"
            # needed for common tasks to avoid EBS & checkout over synced_folders
            ansible.extra_vars = { deploy_type: "vagrant" }
            # seems to avoid the delay with private IP not being available
            ansible.raw_arguments = ["-T 30"]
        end
    end

    config.vm.define "api" do |api|
        api.vm.network "private_network", ip: "10.42.2.102"

        api.vm.synced_folder "../api.opencivicdata.org", "/projects/api.opencivicdata.org/src/api.opencivicdata.org", mount_options: ["dmode=777,fmode=666"]
        api.vm.synced_folder "../imago", "/projects/api.opencivicdata.org/src/imago"

        api.vm.provider "virtualbox" do |v|
            v.name = "api.opencivicdata.org"
        end

        api.vm.provision "ansible" do |ansible|
            ansible.playbook = "ansible/api.yml"
            ansible.inventory_path = "ansible/hosts.vagrant"
            ansible.limit = "all"
            # needed for common tasks to avoid EBS & checkout over synced_folders
            ansible.extra_vars = { deploy_type: "vagrant" }
            # seems to avoid the delay with private IP not being available
            ansible.raw_arguments = ["-T 30"]
        end
    end
end

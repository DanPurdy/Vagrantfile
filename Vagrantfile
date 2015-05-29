Vagrant.configure("2") do |config|

    # VirtualBox defaults
    config.vm.provider "virtualbox" do |v|
        v.gui = false
        v.name = "Loft_Box_Precise64"
        v.memory = 2048
        v.cpus = 2
    end

    # Base box
    config.vm.box = "precise64"

    # Base box url
    config.vm.box_url = "http://files.vagrantup.com/precise64.box"

    # Port forwarding
    config.vm.network "forwarded_port", guest: 80, host: 8080 # HTTP
    config.vm.network "forwarded_port", guest: 443, host: 4443 # HTTPS

    # Public network (allows access on network)
    config.vm.network "public_network", type: "dhcp", bridge: 'en0: Wi-Fi (AirPort)'

    # Private network (required for nfs)
    config.vm.network "private_network", ip: "172.28.128.3"

    # Set synced folders
    config.vm.synced_folder "~/vagrant/www", "/www", type: "nfs"

    # Chef scripts
    config.vm.provision "chef_solo" do |chef|
        chef.cookbooks_path = "../chef/cookbooks"
        chef.roles_path = "../chef/roles"
        chef.data_bags_path = "../chef/databags"
        chef.add_role "devbox"
    end

    # Launch apache on startup
    config.vm.provision "shell", inline: "service apache2 restart", run: "always"

    # Enable port forwarding on startup
    config.trigger.after [:provision, :up, :reload] do
        system(
            'echo "
                rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 80 -> 127.0.0.1 port 8080
                rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 443 -> 127.0.0.1 port 4443
            " | sudo pfctl -ef - > /dev/null 2>&1; echo "==> Fowarding Ports: 80 -> 8080, 443 -> 4443 & Enabling pf"'
        )
    end

    # Disable port forwarding on shutdown
    config.trigger.after [:halt, :destroy] do
        system("sudo pfctl -df /etc/pf.conf > /dev/null 2>&1; echo '==> Removing Port Forwarding & Disabling pf'")
    end

end

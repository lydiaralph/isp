# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/trusty64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
   config.vm.provider "virtualbox" do |vb|
     # Display the VirtualBox GUI when booting the machine
     vb.gui = true
  
     # Customize the amount of memory on the VM:
     vb.memory = "1024"
   end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.

   # LER Note: commands copied from https://www.upcloud.com/support/installing-snort-on-ubuntu/
   config.vm.provision "shell", inline: <<-SHELL
     source /vagrant/properties.env
     apt-get update
     apt-get upgrade
     apt-get install lubuntu-desktop -y
     apt-get install -y xfce4 virtualbox-guest-dkms virtualbox-guest-utils virtualbox-guest-x11
     sed -i 's/allowed_users=.*$/allowed_users=anybody/' /etc/X11/Xwrapper.config
     
     apt install -y gcc libpcre3-dev zlib1g-dev libpcap-dev openssl libssl-dev 
     apt install libnghttp2-dev 
     apt install -y libdumbnet-dev bison flex libdnet
     mkdir ~/snort_src && cd ~/snort_src

     wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
     tar xvzf daq-2.0.6.tar.gz
     cd daq-2.0.6
     ./configure && make && make install

     cd ~/snort_src
     wget https://www.snort.org/downloads/snort/snort-2.9.11.1.tar.gz
     tar xvzf snort-2.9.11.1.tar.gz && cd snort-2.9.11.1
     ./configure --enable-sourcefire && make && make install

     ldconfig
     ln -s /usr/local/bin/snort /usr/sbin/snort
     groupadd snort
     useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort

     mkdir -p /etc/snort/rules
     mkdir /var/log/snort
     mkdir /usr/local/lib/snort_dynamicrules
     chmod -R 5775 /etc/snort
     chmod -R 5775 /var/log/snort
     chmod -R 5775 /usr/local/lib/snort_dynamicrules
     chown -R snort:snort /etc/snort
     chown -R snort:snort /var/log/snort
     chown -R snort:snort /usr/local/lib/snort_dynamicrules

     touch /etc/snort/rules/white_list.rules
     touch /etc/snort/rules/black_list.rules
     touch /etc/snort/rules/local.rules

     cp ~/snort_src/snort-2.9.11.1/etc/*.conf* /etc/snort
     cp ~/snort_src/snort-2.9.11.1/etc/*.map /etc/snort
     sed -i 's/include \$RULE\_PATH/#include \$RULE\_PATH/' /etc/snort/snort.conf
     sed -i 's/ ..\/rules/ etc\/snort\/rules/g' /etc/snort/snort.conf
     sed -i 's/ ..\/so_rules/ \/etc\/snort\/so_rules/g' /etc/snort/snort.conf
     sed -i 's/ ..\/preproc_rules/ \/etc\/snort\/preproc_rules/g' /etc/snort/snort.conf

     wget https://www.snort.org/rules/community -O ~/community.tar.gz
     tar -xvf ~/community.tar.gz -C ~/
     cp ~/community-rules/* /etc/snort/rules

     wget https://www.snort.org/rules/snortrules-snapshot-29111.tar.gz?oinkcode=$OINKCODE -O ~/registered.tar.gz
     tar -xvf ~/registered.tar.gz -C /etc/snort

     cp /vagrant/lydiaralph.rules /etc/snort/rules/
     echo "include \\$RULE_PATH/lydiaralph.rules" >> /etc/snort/snort.conf

     echo "colorscheme desert" > ~/.vimrc

     cp /vagrant/bash_config.env ~/.bash_profile
     source ~/.bash_profile

   SHELL
end

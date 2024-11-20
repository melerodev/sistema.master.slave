Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  # Máquina maestro
  config.vm.define "tierra" do |tierra|
    tierra.vm.hostname = "tierra.sistema.test"
    tierra.vm.network "private_network", ip: "192.168.57.103"
    tierra.vm.provider "virtualbox" do |vb|
      vb.name = "DNS-Maestro"
    end
    tierra.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install bind9 -y
      cp -v /vagrant/named /etc/default/named
      cp -v /vagrant/named.conf.options /etc/bind/named.conf.options
      cp -v /vagrant/named.conf.local /etc/bind/named.conf.local
      cp -v /vagrant/db.sistema.test /etc/bind/db.sistema.test
      cp -v /vagrant/db.192.168.57 /etc/bind/db.192.168.57
      systemctl restart named
      systemctl status named
    SHELL
  end

  # Máquina esclavo
  config.vm.define "venus" do |venus|
    venus.vm.hostname = "venus.sistema.test"
    venus.vm.network "private_network", ip: "192.168.57.102"
    venus.vm.provider "virtualbox" do |vb|
      vb.name = "DNS-Esclavo"
    end
    venus.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install bind9 -y
    SHELL
  end
end

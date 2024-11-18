Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"

  # Máquina maestro
  config.vm.define "tierra" do |tierra|
    tierra.vm.hostname = "tierra.sistema.test"
    tierra.vm.network "private_network", ip: "192.168.57.103"
    tierra.vm.provider "virtualbox" do |vb|
      vb.name = "DNS-Maestro"
    end
  end

  # Máquina esclavo
  config.vm.define "venus" do |venus|
    venus.vm.hostname = "venus.sistema.test"
    venus.vm.network "private_network", ip: "192.168.57.102"
    venus.vm.provider "virtualbox" do |vb|
      vb.name = "DNS-Esclavo"
    end
  end
end

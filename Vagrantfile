Vagrant.configure("2") do |config|
  config.vm.box = "debian/bookworm64"

  config.vm.hostname = "debian-vagrant"

  config.vm.network "private_network", ip: "192.168.56.10"

  config.vm.provider "virtualbox" do |vb|
  end
end
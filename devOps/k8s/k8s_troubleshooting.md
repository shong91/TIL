1. Vagrantfile check
   1-1.
   config.vm.provider :virtualbox do |vb|
   vb.gui = true
   end

1-2. add config options
config.vbguest.installer_options = { allow_kernel_upgrade: true }

1-3. plugin version check
vagrant plugin uninstall vagrant-vbguest vagrant plugin install vagrant-vbguest --plugin-version 0.21

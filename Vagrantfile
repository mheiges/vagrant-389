vagrant_root = File.dirname(__FILE__)

SERVERS = {
  :dsa => {
    :vagrant_box => "ebrc/centos-7-64-puppet",
    :hostname    => 'dsa.vm.apidb.org',
  },
  :dsb => {
   :vagrant_box  => "ebrc/centos-7-64-puppet",
    :hostname    => 'dsb.vm.apidb.org',
  },
}

Vagrant.configure(2) do |config|

  config.ssh.forward_agent = 'true'


  SERVERS.each do |name,cfg|
    config.vm.define name do |vm_config|

      vm_config.vm.box      = cfg[:vagrant_box] if cfg[:vagrant_box]
      vm_config.vm.hostname = cfg[:hostname] if cfg[:hostname]

      vm_config.vm.network :private_network, type: 'dhcp'
      if Vagrant.has_plugin?('landrush')
        vm_config.landrush.enabled = true
        vm_config.landrush.tld = 'vm.apidb.org'
      end

      vm_config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
        v.customize ["modifyvm", :id, "--ioapic", "on"]
      end

      vm_config.vm.provision "file",
        source: "#{vagrant_root}/files/setupLdapInstance",
        destination: "~/setupLdapInstance"


      vm_config.vm.synced_folder "scratch/puppetlabs/code/", "/etc/puppetlabs/code/", owner: "root", group: "root" 

    end # config.vm.define
  end # SERVERS.each

end

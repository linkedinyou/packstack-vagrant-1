VAGRANTFILE_API_VERSION = "2"
HOSTNAME = "packstack.dev"
BOX_URL = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos65-x86_64-20140116"

  config.vm.box_url = BOX_URL
  config.vm.hostname = HOSTNAME

  config.vm.network "public_network"

  config.vm.provider "virtualbox" do |v|
    v.customize ["modifyvm", :id, "--cpus", 4]
    v.customize ["modifyvm", :id, "--hwvirtex", "on"]
    v.customize ["modifyvm", :id, "--ioapic", "on"]
    v.customize ["modifyvm", :id, "--memory", 4096]
    v.customize ["modifyvm", :id, "--pae", "on"]
  end

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
    # nfs doesnt work on bridged networks
    #  config.cache.synced_folder_opts = {
    #    type: :nfs,
    #    mount_options: ['rw', 'vers=3', 'tcp', 'nolock']
    #  }
  end

  $script = <<SCRIPT
    # packstack sshes back into the vagrant instance with this key
    [ ! -f ~/.ssh/id_rsa ] && sudo ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
    sudo cat ~/.ssh/id_rsa.pub > ~/.ssh/authorized_keys

    # iptables exits with 6 if this file doesn't exist and the packstack recipes fail
    sudo touch /etc/sysconfig/iptables

    sudo yum update -y

    sudo yum install -y http://rdo.fedorapeople.org/rdo-release.rpm
    sudo yum install -y openstack-packstack vim
    sudo yum install "openstack-heat-*" python-heatclient

    sudo packstack --allinone                     \
                   --os-heat-install=y            \
                   --os-heat-cloudwatch-install=y \
                   --os-heat-cfn-install=y

    sudo sed -i 's/^virt_type=kvm/virt_type=qemu/g' /etc/nova/nova.conf
    sudo service openstack-nova-compute restart

    date | sudo tee /etc/vagrant_provisioned_at
SCRIPT

  config.vm.provision "shell", inline: $script
end

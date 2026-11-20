
MACHINES = {
  :router1 => {
    :box_name => "ubuntu/focal64",
    :vm_name => "router1",
    :net => [
      {ip: '10.0.10.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
      {ip: '10.0.12.1', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
      {ip: '192.168.10.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net1"},
      {ip: '192.168.250.10', adapter: 5}
    ]
  },

  :router2 => {
    :box_name => "ubuntu/focal64",
    :vm_name => "router2",
    :net => [
      {ip: '10.0.10.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r1-r2"},
      {ip: '10.0.11.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
      {ip: '192.168.20.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net2"},
      {ip: '192.168.250.11', adapter: 5}
    ]
  },

  :router3 => {
    :box_name => "ubuntu/focal64",
    :vm_name => "router3",
    :net => [
      {ip: '10.0.11.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "r2-r3"},
      {ip: '10.0.12.2', adapter: 3, netmask: "255.255.255.252", virtualbox__intnet: "r1-r3"},
      {ip: '192.168.30.1', adapter: 4, netmask: "255.255.255.0", virtualbox__intnet: "net3"},
      {ip: '192.168.250.12', adapter: 5}
    ]
  }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.hostname = boxconfig[:vm_name]

      box.vm.network "private_network" if false

      # Копируем открытый ключ на каждый роутер
      box.vm.provision "file",
        source: "ansible/keys/lab_ed25519.pub",
        destination: "/tmp/lab_ed25519.pub"

      box.vm.provision "shell", inline: <<-SHELL
        mkdir -p /home/vagrant/.ssh
        touch /home/vagrant/.ssh/authorized_keys
        grep -qxF "$(cat /tmp/lab_ed25519.pub)" /home/vagrant/.ssh/authorized_keys || \
          cat /tmp/lab_ed25519.pub >> /home/vagrant/.ssh/authorized_keys
        chown -R vagrant:vagrant /home/vagrant/.ssh
        chmod 700 /home/vagrant/.ssh
        chmod 600 /home/vagrant/.ssh/authorized_keys
        rm -f /tmp/lab_ed25519.pub
      SHELL

      if boxconfig[:vm_name] == "router3"
        # Закрытый ключ хранится только на router3
        box.vm.provision "file",
          source: "ansible/keys/lab_ed25519",
          destination: "/home/vagrant/.ssh/lab_ed25519"

        box.vm.provision "shell", inline: <<-SHELL
          chown vagrant:vagrant /home/vagrant/.ssh/lab_ed25519
          chmod 600 /home/vagrant/.ssh/lab_ed25519
        SHELL

        # Ansible устанавливается и запускается на router3
        box.vm.provision "ansible_local" do |ansible|
          ansible.playbook = "ansible/provision.yml"
          ansible.inventory_path = "ansible/hosts"
          ansible.install = true
          ansible.limit = "routers"
        end
      end

      boxconfig[:net].each do |ipconf|
        box.vm.network "private_network", **ipconf
      end
    end
  end
end

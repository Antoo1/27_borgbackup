# -*- mode: ruby -*-
# vim: set ft=ruby :
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :"client" => {
              :box_name => "generic/ubuntu2204",
              :cpus => 1,
              :memory => 1024,
              :ip => "192.168.56.111",
              :disks => {},
            },
  :"backupserver" => {
              :box_name => "generic/ubuntu2204",
              :cpus => 1,
              :memory => 1024,
              :ip => "192.168.56.112",
              :disks => {
                :sata1 => {
                    :size => "1G",
                    :port => 1
                },
            },
          },
}
  
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.define boxname do |box|
      box.vm.network "private_network", ip: boxconfig[:ip]
      box.vm.host_name = boxname
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]

      box.vm.provision "shell", inline: <<-SHELL
        sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication no/' $(grep -r 'PasswordAuthentication' /etc/ssh/ | awk '{print $1}' FS=':' | uniq)
        systemctl restart sshd.service
        SHELL

      box.vm.provider "libvirt" do |vm|
        vm.memory = boxconfig[:memory]
        vm.cpus = boxconfig[:cpus]
        
        boxconfig[:disks].each do |dname, dconf|
          vm.storage :file, :size=> dconf[:size], :device => "sda", :allow_existing => true, :bus => "sata"
        end
      end
      box.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
        ansible.become = "true"
      end
    end
  end
end

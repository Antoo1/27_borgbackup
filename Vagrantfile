# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :"backupserver" => {
              :box_name => "generic/ubuntu2204",
              :cpus => 1,
              :memory => 1024,
              :ip => "192.168.56.111",
              :disks => {},
            },
  :"client" => {
              :box_name => "generic/ubuntu2204",
              :cpus => 1,
              :memory => 1024,
              :ip => "192.168.56.112",
              :disks => {
                :sata1 => {
                    :dfile => home + '/disk_d/vms/sata1.vdi',
                    :size => "1G",
                    :port => 1
                },
            },
          },
}

ssh_key = `ssh-keygen -t ed25519 -f /tmp/id_ed25519 -N ""`
public_key = File.read("/tmp/id_ed25519.pub")
private_key = File.read("/tmp/id_ed25519")
  
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.define boxname do |box|
      box.vm.network "private_network", ip: boxconfig[:ip]
      box.vm.host_name = boxname
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s

      box.vm.provider "libvirt" do |vm|
        vm.memory = boxconfig[:memory]
        vm.cpus = boxconfig[:cpus]
        
        box.vm.provision "shell", inline: <<-SHELL
          sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication no/' $(grep -r 'PasswordAuthentication' /etc/ssh/ | awk '{print $1}' FS=':' | uniq)
          systemctl restart sshd.service
          useradd -m -s /bin/bash borg
          echo borg:'Otus2022!' | sudo chpasswd
          groupadd -f admin
          usermod borg -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin
          mkdir /home/borg/.ssh
          echo '#{private_key}' >> /home/borg/.ssh/id_ed25519
          chown borg /home/borg/.ssh/ 
          chmod 700 /home/borg/.ssh/
        SHELL
        
        boxconfig[:disks].each do |dname, dconf|
          vm.storage :file, :size=> dconf[:size], :path => dconf[:path], :device => "hdd", :allow_existing => true
          box.vm.provision "shell", inline: <<-SHELL
            dd if=/dev/zero of=/dev/vdb bs=1M count=1024
            mkfs.ext4 /dev/vdb
            mkdir -p /var/backup
            mount /dev/sdb /var/backup
            chown borg:borg /var/backup/
            echo '#{public_key}' >> /home/borg/.ssh/authorized_keys
            chmod 600 /home/borg/.ssh/authorized_keys
          SHELL
        end
      end
      box.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
        ansible.become = "true"
      end
    end
  end
end

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

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    config.vm.define boxname do |box|
      box.vm.network "private_network", ip: boxconfig[:ip]
      box.vm.host_name = boxname
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s

      box.vm.provider "libvirt" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
        
        boxconfig[:disks].each do |dname, dconf|
          v.storage :file, :size=> dconf[:size], :path => dconf[:path], :device => "hdd", :allow_existing => true
        #   unless File.exist?(dconf[:dfile])
        #     v.storage :file, size: dconf[:size]
        #     # v.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
        #     needsController =  true
        #   end
        #   if needsController == true
        #     v.storage_controller :sata, port_count: 1, model_type: 'virtio-scsi' # Changed storage controller for libvirt
        #     # v.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
        #     boxconfig[:disks].each do |dname, dconf|
        #       v.storage_attach :disk, device: 0, disk: dconf[:dfile], port: dconf[:port], bus: 'sata' # Changed storage attachment for libvirt
        #       # v.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
        #     end
        #   end
        end
      end
      box.vm.provision "shell", inline: <<-SHELL
          sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication no/' $(grep -r 'PasswordAuthentication' /etc/ssh/ | awk '{print $1}' FS=':' | uniq)
          systemctl restart sshd.service
      SHELL
    end
  end
end

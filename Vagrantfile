# Additional disk size
DISK_SIZE = 1


Vagrant.configure("2") do |configure|                                                                                                  
  (1..2).each do |i|                                                                                                                
    configure.vm.define "node#{i}" do |config|                                                                                          
      config.vm.box = "generic/ubuntu2004"                                                                                                
      config.vm.network "private_network", ip: "192.168.122.1#{i}",
        :dev => "virbr1",                                                                                                               
        :libvirt_network_name => "private",                                                                                             
        :type => "bridge"
      config.vm.hostname = "pacemaker#{i}.example.net"
   

      config.vm.provider :libvirt do |libvirt|                                                                                          
        libvirt.cpus = 1                                                                                                                
        libvirt.driver = "kvm"                                                                                                          
        libvirt.id_ssh_key_file = "$HOME/.ssh/id_ed25519"                                                                               
        libvirt.keymap = "ru"                                                                                                           
        libvirt.machine_virtual_size = 20                                                                                               
        libvirt.memory = 1024                                                                                                           
        libvirt.storage_pool_name = "default"                                                                                           
        libvirt.uri = "qemu:///system"
        libvirt.storage :file, :size => DISK_SIZE, :type => 'qcow2', :path => "storage#{i}-vdb.qcow2"                                           
      end

      # vm provisioning: disk partitioning
      config.vm.provision "shell", env: {"DISK_SIZE" => DISK_SIZE}, inline: <<-SCRIPT
        set -x
        DISK="/dev/vdb"
        BASE="$( /usr/bin/basename $DISK )"
        SIZE="$( /usr/bin/lsblk | /usr/bin/grep -E "$BASE.*252:16" | /usr/bin/awk '{print$4}' | /usr/bin/cut -c1 )"
        if [ "$SIZE" != "$DISK_SIZE" ]; then
          echo "Something is wrong with your additional disk!"
          exit 1
        fi
        /usr/bin/cat <<EOF | /usr/sbin/fdisk $DISK
        n
        p
        1


        t
        83
        w
        EOF
        /usr/sbin/partprobe $DISK
        set +x
      SCRIPT


      # vm provisioning: by ansible
      config.vm.provision "ansible" do |ansible|
        ansible.playbook = "playbook.yml"
      end
    end
  end                                                                                                                               
end 

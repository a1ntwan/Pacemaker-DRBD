require 'fileutils'

# file operations needs to be relative to this file
VAGRANT_ROOT = File.dirname(File.expand_path(__FILE__))

# directory that will contain VDI files
VAGRANT_DISKS_DIRECTORY = "disks"

# controller type definition
VAGRANT_CONTROLLER_TYPE = "virtio-scsi"

# Additional disk size
DISK_SIZE = 1


Vagrant.configure("2") do |configure|
  (1..2).each do |i|
  # define disks
  # The format is filename, size (GB), port (see controller docs)
  local_disks = [
    { :filename => "disk#{i}", :size => DISK_SIZE, :port => 0 },
  ]
  # controller name definition
  vagrant_controller_name = "Virtual I/O Device SCSI controller-#{i}"

    configure.vm.define "node#{i}" do |config|
      config.vm.box = "generic/centos7"
      #config.vm.box = "ubuntu/focal64"

      # Extra vm options
      config.vm.network "private_network", ip: "10.10.10.2#{i}"
      config.vm.hostname = "pacemaker#{i}.example.net"
      config.vm.boot_timeout = 600

      disks_directory = File.join(VAGRANT_ROOT, VAGRANT_DISKS_DIRECTORY)

      unless File.directory?(disks_directory)
        config.vm.provider "virtualbox" do |storage_provider|
          storage_provider.customize ["storagectl", :id, "--name", vagrant_controller_name, "--add", VAGRANT_CONTROLLER_TYPE, '--hostiocache', 'off']
        end
      end

      # create disks before "up" action
      config.trigger.before :up do |trigger|
        trigger.name = "Create disks"
        trigger.ruby do
          unless File.directory?(disks_directory)
            FileUtils.mkdir_p(disks_directory)
          end
          local_disks.each do |local_disk|
            local_disk_filename = File.join(disks_directory, "#{local_disk[:filename]}.vdi")
            unless File.exist?(local_disk_filename)
              puts "Creating \"#{local_disk[:filename]}\" disk"
              system("vboxmanage createmedium --filename #{local_disk_filename} --size #{local_disk[:size] * 1024} --format VDI")
            end
          end
        end
      end

      # attach storage devices
      config.vm.provider "virtualbox" do |storage_provider|
        local_disks.each do |local_disk|
          local_disk_filename = File.join(disks_directory, "#{local_disk[:filename]}.vdi")
          unless File.exist?(local_disk_filename)
            storage_provider.customize ['storageattach', :id, '--storagectl', vagrant_controller_name, '--port', local_disk[:port], '--device', 0, '--type', 'hdd', '--medium', local_disk_filename]
          end
        end
      end

      # cleanup after "destroy" action
      config.trigger.after :destroy do |trigger|
        trigger.name = "Cleanup operation"
        trigger.ruby do
          # the following loop is now obsolete as these files will be removed automatically as machine dependency
          local_disks.each do |local_disk|
            local_disk_filename = File.join(disks_directory, "#{local_disk[:filename]}.vdi")
            if File.exist?(local_disk_filename)
              puts "Deleting \"#{local_disk[:filename]}\" disk"
              system("vboxmanage closemedium disk #{local_disk_filename} --delete")
            end
            if File.exist?(disks_directory) && "#{i}" == "1"
              FileUtils.rmdir(disks_directory)
            end
          end
        end
      end

      # vm settings
      config.vm.provider "virtualbox" do |vb|
        vb.memory = "1024"
        vb.cpus = "1"
        vb.gui = false
      end

      # vm provisioning: disk partitioning
      config.vm.provision "shell", env: {"DISK_SIZE" => DISK_SIZE}, inline: <<-SCRIPT
        set -x
        DISK="/dev/sda"
        BASE="$( /usr/bin/basename $DISK )"
        SIZE="$( /usr/bin/lsblk | /usr/bin/grep -E "$BASE.*8:0" | /usr/bin/awk '{print$4}' | /usr/bin/cut -c1 )"
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

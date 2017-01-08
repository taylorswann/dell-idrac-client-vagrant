# -*- mode: ruby -*-
# vi: set ft=ruby :

# Launch a Fedora minimal desktop pre-configured to connect into Dell iDRACs
Vagrant.configure('2') do |config|
  config.vm.define :dell_idrac_client do |d|
    # base on fedora
    d.vm.box = 'fedora/25-cloud-base'

    # use a gui
    d.vm.provider :virtualbox do |vb|
       vb.gui = true
    end
    d.vm.provider :libvirt do |libvirt|
       libvirt.graphics_type = 'vnc'
    end
    d.ssh.forward_x11 = true

    # use all available cpu cores
    host = RbConfig::CONFIG['host_os']
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
    elsif host =~ /linux/
      cpus = `nproc`.to_i
    else
      cpus = 1
    end
    d.vm.provider :virtualbox do |vb|
      vb.customize ['modifyvm', :id, '--cpus', cpus]
    end
    d.vm.provider :libvirt do |libvirt|
      libvirt.cpus = cpus
    end

    # configure system for use with the java idrac client
    d.vm.provision 'shell', inline: <<-SHELL
       dnf -y --setopt=deltarpm=false groupinstall 'LXDE Desktop'
       dnf -y --setopt=deltarpm=false install firefox java-\*-openjdk.i\*86 icedtea-web
       sed -i 's@JAVA=.*@'"JAVA=$(rpm -qa java\*|grep i686|xargs rpm -ql|grep jre/bin/java)"'@' /usr/bin/javaws.itweb
       sed -i '/jdk.certpath.disabledAlgorithms/ s@^#*@#@' $(rpm -qa java\*|grep i686|xargs rpm -ql|grep jre/lib/security/java.security)
       sed -i '/jdk.tls.disabledAlgorithms/ s@^#*@#@' $(rpm -qa java\*|grep i686|xargs rpm -ql|grep jre/lib/security/java.security)
       sed -i 's@# autologin=.*@autologin=vagrant@' /etc/lxdm/lxdm.conf
       mkdir -p /home/vagrant/.config/autostart
       echo -e '[Desktop Entry]\nType=Application\nExec=firefox' > /home/vagrant/.config/autostart/firefox.desktop
       chown -R vagrant /home/vagrant/.config
       systemctl set-default runlevel5.target
       reboot
    SHELL
  end
end
# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:inetRouter => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        #:public => {:ip => '10.10.10.1', :adapter => 1},
        :net => [
                   {ip: '192.168.255.1', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                ]
  },
  :centralRouter => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
                   {ip: '192.168.255.2', adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {ip: '192.168.0.33', adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: true},
                   {ip: '192.168.0.65', adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: true},
                ]
  },
  
  :centralServer => {
        :box_name => "centos/7",
        :box_version => "2004.01",
        :net => [
                   {ip: '192.168.0.2', adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "dir-net"},
                   {adapter: 3, auto_config: false, virtualbox__intnet: true},
                   {adapter: 4, auto_config: false, virtualbox__intnet: true},
                ]
  },
  :office1Router => {
    :box_name => "centos/7",
    :box_version => "2004.01",
    :net => [
      {adapter:2, device:"eth1", auto_config: false, virtualbox__intnet: true},
      {adapter:3, device:"eth2", auto_config: false, virtualbox__intnet: true},
      {adapter:4, device:"eth3", auto_config: false, virtualbox__intnet: true},
      {adapter:5, device:"eth4", auto_config: false, virtualbox__intnet: true},
    ]
  },
  :office2Router => {
    :box_name => "centos/7",
    :box_version => "2004.01",
    :net => [
      {adapter:2, device:"eth1", auto_config: false, virtualbox__intnet: true},
      {adapter:3, device:"eth2", auto_config: false, virtualbox__intnet: true},
      {adapter:4, device:"eth3", auto_config: false, virtualbox__intnet: true},
    ]
  },
  :office1Server => {
    :box_name => "centos/7",
    :box_version => "2004.01",
    :net => [
      {adapter:2, auto_config: false, virtualbox__intnet: true}
    ]
  },
  :office2Server => {
    :box_name => "centos/7",
    :box_version => "2004.01",
    :net => [
      {adapter:2, auto_config: false, virtualbox__intnet: true}
    ]
  },
  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "192"]
          vb.customize ["modifyvm", :id, "--cpus", "1"]
        end
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "inetRouter"
          box.vm.provision "shell", inline: <<-SHELL
            sysctl net.ipv4.conf.all.forwarding=1
            iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
            # persistent settings
            echo "192.168.0.0/24 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
            echo "192.168.1.0/24 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
            echo "192.168.255.4/30 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
            echo "192.168.255.8/30 via 192.168.255.2 dev eth1" >> /etc/sysconfig/network-scripts/route-eth1
            # runtime settings
            ip route add 192.168.0.0/24 via 192.168.255.2 dev eth1
            ip route add 192.168.1.0/24 via 192.168.255.2 dev eth1
            ip route add 192.168.255.4/30 via 192.168.255.2 dev eth1
            ip route add 192.168.255.8/30 via 192.168.255.2 dev eth1
            SHELL
        when "centralRouter"
          box.vm.provision "shell", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.d/netforward.conf
            sysctl --system > /dev/null
            # persistent settings
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.255.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "DEVICE=eth3:1" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "IPADDR=192.168.255.6" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "NETMASK=255.255.255.252" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "192.168.2.0/24 via 192.168.255.5 dev eth3" > /etc/sysconfig/network-scripts/route-eth3
            echo "DEVICE=eth4:1" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "IPADDR=192.168.255.10" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "NETMASK=255.255.255.252" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "192.168.1.0/24 via 192.168.255.9 dev eth4" > /etc/sysconfig/network-scripts/route-eth4
            # runtime settings
            ip route delete default
            ip address add 192.168.255.6/30 dev eth3:1
            ip link set eth3:1 up
            ip address add 192.168.255.10/30 dev eth4:1
            ip link set eth4:1 up
            ip route add 192.168.1.0/24 via 192.168.255.9 dev eth4
            ip route add 192.168.2.0/24 via 192.168.255.5 dev eth3
            ip route add 0.0.0.0/0 via 192.168.255.1 dev eth1
            
            SHELL
          when "centralServer"
            box.vm.provision "shell", inline: <<-SHELL
            # persistent
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "GATEWAY=192.168.0.1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            # runtime
            ip route delete default
            ip route add default via 192.168.0.1 dev eth1
            
            SHELL
          when "office1Router"
            box.vm.provision "shell", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.d/netforward.conf
            sysctl --system > /dev/null
            # persistent
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "DEVICE=eth1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "IPADDR=192.168.255.5" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "GATEWAY=192.168.255.6" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "NETMASK=255.255.255.252" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "DEVICE=eth2" >> /etc/sysconfig/network-scripts/ifcfg-eth2
            echo "IPADDR=192.168.2.1" >> /etc/sysconfig/network-scripts/ifcfg-eth2
            echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth2
            echo "DEVICE=eth3" >> /etc/sysconfig/network-scripts/ifcfg-eth3
            echo "IPADDR=192.168.2.65" >> /etc/sysconfig/network-scripts/ifcfg-eth3
            echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth3
            echo "DEVICE=eth4" >> /etc/sysconfig/network-scripts/ifcfg-eth4
            echo "IPADDR=192.168.2.129" >> /etc/sysconfig/network-scripts/ifcfg-eth4
            echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth4
            echo "DEVICE=eth4:1" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "IPADDR=192.168.2.193" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth4:1
            # runtime settings
            ip route delete default
            ip address add 192.168.255.5/30 dev eth1
            ip link set eth1 up
            ip route add default via 192.168.255.6 dev eth1
            ip address add 192.168.2.1/26 dev eth2
            ip link set eth2 up
            ip address add 192.168.2.65/26 dev eth3
            ip link set eth3 up
            ip address add 192.168.2.129/26 dev eth4
            ip link set eth4 up
            ip address add 192.168.2.193/26 dev eth4:1
            ip link set eth4:1 up

          SHELL
        when "office1Server"
          box.vm.provision "shell", inline: <<-SHELL
          # peristent settings
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "DEVICE=eth1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "IPADDR=192.168.2.66" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "GATEWAY=192.168.2.65" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          # runtime settings
          ip route delete default
          ip address add 192.168.2.66/26 dev eth1
          ip link set eth1 up
          ip route add default via 192.168.2.65 dev eth1
          SHELL
        when "office2Router"
          box.vm.provision "shell", inline: <<-SHELL
            echo "net.ipv4.conf.all.forwarding=1" >> /etc/sysctl.d/netforward.conf
            sysctl --system > /dev/null
            # runtime settings
            echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
            echo "DEVICE=eth1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "IPADDR=192.168.255.9" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "NETMASK=255.255.255.252" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "GATEWAY=192.168.255.10" >> /etc/sysconfig/network-scripts/ifcfg-eth1
            echo "DEVICE=eth2" >> /etc/sysconfig/network-scripts/ifcfg-eth2
            echo "IPADDR=192.168.1.1" >> /etc/sysconfig/network-scripts/ifcfg-eth2
            echo "NETMASK=255.255.255.128" >> /etc/sysconfig/network-scripts/ifcfg-eth2
            echo "DEVICE=eth3" >> /etc/sysconfig/network-scripts/ifcfg-eth3
            echo "IPADDR=192.168.1.129" >> /etc/sysconfig/network-scripts/ifcfg-eth3
            echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth3
            echo "DEVICE=eth3:1" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "IPADDR=192.168.1.193" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth3:1
            # runtime settings
            ip route delete default
            ip address add 192.168.255.9/30 dev eth1
            ip link set eth1 up
            ip route add default via 192.168.255.10 dev eth1
            ip address add 192.168.1.1/25 dev eth2
            ip link set eth2 up
            ip address add 192.168.1.129/26 dev eth3
            ip link set eth3 up
            ip address add 192.168.1.193/26 dev eth3:1
            ip link set eth3:1 up            
          SHELL
        when "office2Server"
          box.vm.provision "shell", inline: <<-SHELL
          # persistent settings
          echo "DEFROUTE=no" >> /etc/sysconfig/network-scripts/ifcfg-eth0 
          echo "GATEWAY=192.168.1.129" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "DEVICE=eth1" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "IPADDR=192.168.1.130" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          echo "NETMASK=255.255.255.192" >> /etc/sysconfig/network-scripts/ifcfg-eth1
          # runtime settings
          ip route delete default
          ip address add 192.168.1.130/26 dev eth1
          ip link set eth1 up
          ip route add default via 192.168.1.129 dev eth1
          SHELL
        end

      end

  end
  
  
end

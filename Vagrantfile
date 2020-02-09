# -*- mode: ruby -*-
# vim: set ft=ruby :
# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :les7 => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            vb.customize ["modifyvm", :id, "--memory", "512"]
          end

          box.vm.provision "shell", inline: <<-SHELL
           mkdir -p ~root/.ssh
           cp ~vagrant/.ssh/auth* ~root/.ssh
           yum install epel-release -y -q
           yum install fish wget -y -q
# Install tools for building rpm
           yum install redhat-lsb-core rpmdevtools rpm-build createrepo yum-utils -y -q
           yum install tree mc wget gcc vim git -y -q
# Install tools for building woth mock and make prepares    
           yum install mock -y -q
           usermod -a -G mock root
# Install docker-ce
           sudo yum install -y -q yum-utils links \
           device-mapper-persistent-data \
           lvm2
           sudo yum-config-manager \
           --add-repo \
           https://download.docker.com/linux/centos/docker-ce.repo
           yum install docker-ce docker-compose -y -q
           systemctl start docker
           docker run hello-world
# Install nginx
           wget -P /root https://nginx.org/packages/centos/7/SRPMS/nginx-1.14.1-1.el7_4.ngx.src.rpm
           rpm -i /root/nginx-1.14.1-1.el7_4.ngx.src.rpm
           wget -P /root https://www.openssl.org/source/latest.tar.gz
           tar -C /root -xvf /root/latest.tar.gz
           
      SHELL

      end
  end
end


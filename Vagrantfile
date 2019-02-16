Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.2"
  config.vm.provider :virtualbox do |vb|
  vb.memory = "512"
  end
  config.vm.define "server1" do |server1|
    server1.vm.network :private_network, ip: "192.168.56.11"
    server1.vm.hostname = "server1"
    server1.vm.provision "shell",
      inline: <<-SHELL
        yum install -y java-1.8.0-openjdk tomcat tomcat-webapps tomcat-admin-webapps
        systemctl enable tomcat
        systemctl restart tomcat
        mkdir -p /usr/share/tomcat/webapps/test
        echo "server1" | tee /usr/share/tomcat/webapps/test/index.html > /dev/null
      SHELL
  end
  config.vm.define "server2" do |server2|
    server2.vm.network :private_network, ip: "192.168.56.22"
    server2.vm.hostname = "server2"
    server2.vm.provision "shell",
      inline: <<-SHELL
        yum install -y java-1.8.0-openjdk tomcat tomcat-webapps tomcat-admin-webapps
        systemctl enable tomcat
        systemctl restart tomcat
        mkdir -p /usr/share/tomcat/webapps/test
        echo "server2" | tee /usr/share/tomcat/webapps/test/index.html > /dev/null
      SHELL
  end
  config.vm.define "balans" do |balans|
    balans.vm.network :private_network, ip: "192.168.56.33"
    balans.vm.network :forwarded_port, guest: 80, host: 8080
    balans.vm.hostname = "balans"
    balans.vm.provision "shell",
      inline: <<-SHELL
        yum install -y httpd mc
        systemctl enable httpd
        systemctl restart httpd
         cp /vagrant/mod_jk.so /etc/httpd/modules/
		 echo "worker.list=lb\nworker.lb.type=lb\nworker.lb.balance_workers=worker1, worker2\nworker.worker1.host=server1\nworker.worker1.port=8009\nworker.worker1.type=ajp13\nworker.worker2.host=server2\nworker.worker2.port=8009\nworker.worker2.type=ajp13" > /etc/httpd/conf/workers.properties
          cat <<-EOT >> /etc/httpd/conf.d/httpd.conf 
LoadModule jk_module modules/mod_jk.so
JkWorkersFile conf/workers.properties
JkShmFile /tmp/shm
JkLogFile logs/mod_jk.log
JkLogLevel info
JkMount /tc* lb
				EOT
		systemctl restart httpd
		       SHELL
  end
  config.vm.provision "shell",
    inline: <<-SHELL
		echo -e "192.168.56.22 server2\n192.168.56.11 server1\n192.168.56.33 balans\n127.0.0.1 localhost" > /etc/hosts
               SHELL
end

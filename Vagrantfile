# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "comiq/robotbox"

  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  config.vm.synced_folder "../workspace", "/workspace"

  config.vm.provider "virtualbox" do |v|
    v.name = "comiq-robotbox_test"
    v.memory = 2048
    v.cpus = 2
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "90"]
  end

  config.vm.network "forwarded_port", guest: 9966, host: 9966

  config.trigger.after :up do
    run_remote "docker run -e MYSQL_ROOT_PASSWORD=petclinic -de MYSQL_DATABASE=petclinic -p 3306:3306 mysql:5.7.8"
    run_remote "cd Docker/xvfb-container/ && sudo docker build -t xvfb github.com/phavukai/xvfb_docker"
    run_remote  " cd Docker/xvfb-container/ && docker run -d xvfb "
    run_remote "cd spring-petclinic && ./mvnw tomcat7:run & "

  end

  config.vm.provision "shell", inline: <<-SHELL
     apt-get install -y default-jdk
     apt-get install -y git

     # Install and precompile PetClinic
     git clone https://github.com/spring-projects/spring-petclinic.git
     cd spring-petclinic
     ./mvnw package
     cd ..
     chown -R vagrant:vagrant spring-petclinic

     # Install Docker, Docker-compose and Docker-machine
     apt-get update
     apt-get install -y apt-transport-https ca-certificates
     apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D

     if ! grep -q "apt.dockerproject.org" /etc/apt/sources.list; then
       echo 'deb https://apt.dockerproject.org/repo ubuntu-trusty main' >> /etc/apt/sources.list
     fi

     apt-get update -y
     apt-get install -y docker-engine
     usermod -aG docker vagrant
     curl -L https://github.com/docker/compose/releases/download/1.7.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
     chmod +x /usr/local/bin/docker-compose
     curl -L https://github.com/docker/machine/releases/download/v0.6.0/docker-machine-`uname -s`-`uname -m` > /usr/local/bin/docker-machine
     chmod +x /usr/local/bin/docker-machine


     # Change database to MYSQL
     sudo cp -f /workspace/database-access.properties spring-petclinic/src/main/resources/spring/data-access.properties
     sudo cp -f /workspace/pom.xml spring-petclinic/pom.xml

     # Create dir for xvfb docker and set display
      sudo mkdir -p Docker/xvfb-container/
      echo "DISPLAY=172.17.0.3:0" >> /etc/environment


      #Install google-chrome and webdriver
      wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
      sudo dpkg -i --force-depends google-chrome-stable_current_amd64.deb
      LATEST=$(wget -q -O - http://chromedriver.storage.googleapis.com/LATEST_RELEASE)
      wget http://chromedriver.storage.googleapis.com/$LATEST/chromedriver_linux64.zip
      sudo apt-get update
      sudo apt-get install -f -y
      sudo apt-get install -y unzip
      unzip chromedriver_linux64.zip && sudo ln -s $PWD/chromedriver /usr/local/bin/chromedriver

      # Install robotframework/python depencies
      sudo pip3 install robotframework-databaselibrary
      sudo pip3 install pymysql


       SHELL

end

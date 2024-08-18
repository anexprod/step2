VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.boot_timeout = 600  

  config.vm.define "jenkins-master" do |master|
    master.vm.box = "ubuntu/bionic64"
    master.vm.hostname = "jenkins-master"
    master.vm.network "private_network", ip: "192.168.33.15"
    master.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end

    master.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y openjdk-11-jdk
      curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
      echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
      sudo apt-get update
      sudo apt-get install -y jenkins
      sudo sed -i 's/^HTTP_PORT=.*/HTTP_PORT=8081/' /etc/default/jenkins
      sudo systemctl restart jenkins
      sudo systemctl enable jenkins
      sudo apt-get install -y openssh-server
      sudo systemctl start ssh
      sudo systemctl enable ssh
    SHELL

    master.vm.provision "shell", inline: <<-SHELL
      wget http://localhost:8081/jnlpJars/jenkins-cli.jar
      java -jar jenkins-cli.jar -s http://localhost:8081/ install-plugin git docker-plugin -deploy
    SHELL
  end

  config.vm.define "jenkins-worker" do |worker|
    worker.vm.box = "ubuntu/bionic64"
    worker.vm.hostname = "jenkins-worker"
    worker.vm.network "private_network", ip: "192.168.33.16"
    worker.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.cpus = 1
    end

    worker.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y curl
      curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
      sudo apt-get install -y nodejs
      sudo apt-get install -y build-essential
      sudo apt-get install -y docker.io
      sudo systemctl start docker
      sudo systemctl enable docker
      sudo usermod -aG docker vagrant
    SHELL
  end
end

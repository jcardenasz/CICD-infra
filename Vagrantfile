Vagrant.configure("2") do |config|
  config.vm.box = "jcardenasz/ubuntu-docker"
  config.vm.hostname = "jenkins"

  # Access Jenkins UI from your host at http://localhost:8080
  config.vm.network "forwarded_port", guest: 8080, host: 8080, auto_correct: true
  # Optional: JNLP inbound agents port (we can also use SSH agents)
  config.vm.network "forwarded_port", guest: 50000, host: 50000, auto_correct: true

  config.vm.provider "virtualbox" do |vb|
    vb.name = "jenkins-controller"
    vb.cpus = 2
    vb.memory = 4096
  end

  config.vm.provision "shell", inline: <<-'SHELL'
    set -eux

    # --- Jenkins container (LTS, JDK17) ---
    sudo mkdir -p /var/jenkins_home
    sudo chown 1000:1000 /var/jenkins_home

    # If re-provisioning, remove an existing container
    docker rm -f jenkins || true

    # Run Jenkins. We DO NOT mount docker.sock yet (we'll build on the agent VM).
    docker run -d --name jenkins \
      -p 8080:8080 \
      -p 50000:50000 \
      -v /var/jenkins_home:/var/jenkins_home \
      jenkins/jenkins:lts-jdk17

    echo "Jenkins is starting. First-time admin password will be in /var/jenkins_home/secrets/initialAdminPassword"

    # ----------- VM tunnel setup --------------
    curl -Lk 'https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64' --output vscode_cli.tar.gz
    tar -xf vscode_cli.tar.gz
  SHELL
end

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

    # Pass your env vars (for JCasC ${...} resolution) into the Jenkins container
    if [ -f /vagrant/.env ]; then
      sudo cp /vagrant/.env /var/jenkins_home/.env
      sudo chown 1000:1000 /var/jenkins_home/.env
    else
      echo "WARNING: /vagrant/.env not found. JCasC variables will be empty."
      sudo sh -lc 'echo JENKINS_ADMIN_PASSWORD=changeme > /var/jenkins_home/.env'
      sudo chown 1000:1000 /var/jenkins_home/.env
    fi

    # If re-provisioning, remove an existing container
    docker rm -f jenkins || true

    # Run Jenkins with JCasC enabled and the casc dir mounted read-only
    docker run -d --name jenkins \
      -p 8080:8080 \
      -p 50000:50000 \
      --env-file /var/jenkins_home/.env \
      -v /var/jenkins_home:/var/jenkins_home \
      -v /vagrant/casc:/var/jenkins_home/casc:ro \
      -e CASC_JENKINS_CONFIG=/var/jenkins_home/casc/jenkins.yaml \
      jenkins/jenkins:lts-jdk17

    echo "Jenkins is starting. Initial admin password (if JCasC admin not applied yet) at /var/jenkins_home/secrets/initialAdminPassword"

    # Install required plugins so JCasC can apply cleanly, then restart Jenkins
    sleep 20
    docker exec jenkins jenkins-plugin-cli --plugins \
      configuration-as-code \
      ssh-slaves \
      credentials \
      git \
      github \
      workflow-aggregator

    docker restart jenkins

    echo "Jenkins restarted to load plugins & apply JCasC"
  SHELL
end

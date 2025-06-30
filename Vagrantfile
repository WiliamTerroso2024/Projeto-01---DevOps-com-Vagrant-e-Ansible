# -*- mode: ruby -*-
# vi: set ft=ruby :

# Caminho e leitura da chave pública SSH
ssh_pub_key_path = File.expand_path("~/.ssh/id_rsa.pub")
ssh_public_key = File.read(ssh_pub_key_path).strip

# Função para configurar Ansible provisioner
def configure_ansible(ansible, playbook, limit)
  ansible.playbook = playbook
  ansible.become = true
  ansible.limit = limit
  ansible.compatibility_mode = "2.0"
end

Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
  end

  # Provisionamento shell comum
  config.vm.provision "shell", inline: <<-SHELL
    sleep 10
    apt-get update -y
    apt-get install -y python3 python3-pip openssh-server

    if ! id "wiliam" &>/dev/null; then
      useradd -m -s /bin/bash wiliam
      echo "wiliam:wiliam" | chpasswd
      usermod -aG sudo wiliam
    fi

    mkdir -p /home/wiliam/.ssh
    echo "#{ssh_public_key}" > /home/wiliam/.ssh/authorized_keys
    chmod 700 /home/wiliam/.ssh
    chmod 600 /home/wiliam/.ssh/authorized_keys
    chown -R wiliam:wiliam /home/wiliam/.ssh

    mkdir -p /root/.ssh
    echo "#{ssh_public_key}" > /root/.ssh/authorized_keys
    chmod 700 /root/.ssh
    chmod 600 /root/.ssh/authorized_keys

    mkdir -p /home/vagrant/.ssh
    echo "#{ssh_public_key}" >> /home/vagrant/.ssh/authorized_keys
    chmod 700 /home/vagrant/.ssh
    chmod 600 /home/vagrant/.ssh/authorized_keys
    chown -R vagrant:vagrant /home/vagrant/.ssh

    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
    sed -i 's/#AuthorizedKeysFile/AuthorizedKeysFile/' /etc/ssh/sshd_config
    systemctl restart ssh

    python3 --version
    echo "Chave SSH configurada para os usuários: wiliam, root e vagrant"
  SHELL

  # Provisionamento comum com Ansible
  config.vm.provision "ansible" do |ansible|
    configure_ansible(ansible, "playbooks/common_provisioning.yml", "all")
  end

  # VM: arq
  config.vm.define "arq" do |arq|
    arq.vm.box = "debian/bookworm64"
    arq.vm.hostname = "arq.wiliam.devops"
    arq.vm.network "private_network", ip: "192.168.56.139"

    arq.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      disk_path = File.join(File.dirname(__FILE__), "disks")
      Dir.mkdir(disk_path) unless Dir.exist?(disk_path)
      (1..3).each do |i|
        disk_file = File.join(disk_path, "arq_disk#{i}.vdi")
        unless File.exist?(disk_file)
          vb.customize ["createhd", "--filename", disk_file, "--size", 10 * 1024]
        end
        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", disk_file]
      end
    end

    arq.vm.provision "ansible" do |ansible|
      configure_ansible(ansible, "playbooks/arq_provisioning.yml", "arq")
    end
  end

  # VM: db
  config.vm.define "db" do |db|
    db.vm.box = "debian/bookworm64"
    db.vm.hostname = "db.wiliam.devops"
    db.vm.network "private_network", type: "dhcp"

    db.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end

    db.vm.provision "ansible" do |ansible|
      configure_ansible(ansible, "playbooks/db_provisioning.yml", "db")
    end
  end

  # VM: app
  config.vm.define "app" do |app|
    app.vm.box = "debian/bookworm64"
    app.vm.hostname = "app.wiliam.devops"
    app.vm.network "private_network", type: "dhcp"

    app.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
    end

    app.vm.provision "ansible" do |ansible|
      configure_ansible(ansible, "playbooks/app_provisioning.yml", "app")
    end
  end

  # VM: cli
  config.vm.define "cli" do |cli|
    cli.vm.box = "debian/bookworm64"
    cli.vm.hostname = "cli.wiliam.devops"
    cli.vm.network "private_network", type: "dhcp"

    cli.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
    end

    cli.vm.provision "ansible" do |ansible|
      configure_ansible(ansible, "playbooks/cli_provisioning.yml", "cli")
    end
  end
end


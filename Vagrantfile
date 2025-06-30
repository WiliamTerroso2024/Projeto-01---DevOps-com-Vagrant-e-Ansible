# -*- mode: ruby -*-
# vi: set ft=ruby :

# Função para gerar chave SSH (sempre cria uma nova)
def ensure_ssh_key
  ssh_key_path = File.expand_path("~/.ssh/id_rsa")
  ssh_pub_key_path = "#{ssh_key_path}.pub"
  
  # Remove chaves existentes se existirem
  if File.exist?(ssh_key_path)
    File.delete(ssh_key_path)
    puts "Chave SSH privada antiga removida"
  end
  
  if File.exist?(ssh_pub_key_path)
    File.delete(ssh_pub_key_path)
    puts "Chave SSH pública antiga removida"
  end
  
  # Gera nova chave SSH
  puts "Gerando nova chave SSH..."
  system("ssh-keygen -t rsa -b 4096 -f #{ssh_key_path} -N '' -C 'vagrant@wiliam.devops'")
  puts "Nova chave SSH gerada em #{ssh_key_path}"
  
  # Retorna o conteúdo da chave pública
  File.read(ssh_pub_key_path).strip
end

# Gerar/verificar chave SSH antes de configurar as VMs
ssh_public_key = ensure_ssh_key

Vagrant.configure("2") do |config|
  # Configurações comuns para todas as VMs
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true # Utilização de clones (linked_clone)
  end

  # Provisioning comum para configurar chaves SSH em todas as máquinas
  config.vm.provision "shell", inline: <<-SHELL
    # Aguardar sistema estar completamente pronto
    sleep 10
    
    # Atualizar sistema primeiro
    apt-get update -y
    apt-get install -y python3 python3-pip openssh-server
    
    # Criar usuário wiliam se não existir
    if ! id "wiliam" &>/dev/null; then
      useradd -m -s /bin/bash wiliam
      echo "wiliam:wiliam" | chpasswd
      usermod -aG sudo wiliam
    fi
    
    # Configurar chave SSH para o usuário wiliam
    mkdir -p /home/wiliam/.ssh
    echo "#{ssh_public_key}" > /home/wiliam/.ssh/authorized_keys
    chmod 700 /home/wiliam/.ssh
    chmod 600 /home/wiliam/.ssh/authorized_keys
    chown -R wiliam:wiliam /home/wiliam/.ssh
    
    # Configurar chave SSH para o usuário root
    mkdir -p /root/.ssh
    echo "#{ssh_public_key}" > /root/.ssh/authorized_keys
    chmod 700 /root/.ssh
    chmod 600 /root/.ssh/authorized_keys
    
    # Configurar chave SSH para o usuário vagrant
    mkdir -p /home/vagrant/.ssh
    echo "#{ssh_public_key}" >> /home/vagrant/.ssh/authorized_keys
    chmod 700 /home/vagrant/.ssh
    chmod 600 /home/vagrant/.ssh/authorized_keys
    chown -R vagrant:vagrant /home/vagrant/.ssh
    
    # Configurar SSH para permitir autenticação por chave
    sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
    sed -i 's/#AuthorizedKeysFile/AuthorizedKeysFile/' /etc/ssh/sshd_config
    systemctl restart ssh
    
    # Garantir que Python3 está disponível
    python3 --version
    
    echo "Chave SSH configurada para os usuários: wiliam, root e vagrant"
  SHELL

  # Playbook comum para todas as máquinas
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbooks/common_provisioning.yml"
    ansible.become = true
    ansible.limit = "all" # Aplica a todas as máquinas definidas
  end

  # Servidor de arquivos (arq)
  config.vm.define "arq" do |arq|
    arq.vm.box = "debian/bookworm64"
    arq.vm.hostname = "arq.wiliam.devops"
    arq.vm.network "private_network", ip: "192.168.56.139"
    arq.vm.provider "virtualbox" do |vb|
      vb.memory = "512" # Memória RAM: 512 MB
      # Discos adicionais: 3 discos de 10 GB cada
      disk_path = File.join(File.dirname(__FILE__), "disks")
      Dir.mkdir(disk_path) unless Dir.exist?(disk_path)
      (1..3).each do |i|
        disk_file = File.join(disk_path, "arq_disk#{i}.vdi")
        # Só cria o disco se não existir
        unless File.exist?(disk_file)
          vb.customize ["createhd", "--filename", disk_file, "--size", 10 * 1024]
        end
        
        # Anexa o disco à VM
        vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", i, "--device", 0, "--type", "hdd", "--medium", disk_file]
      end
    end
    arq.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/arq_provisioning.yml"
      ansible.become = true
      ansible.limit = "arq"
    end
  end

  # Servidor de banco de dados (db)
  config.vm.define "db" do |db|
    db.vm.box = "debian/bookworm64"
    db.vm.hostname = "db.wiliam.devops"
    db.vm.network "private_network", type: "dhcp"
    db.vm.provider "virtualbox" do |vb|
      vb.memory = "512" # Memória RAM: 512 MB
    end
    db.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/db_provisioning.yml"
      ansible.become = true
      ansible.limit = "db"
    end
  end

  # Servidor de aplicação (app)
  config.vm.define "app" do |app|
    app.vm.box = "debian/bookworm64"
    app.vm.hostname = "app.wiliam.devops"
    app.vm.network "private_network", type: "dhcp"
    app.vm.provider "virtualbox" do |vb|
      vb.memory = "512" # Memória RAM: 512 MB
    end
    app.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/app_provisioning.yml"
      ansible.become = true
      ansible.limit = "app"
    end
  end

  # Host Cliente (cli)
  config.vm.define "cli" do |cli|
    cli.vm.box = "debian/bookworm64"
    cli.vm.hostname = "cli.wiliam.devops"
    cli.vm.network "private_network", type: "dhcp"
    cli.vm.provider "virtualbox" do |vb|
      vb.memory = "1024" # Memória RAM: 1024 MB
    end
    cli.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbooks/cli_provisioning.yml"
      ansible.become = true
      ansible.limit = "cli"
    end
  end
end

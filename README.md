# Projeto 01 - DevOps com Vagrant e Ansible

## Informações da Disciplina

- **Instituição:** Instituto Federal da Paraíba - Campus João Pessoa
- **Disciplina:** Administração de Sistemas Abertos
- **Professor:** Leonidas Francisco de Lima Júnior
- **Período:** 2025.1

## Equipe

- **Aluno 1:** Wiliam Terroso de Sousa Melo
- **Matrícula:** 20212380039

## Descrição do Projeto

Este projeto tem como objetivo desenvolver competências práticas em DevOps e Infraestrutura como Código (IaC) utilizando as ferramentas Vagrant e Ansible. O projeto provisiona uma infraestrutura virtual composta por quatro máquinas virtuais e automatiza a configuração do sistema operacional e serviços essenciais.

## Arquitetura da Infraestrutura

### Topologia de Rede

A infraestrutura é composta por quatro máquinas virtuais conectadas em uma rede privada (192.168.56.0/24):

```
┌─────────────────────────────────────────────────────────────┐
│                    Rede Privada 192.168.56.0/24            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ Servidor    │  │ Servidor    │  │ Servidor    │         │
│  │ Arquivo     │  │ Banco       │  │ Aplicação   │         │
│  │ (arq)       │  │ (db)        │  │ (app)       │         │
│  │ .239        │  │ DHCP        │  │ DHCP        │         │
│  │ DHCP/NFS/   │  │ MariaDB     │  │ Apache2     │         │
│  │ LVM         │  │ AutoFS      │  │ AutoFS      │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                             │
│                    ┌─────────────┐                         │
│                    │ Cliente     │                         │
│                    │ (cli)       │                         │
│                    │ DHCP        │                         │
│                    │ Firefox/X11 │                         │
│                    │ AutoFS      │                         │
│                    └─────────────┘                         │
└─────────────────────────────────────────────────────────────┘
```

### Especificações das Máquinas Virtuais

#### Servidor de Arquivos (arq)
- **Hostname:** arq.wiliam.[nome2].devops
- **IP:** 192.168.56.139 (fixo)
- **RAM:** 512 MB
- **Discos Adicionais:** 3 discos de 10 GB cada
- **Serviços:** DHCP Server, NFS Server, LVM

#### Servidor de Banco de Dados (db)
- **Hostname:** db.wiliam.[nome2].devops
- **IP:** 192.168.56.1[YY] (via DHCP - reserva manual)
- **RAM:** 512 MB
- **Serviços:** MariaDB Server, AutoFS

#### Servidor de Aplicação (app)
- **Hostname:** app.wiliam.[nome2].devops
- **IP:** 192.168.56.1[XY] (via DHCP - reserva manual)
- **RAM:** 512 MB
- **Serviços:** Apache2, AutoFS

#### Cliente (cli)
- **Hostname:** cli.wiliam.[nome2].devops
- **IP:** DHCP (faixa 50-100)
- **RAM:** 1024 MB
- **Serviços:** Firefox ESR, X11 Forwarding, AutoFS

## Tecnologias Utilizadas

- **Vagrant:** Provisionamento e gerenciamento das máquinas virtuais
- **Ansible:** Automação da configuração e deploy
- **VirtualBox:** Provider de virtualização
- **Debian 12 (Bookworm):** Sistema operacional base
- **DHCP:** Atribuição dinâmica de endereços IP
- **NFS:** Compartilhamento de arquivos em rede
- **LVM:** Gerenciamento lógico de volumes
- **Apache2:** Servidor web
- **MariaDB:** Sistema de gerenciamento de banco de dados
- **AutoFS:** Montagem automática de sistemas de arquivos

## Estrutura do Projeto

```
projeto-devops-vagrant-ansible/
├── README.md
├── Vagrantfile
├── ansible/
│   ├── playbooks/
│   │   ├── main.yml
│   │   ├── common.yml
│   │   ├── arq.yml
│   │   ├── db.yml
│   │   ├── app.yml
│   │   └── cli.yml
│   ├── inventory/
│   │   └── hosts.yml
│   ├── group_vars/
│   │   └── all.yml
│   └── files/
│       ├── index.html
│       ├── dhcpd.conf
│       ├── exports
│       └── ssh_banner
└── docs/
    └── configuracao-detalhada.md
```

## Configurações Implementadas

### Configurações Comuns (Todas as Máquinas)

- ✅ Atualização completa do sistema operacional
- ✅ Configuração do servidor NTP (chrony) com pool.ntp.br
- ✅ Ajuste de timezone para America/Recife
- ✅ Criação do grupo `ifpb`
- ✅ Criação dos usuários `wiliam` e `[nome2]`
- ✅ Configuração SSH com autenticação por chaves públicas
- ✅ Bloqueio de acesso SSH para usuário root
- ✅ Restrição de acesso SSH aos grupos `vagrant` e `ifpb`
- ✅ Banner de segurança no SSH
- ✅ Configuração de sudo para o grupo `ifpb`
- ✅ Instalação do cliente NFS

### Servidor de Arquivos (arq)

- ✅ **DHCP Server:**
  - Domínio: wiliam.[nome2].devops
  - DNS: 1.1.1.1, 8.8.8.8
  - Faixa DHCP: 192.168.56.50-100
  - Reservas manuais para db e app

- ✅ **LVM:**
  - Volume Group: `dados` (3 discos de 10GB)
  - Logical Volume: `ifpb` (15GB)
  - Sistema de arquivos: ext4
  - Ponto de montagem: `/dados`

- ✅ **NFS Server:**
  - Compartilhamento: `/dados/nfs`
  - Acesso: rede 192.168.56.0/24
  - Usuário: `nfs-ifpb`
  - Mapeamento automático de usuários

### Servidor de Banco de Dados (db)

- ✅ Instalação do MariaDB Server
- ✅ Configuração do AutoFS para montagem NFS
- ✅ Montagem automática de `/dados/nfs` em `/var/nfs`

### Servidor de Aplicação (app)

- ✅ Instalação e configuração do Apache2
- ✅ Página customizada com informações do projeto
- ✅ Configuração do AutoFS para montagem NFS
- ✅ Montagem automática de `/dados/nfs` em `/var/nfs`

### Cliente (cli)

- ✅ Instalação do Firefox ESR e xauth
- ✅ Configuração SSH para X11 Forwarding
- ✅ Configuração do AutoFS para montagem NFS
- ✅ Montagem automática de `/dados/nfs` em `/var/nfs`

## Pré-requisitos

Antes de executar o projeto, certifique-se de ter instalado:

- [Vagrant](https://www.vagrantup.com/) (versão 2.2+)
- [VirtualBox](https://www.virtualbox.org/) (versão 6.0+)
- [Ansible](https://www.ansible.com/) (versão 2.9+)

### Verificação dos Pré-requisitos

```bash
# Verificar versão do Vagrant
vagrant --version

# Verificar versão do VirtualBox
vboxmanage --version

# Verificar versão do Ansible
ansible --version
```

## Como Executar o Projeto

### 1. Clonar o Repositório

```bash
git clone https://github.com/[usuario]/projeto-devops-vagrant-ansible.git
cd projeto-devops-vagrant-ansible
```

### 2. Inicializar a Infraestrutura

```bash
# Criar e provisionar todas as VMs
vagrant up

# Ou criar VMs individualmente
vagrant up arq
vagrant up db
vagrant up app
vagrant up cli
```

### 3. Executar o Provisionamento Ansible

```bash
# Executar playbook principal
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/main.yml

# Executar playbook específico
ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/arq.yml
```

### 4. Verificar o Status das Máquinas

```bash
# Verificar status de todas as VMs
vagrant status

# Acessar uma VM específica
vagrant ssh arq
vagrant ssh db
vagrant ssh app
vagrant ssh cli
```

## Comandos Úteis

### Gerenciamento das VMs

```bash
# Parar todas as VMs
vagrant halt

# Reiniciar uma VM específica
vagrant reload arq

# Destruir todas as VMs
vagrant destroy -f

# Re-provisionar uma VM
vagrant provision arq
```

### Teste dos Serviços

```bash
# Testar conectividade
ping 192.168.56.139  # Servidor arq

# Testar DHCP
dhclient -v eth1

# Testar NFS
showmount -e 192.168.56.139

# Testar Apache
curl http://192.168.56.1[XY]

# Testar SSH com X11
ssh -X wiliam@192.168.56.139
```

## Solução de Problemas

### Problemas Comuns

1. **Erro de Box não encontrada:**
   ```bash
   vagrant box add debian/bookworm64
   ```

2. **Conflito de IP:**
   - Verificar se a rede 192.168.56.0/24 não está em uso
   - Ajustar configurações de rede no Vagrantfile

3. **Falha no provisionamento Ansible:**
   ```bash
   # Verificar conectividade
   ansible all -i ansible/inventory/hosts.yml -m ping
   
   # Executar em modo verbose
   ansible-playbook -i ansible/inventory/hosts.yml ansible/playbooks/main.yml -vvv
   ```

4. **Problemas com NFS:**
   ```bash
   # Verificar status do servidor NFS
   vagrant ssh arq
   sudo systemctl status nfs-kernel-server
   
   # Verificar exports
   sudo exportfs -v
   ```

## Testes e Validação

### Checklist de Validação

- [ ] Todas as VMs foram criadas com sucesso
- [ ] Configuração de rede está funcional
- [ ] DHCP está atribuindo IPs corretamente
- [ ] NFS está compartilhando `/dados/nfs`
- [ ] AutoFS está montando automaticamente
- [ ] Apache está servindo a página customizada
- [ ] MariaDB está instalado e funcionando
- [ ] SSH está configurado corretamente
- [ ] X11 Forwarding está funcionando
- [ ] Usuários e grupos foram criados
- [ ] Sudo está configurado para o grupo `ifpb`

### Scripts de Teste

```bash
# Executar testes automatizados
./scripts/test-infrastructure.sh

# Testar cada serviço individualmente
./scripts/test-dhcp.sh
./scripts/test-nfs.sh
./scripts/test-apache.sh
./scripts/test-ssh.sh
```

## Monitoramento e Logs

### Localização dos Logs

- **Vagrant:** `~/.vagrant.d/logs/`
- **Ansible:** Output no terminal durante execução
- **Sistema:** `/var/log/` em cada VM
- **Serviços específicos:**
  - DHCP: `/var/log/dhcp/`
  - Apache: `/var/log/apache2/`
  - NFS: `/var/log/syslog`

### Comandos de Monitoramento

```bash
# Status dos serviços em cada VM
sudo systemctl status dhcpd        # arq
sudo systemctl status nfs-server   # arq
sudo systemctl status mariadb      # db
sudo systemctl status apache2      # app
sudo systemctl status autofs       # db, app, cli
```

## Documentação Adicional

- [Configuração Detalhada](docs/configuracao-detalhada.md)
- [Troubleshooting Avançado](docs/troubleshooting.md)
- [Customizações](docs/customizacoes.md)

## Contribuição

Para contribuir com o projeto:

1. Fork do repositório
2. Criar branch para feature (`git checkout -b feature/nova-feature`)
3. Commit das mudanças (`git commit -am 'Adiciona nova feature'`)
4. Push para branch (`git push origin feature/nova-feature`)
5. Criar Pull Request

## Licença

Este projeto é desenvolvido para fins acadêmicos no âmbito da disciplina de Administração de Sistemas Abertos do IFPB.

## Contato

- **Wiliam Terroso de Sousa Melo** - [wiliam.melo@academico.ifpb.edu.br]

---

**Instituto Federal da Paraíba - Campus João Pessoa**  
**Disciplina:** Administração de Sistemas Abertos  
**Professor:** Leonidas Francisco de Lima Júnior  
**Período:** 2025.1


# OS/Box
VAGRANT_BOX = 'bento/centos-7.7'
# VM User — 'vagrant' by default
VM_USER = 'vagrant'


# Master config params
VM_NAME_ANSIBLE = 'ansible'
ANSIBLE_ADDRESS = '192.168.1.100' ########## PLEASE CHANGE ME AS NEEDED
ANSIBLE_HOSTNAME = "ansible"

# Worker config params
VM_NAME_VAULT ='vault'
VAULT_ADDRESS = '192.168.1.101' ########## PLEASE CHANGE ME AS NEEDED
VAULT_HOSTNAME = "vault"


Vagrant.configure(2) do |config|

  # Configuration definitions for the "Master" kube VM
  config.vm.define "master" do |master|

    # Configure box type
    master.vm.box = VAGRANT_BOX
    master.vm.hostname = ANSIBLE_HOSTNAME

    # Configure the Network
    master.vm.network "private_network", ip: ANSIBLE_ADDRESS

    master.vm.provider "virtualbox" do |v|
      v.name = VM_NAME_ANSIBLE
      v.memory = 2048
    end

    # Script provisioner for Master
   $masterFirstBoot = <<-SCRIPT
  ##  systemctl restart networking
  sudo yum install epel-release
  sudo yum install ansible
  SCRIPT

    #master.vm.provision "shell", inline: $masterFirstBoot
  end


  # Configuration definitions for the "Worker" kube VM
  config.vm.define "vault" do |worker|
    # Configure box type
    worker.vm.box = VAGRANT_BOX
    worker.vm.hostname = VAULT_HOSTNAME

    # Configure the Network
    worker.vm.network "private_network", ip: VAULT_ADDRESS

    worker.vm.provider "virtualbox" do |v|
      v.name = VM_NAME_VAULT
      v.memory = 1024
      worker.vm.synced_folder ".", "/vagrant", disabled: true
    end

    # Script provisioner for worker
    $scriptworker = <<-SCRIPT
    ping -c 10 ${masteraddress}
    sudo yum install unzip
    export VAULT_URL="https://releases.hashicorp.com/vault" \
         VAULT_VERSION="1.5.5"
    curl \
      --silent \
      --remote-name \
     "${VAULT_URL}/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip"

    curl \
      --silent \
      --remote-name \
      "${VAULT_URL}/${VAULT_VERSION}/vault_${VAULT_VERSION}_SHA256SUMS"

    curl \
      --silent \
      --remote-name \
      "${VAULT_URL}/${VAULT_VERSION}/${VAULT_VERSION}/vault_${VAULT_VERSION}_SHA256SUMS.sig"

    unzip vault_${VAULT_VERSION}_linux_amd64.zip
    sudo chown root:root vault
    sudo mv vault /usr/local/bin/
    vault -autocomplete-install
    complete -C /usr/local/bin/vault vault

    SCRIPT


    worker.vm.provision "shell", inline: $scriptworker, env: {"masteraddress" => VAULT_ADDRESS},
      run: "always"
  end






end

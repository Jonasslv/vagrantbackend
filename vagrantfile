#echo dos recursos da VM, só funciona em linux
cpus = `nproc`.to_i
memory = `grep 'MemTotal' /proc/meminfo | sed -e 's/MemTotal://' -e 's/ kB//'`.to_i / 1024 / 4

memory = [memory, 4096].max
puts "Platform: " + cpus.to_s + " CPUs, " + memory.to_s + " MB memory" 


Vagrant.configure("2") do |config|
  #fedora32, atualizável
  config.vm.box = "generic/fedora32"
  #nome qualquer 
  config.vm.hostname = "dev.local" 
  #atenção verificar com ip após com ip addr show
  config.vm.network "public_network", type:"dhcp" 
  
  #pasta dos scripts a serem modificados
  config.vm.synced_folder "./data", "/vagrant_data" 

  
  config.vm.provider "virtualbox" do |vb|
    #recursos a serem utilizados
    vb.memory = memory 
    vb.cpus = [cpus, 32].min
  end

  #roda como root
  config.vm.provision "shell", privileged: "true", inline: <<-SHELL
    #atualiza os pacotes do sistema
    dnf update -y 
    #instala o kit de desenvolvimento banco de dados e net core
    dnf install -y postgresql-server postgresql-contrib dotnet-sdk-3.1 nano
    #inicia o banco de configuração do postgres 
    postgresql-setup --initdb --unit postgresql 
    #ativa na inicialização o serviço do postgres
    systemctl enable postgresql.service 
    #inicia agora o serviço do postgres
    systemctl start postgresql.service 
    #adiciona o repositorio atualizado do pgadmin
    rpm -i https://ftp.postgresql.org/pub/pgadmin/pgadmin4/yum/pgadmin4-fedora-repo-1-1.noarch.rpm 
    #instala o pgadmin versao headless
    yum -y install pgadmin4-web 
    #desativa o firewall
    systemctl disable firewalld & systemctl stop firewalld 
    #copia os arquivos de configuração
    cp -f /vagrant_data/pg_hba.conf /var/lib/pgsql/data/ 
    cp -f /vagrant_data/postgresql.conf /var/lib/pgsql/data/ 
    #senha pgadmin
    export PGADMIN_SETUP_PASSWORD=123456 
    #email pgadmin
    export PGADMIN_SETUP_EMAIL=dev@dev.com 
    #configura o pgadmin
    /usr/pgadmin4/bin/setup-web.sh --yes 
    #recarrega os processos e ativa o httpd
    systemctl enable httpd & systemctl start httpd & systemctl reload postgresql 
    #altera a senha de usuario do postgres
    sudo -u postgres psql -U postgres -d postgres -c "alter user postgres with password 'postgres'" 
    #mostra o ip da maquina
    ip addr show 
    #talvez configurar httpd para allow conexões externas
    #só usar o ip da maquina/pgadmin4 para conectar, se quiser compilar código usar dotnet da maquina com vagrant ssh
  SHELL

end


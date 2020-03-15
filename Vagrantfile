Vagrant.configure(2) do |config|
######################################################################################################
#---------------------------------------------- VM Web ----------------------------------------------#
######################################################################################################
  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/xenial64"

    #hier werden div. VM Konfigurationen eingestellt
    web.vm.provider "virtualbox" do |v|
      v.memory = "512"
      v.name = "LB2_web"
      v.cpus = 1
    end

    #Hier wird die Port-Weiterleitung eingerichtet
    web.vm.network "forwarded_port", guest:80, host:1657, auto_correct:true

    #hier wird angegeben, dass der Folder "." auf der lokalen Maschine = /var/www/html auf der VM ist
    web.vm.synced_folder ".", "/var/www/html"

    #Hier wird angegeben, dass in der Shell Apache2 installiert werden soll
    config.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get -y install apache2
    SHELL
  end

######################################################################################################
#--------------------------------------------- VM Share ---------------------------------------------#
######################################################################################################
  config.vm.define "share" do |sh|
    sh.vm.box = "ubuntu/xenial64"

    #hier werden div. VM Konfigurationen eingestellt
    sh.vm.provider "virtualbox" do |v|
      v.memory = "1024"
      v.name = "LB2_share"
      v.cpus = 1
    end

    #Hier wird die Port-Weiterleitung eingerichtet
    sh.vm.network "forwarded_port", guest:80, host:1680, auto_correct:true

    #hier wird angegeben, dass der Folder "./share/" auf der lokalen Maschine = /var/www/html auf der VM ist
    sh.vm.synced_folder "./share/", "/var/www/html"

    #Hier wird angegeben, dass in der Shell Apache2 installiert werden soll. Ausserdem wird die index.html Seite, welche Standardmässig von Apache erstellt wird, gelöscht
    sh.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get -y install apache2
      
      #Falls die Datei 'index.html' existiert, soll sie gelöscht werden
      if [ -e /var/www/html/index.html ]; then
        rm /var/www/html/index.html
      fi
    SHELL
  end
end

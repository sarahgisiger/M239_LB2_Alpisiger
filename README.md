# Dokumentation LB2
## Modul300
**Modul 300:** Plattformübergreifende Dienste in ein Netzwerk integrieren
**Autor:** Sarah Gisiger
**Datum:** 15.03.20

## Allgemeines
Mein Vagrantfile habe ich so geschrieben, dass es drei VM's erstellt. Für alle VM's habe ich jeweils 'ubuntu/xenial64' als Box verwendet. Nachfolgend werden die drei VM's beschrieben:

### LB2_web
#### Fakten
>- **Name:** LB2_web
>- **Installierte Dienste:** Apache
>- **Portweitetleitung:** Port 1657
>- **Synchronisierter Folder:** "." 
>- **[Link zur Index-Seite](http://localhost:1657/)**

#### Weiteres
> Die Index Seite habe ich selber erstellt. Ich habe nur etwas einfaches gemacht, da es nicht weiter relevant für die LB ist. Damit das GIF auf der Seite angezeigt wird, muss die Datei 'theoffice.gif' zwingend vorhanden sein. 
> 
> Desweiteren ist der Link zum Share und zur Datenbank ebenfalls auf der Index Seite verlinkt.

#### Vorgehen
> Um meine eigene Index Seite verwenden zu können, habe ich im Vagrantfile den Pfad unter synced_folder folgendermassen abgeändert: **web.vm.synced_folder ".", "/var/www/html"**

#### Bild
![ ](./indexseite.png "Index Seite")

#### Code
##### HTML
```css
<!doctype html>
<html>
  <head>
    <title>Meine Index Seite</title>
  </head>
  <body>
    <p>Hello :)</p>
    <IMG SRC="theoffice.gif">
    <br><br>
    <h1>Link</h1>
    <a href="http://localhost:1680/">Link zum Share</a>
    <br>
    <a href="http://localhost:1670/adminer.php">Link zur DB</a>
  </body>
</html>
```
##### Vagrant - nur für LB2_web
```css
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
```


### LB2_share
#### Fakten
>- **Name:** LB2_web
>- **Installierte Dienste:** Apache
>- **Portweitetleitung:** Port 1680
>- **Synchronisierter Folder:** "./share" 
>- **[Link zum Share](http://localhost:1680/)**

#### Weiteres
> Damit der Share funktioniert, wird der Folder './share' benötigt.
> 
> In diesem Share werden alle Datein vom Ordner angezeigt. Wenn eine Datei geändert wird, wird dies ebenfalls auf der Seite gezeigt. 


#### Vorgehen
> Damit der Share funktioniert muss zuerst ein Ornder erstellt werden, welcher anschliessend geshared wird. Dafür habe ich ein Ordner namens 'share' erstellt. Anschliessend muss der Pfad im Vagrantfile unter synced_folder folgendermassen abgeändert werden: **web.vm.synced_folder "./share", "/var/www/html"**
> 
> Wichtig hier ist, dass im Vagrantfile in der Shell angegeben wird, dass die Index Seite, welche Apache standardmässig erstellt, gelöscht wird. Falls dies nicht gemacht wird, sieht man nicht den Share, sondern die Standard-Index Seite von Apache.

#### Bild
![ ](./share.png "Share")

#### Code
##### Vagrant - nur für LB2_share
```css
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
```

### LB2_db
#### Fakten
>- **Name:** LB2_db
>- **Installierte Dienste:** Apache, PHP, debconf-utils
>- **Portweitetleitung:** Port 1670
>- ~~**Synchronisierter Folder:**~~ 
>- **Anmeldedaten:** Benutzer = root, Passwort = admin
>- **[Link zum Share](http://localhost:1670/adminer.php)**

#### Weiteres
> Als Vorlage für diese VM habe ich das Vagrantfile von [hier](https://github.com/mc-b/M300/tree/master/vagrant/db) verwendet. 


#### Vorgehen
> Ich habe die Vorlage verwendet. Zuerst habe ich mir das File angeschaut und Zeilen, die ich nicht verstanden habe, nachgeschaut. Anschliessend habe ich selber ein paar Kleinigkeiten abgeändert und noch etwas dazu Kommentiert.

#### Bild
![ ](./db.png "Datenbank")

#### Code
##### Vagrant - nur für LB2_share
```css
config.vm.define "db" do |db|
  db.vm.box = "ubuntu/xenial64"

  #hier werden div. VM Konfigurationen eingestellt
  db.vm.provider "virtualbox" do |v|
    v.memory = "512"
    v.name = "LB2_db"
    v.cpus = 1
  end

  #Hier wird die Port-Weiterleitung eingerichtet
  db.vm.network "forwarded_port", guest:80, host:1670, auto_correct:true

  config.vm.provision "shell", inline: <<-SHELL
    set -o xtrace
    #debconf-utils = Debian configuration management system
    #installation von debconf-utils, mysql Passwort setzung & installation von apache und php  
    sudo apt-get update
    sudo apt-get -y install debconf-utils
    sudo apt-get -y install apache2
    sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password admin'
    sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password admin'
    sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client mysql-server
     
    #Herunterladen von 'latest.php' in das erstellte Verzeichnis & schliesslich Restart von Apache 
    sudo mkdir /usr/share/adminer
    sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
    sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
    echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
    sudo a2enconf adminer.conf
    sudo service apache2 restart
  SHELL
end
```

## Vagrantfile Code
``` css
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

######################################################################################################
#----------------------------------------------- VM DB ----------------------------------------------#
######################################################################################################
  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/xenial64"

    #hier werden div. VM Konfigurationen eingestellt
    db.vm.provider "virtualbox" do |v|
      v.memory = "512"
      v.name = "LB2_db"
      v.cpus = 1
    end

    #Hier wird die Port-Weiterleitung eingerichtet
    db.vm.network "forwarded_port", guest:80, host:1670, auto_correct:true

    config.vm.provision "shell", inline: <<-SHELL
      set -o xtrace
      #debconf-utils = Debian configuration management system
      #installation von debconf-utils, mysql Passwort setzung & installation von apache und php  
      sudo apt-get update
      sudo apt-get -y install debconf-utils
      sudo apt-get -y install apache2
      sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password password admin'
      sudo debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password admin'
      sudo apt-get -y install php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client mysql-server
       
      #Herunterladen von 'latest.php' in das erstellte Verzeichnis & schliesslich Restart von Apache 
      sudo mkdir /usr/share/adminer
      sudo wget "http://www.adminer.org/latest.php" -O /usr/share/adminer/latest.php
      sudo ln -s /usr/share/adminer/latest.php /usr/share/adminer/adminer.php
      echo "Alias /adminer.php /usr/share/adminer/adminer.php" | sudo tee /etc/apache2/conf-available/adminer.conf
      sudo a2enconf adminer.conf
      sudo service apache2 restart
    SHELL
  end
end
```


## Links

 [Link zur Index-Seite](http://localhost:1657/) 
 
 [Link zum Share](http://localhost:1680/)

[Link zur DB](http://localhost:1670/adminer.php)

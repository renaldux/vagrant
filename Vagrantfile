# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = "scotch/box"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.hostname = "scotchbox"
  config.vm.synced_folder ".", "/var/www", :nfs => { :mount_options => ["dmode=777","fmode=666"] }
  config.ssh.shell = "bash -c 'BASH_ENV=/etc/profile exec bash'"

  ## create public dir
  config.vm.provision "shell", inline: "mkdir -p /var/www/public"

  ## RUN MAILCATCHER ON EVERY START
  ## REMOVE AFTER FIRST LAUNCH !!!!
  config.vm.provision "shell", inline: "/home/vagrant/.rbenv/shims/mailcatcher --http-ip=0.0.0.0", run: "always"
#    check emails
#    http://192.168.33.10:1080


  ## IMPORT DATABASES
  config.vm.provision "shell", inline: <<-SHELL
#   DATABASES=("dbone" "dbtwo" "dbthree")
      for ((i=0; i < ${#DATABASES[@]}; i++)); do

          ## Current Database
          DATABASE=${DATABASES[$i]}

          echo "Creating Database for $DATABASE..."
          mysql -u root -proot -e "DROP DATABASE IF EXISTS $DATABASE";
          mysql -u root -proot -e "CREATE DATABASE $DATABASE";
          echo "Importing data to database $DATABASE"
          mysql -u root -proot $DATABASE < /var/www/sql/$DATABASE.sql

      done
  SHELL

  ## install xdebug
  config.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install php5-xdebug
      sudo sh -c "echo 'zend_extension=xdebug.so
xdebug.remote_enable=1
xdebug.remote_connect_back=1
xdebug.remote_port=9000
xdebug.remote_host=192.168.33.10' > /etc/php5/apache2/conf.d/20-xdebug.ini"
      sudo service apache2 restart

  SHELL

  ## add composer to path
  config.vm.provision "shell", inline: <<-SHELL
      export PATH="~/.composer/vendor/bin:$PATH"
  SHELL

  ## configure domains (for first run only)
  config.vm.provision "shell", inline: <<-SHELL

      ## !!!!!!!!!!!!!!ENTER DOMAIN NAMES HERE!!!!!!!!!!!!!!
      DOMAINS=("symfony.local" "test.local")

      ## SSL konfiguracija
      sudo a2enmod ssl
      sudo a2enmod socache_dbm
      sudo a2enmod socache_memcache
      sudo a2enmod socache_shmcb

      for ((i=0; i < ${#DOMAINS[@]}; i++)); do

          ## Current Domain
          DOMAIN=${DOMAINS[$i]}

          echo "Creating directory for $DOMAIN..."
          mkdir -p /var/www/$DOMAIN/web

          echo "Creating vhost config for $DOMAIN..."
          sudo rm -fR /etc/apache2/sites-available/$DOMAIN.conf /etc/apache2/sites-available/m.$DOMAIN.conf /etc/apache2/sites-available/$DOMAIN-ssl.conf /etc/apache2/sites-available/m.$DOMAIN-ssl.conf
          sudo cp /etc/apache2/sites-available/scotchbox.local.conf /etc/apache2/sites-available/$DOMAIN.conf
          sudo cp /etc/apache2/sites-available/scotchbox.local.conf /etc/apache2/sites-available/m.$DOMAIN.conf
          sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/$DOMAIN-ssl.conf
          sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/m.$DOMAIN-ssl.conf

          oldpb='/var/www/public'
          oldht='/var/www/html'
          newpath="/var/www/$DOMAIN/web"

          ## UPDATE WEBSITES CONFIG FILES
          sudo sed -i -e "s|scotchbox.local|$DOMAIN|g" /etc/apache2/sites-available/$DOMAIN.conf
          sudo sed -i -e "s|scotchbox.local|m.$DOMAIN|g" /etc/apache2/sites-available/m.$DOMAIN.conf
          sudo sed -i -e "s|$oldpb|$newpath|g" /etc/apache2/sites-available/m.$DOMAIN.conf
          sudo sed -i -e "s|$oldpb|$newpath|g" /etc/apache2/sites-available/$DOMAIN.conf

          ## UPDATE SSL WEBSITES CONFIG FILES
          sudo sed -i s,"ServerAdmin webmaster@localhost","ServerName $DOMAIN",g /etc/apache2/sites-available/$DOMAIN-ssl.conf
          sudo sed -i s,"ServerAdmin webmaster@localhost","ServerName m.$DOMAIN",g /etc/apache2/sites-available/m.$DOMAIN-ssl.conf
          sudo sed -i s,$oldht,$newpath,g /etc/apache2/sites-available/m.$DOMAIN-ssl.conf
          sudo sed -i s,$oldht,$newpath,g /etc/apache2/sites-available/$DOMAIN-ssl.conf

       sudo a2ensite $DOMAIN.conf
       sudo a2ensite $DOMAIN-ssl.conf
       sudo a2ensite m.$DOMAIN-ssl.conf
       sudo a2ensite m.$DOMAIN.conf

      done

      sudo apache2ctl graceful
      sudo apache2ctl -S
      sudo service apache2 restart

  SHELL

end
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.network "forwarded_port", guest: 80, host: 8080

  config.vm.provider 'virtualbox' do |vb|
    vb.memory = 4096
    vb.cpus = 4
  end

  config.vm.provision "shell", inline: <<-SHELL
    sudo apt-get update -y
    wget https://wordpress.org/wordpress-6.2.tar.gz
    wget https://ftp.drupal.org/files/projects/drupal-10.0.7.tar.gz
    tar -xzf wordpress-6.2.tar.gz && tar -xzf drupal-10.0.7.tar.gz
    echo "<?php

define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'wordpress' );
define( 'DB_PASSWORD', '1111' );
define( 'DB_HOST', 'localhost' );
define( 'DB_CHARSET', 'utf8' );
define( 'DB_COLLATE', '' );
" > /home/vagrant/wordpress/wp-config.php
    curl https://api.wordpress.org/secret-key/1.1/salt/ >> /home/vagrant/wordpress/wp-config.php
    sed '1,60d' /home/vagrant/wordpress/wp-config-sample.php >> /home/vagrant/wordpress/wp-config.php

    sudo su
    apt-get install -y apache2 mysql-server libapache2-mod-php php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip php-mysql
    systemctl start apache2 ; systemctl enable apache2 ; a2enmod rewrite ; systemctl restart apache2
    mysql -f -e "CREATE DATABASE wordpress;"
    mysql -f -e "CREATE USER wordpress@'localhost' IDENTIFIED BY '1111';"
    mysql -f -e "GRANT ALL ON wordpress.* TO 'wordpress'@'localhost';"
    mysql -f -e "CREATE DATABASE drupal;"
    mysql -f -e "CREATE USER drupal@'localhost' IDENTIFIED BY '1111';"
    mysql -f -e "GRANT ALL ON drupal.* TO 'drupal'@'localhost';"
    mv /home/vagrant/wordpress /var/www/wordpress && mv /home/vagrant/drupal-10.0.7 /var/www/drupal
    chown -R root: /var/www/wordpress/
    cp /var/www/drupal/sites/default/default.settings.php /var/www/drupal/sites/default/settings.php
    chmod -R a+w /var/www/drupal/sites/default
    chown -R www-data: /var/www/drupal/

    echo "<VirtualHost *:80>
  ServerName network1.com
  ServerAdmin admin@wordpress
  DocumentRoot /var/www/wordpress

  <Directory /var/www/wordpress>
    AllowOverride All
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/wordpress-error.log
  CustomLog ${APACHE_LOG_DIR}/wordpress-access.log combined
</VirtualHost>
" > /etc/apache2/sites-available/001-wordpress.conf
    echo "<VirtualHost *:80>
  ServerName network2.com
  ServerAdmin admin@drupal
  DocumentRoot /var/www/drupal

  <Directory /var/www/drupal>
    AllowOverride All
  </Directory>

  ErrorLog ${APACHE_LOG_DIR}/drupal-error.log
  CustomLog ${APACHE_LOG_DIR}/drupal-access.log combined
</VirtualHost>
" > /etc/apache2/sites-available/002-drupal.conf

    a2dissite 000-default.conf
    a2ensite 001-wordpress.conf 002-drupal.conf
    service apache2 restart
  SHELL
end

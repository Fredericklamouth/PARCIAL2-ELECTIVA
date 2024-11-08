Vagrant.configure("2") do |config|
  # Configuración de los servidores Apache
  (1..2).each do |i|
    config.vm.define "apache#{i}" do |apache|
      apache.vm.box = "ubuntu/bionic64"
      apache.vm.hostname = "apache#{i}"
      apache.vm.network "private_network", ip: "192.168.56.1#{i}"
      apache.vm.provider "virtualbox" do |vb|
        vb.memory = "512"
        vb.cpus = 1
      end
      # Compartir directorio local con Apache
      apache.vm.synced_folder "./html/apache#{i}", "/var/www/html"
      # Provisionar Apache
      apache.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y apache2
        sudo systemctl enable apache2
        sudo systemctl start apache2
      SHELL
    end
  end

  # Configuración del servidor Nginx
  config.vm.define "nginx" do |nginx|
    nginx.vm.box = "ubuntu/bionic64"
    nginx.vm.hostname = "nginx"
    nginx.vm.network "private_network", ip: "192.168.56.30"
    nginx.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
    # Provisionar Nginx
    nginx.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y nginx
      # Configurar balanceo de carga
      echo '
      upstream backend {
          server 192.168.56.11;
          server 192.168.56.12;
      }
      server {
          listen 80;
          location / {
              proxy_pass http://backend;
          }
      }
      ' | sudo tee /etc/nginx/sites-available/default
      sudo systemctl restart nginx
      sudo systemctl enable nginx
    SHELL
  end
end
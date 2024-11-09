# Vagrantfile
Vagrant.configure("2") do |config|
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "512"
      vb.cpus = 1
    end
  
    # Configuración del primer servidor Apache
    config.vm.define "apache1" do |apache1|
      apache1.vm.box = "ubuntu/jammy64"
      apache1.vm.network "private_network", ip: "192.168.56.2"
      apache1.vm.hostname = "apache1"
      apache1.vm.synced_folder "./Web1", "/var/www/html"  # Directorio compartido Web1
      apache1.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y apache2
        sudo systemctl enable apache2
      SHELL
    end
  
    # Configuración del segundo servidor Apache
    config.vm.define "apache2" do |apache2|
      apache2.vm.box = "ubuntu/jammy64"
      apache2.vm.network "private_network", ip: "192.168.56.3"
      apache2.vm.hostname = "apache2"
      apache2.vm.synced_folder "./Web2", "/var/www/html"  # Directorio compartido Web2
      apache2.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y apache2
        sudo systemctl enable apache2
      SHELL
    end
  
    # Configuración del servidor Nginx como balanceador de carga
    config.vm.define "loadbalancer" do |lb|
      lb.vm.box = "ubuntu/jammy64"
      lb.vm.network "private_network", ip: "192.168.56.4"
      lb.vm.hostname = "loadbalancer"
      lb.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update
        sudo apt-get install -y nginx
        sudo systemctl enable nginx
  
        # Eliminar configuración predeterminada de Nginx en sites-enabled
        sudo rm -f /etc/nginx/sites-enabled/default
  
        # Crear archivo de configuración para balanceo de carga
        sudo bash -c 'cat > /etc/nginx/sites-available/default' <<EOF
        upstream backend {
            server 192.168.56.2;
            server 192.168.56.3;
        }
  
        server {
            listen 80;
            location / {
                proxy_pass http://backend;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;
            }
        }
        EOF
  
        # Crear enlace simbólico para activar la configuración de balanceo de carga
        sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default
  
        # Reiniciar Nginx para aplicar cambios
        sudo systemctl restart nginx
      SHELL
    end
  end
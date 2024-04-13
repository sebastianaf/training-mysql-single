desired_variables = [
  'DB_USER', 
  'DB_PASSWORD', 
  'DB_NAME', 
  'DB_PORT',
  'TZ', 
  'PMA_PORT'
]

if File.exists?('.env')
  File.foreach('.env') do |line|
    if line =~ /^(.+)=(.+)$/ && desired_variables.include?($1)
      ENV[$1] = $2.strip
    end
  end
end

Vagrant.configure("2") do |config|

  db_user = ENV['DB_USER']
  db_password = ENV['DB_PASSWORD']
  db_name = ENV['DB_NAME']
  db_port = ENV['DB_PORT']
  tz = ENV['TZ']
  pma_port = ENV['PMA_PORT']

  config.vm.box = "ubuntu/jammy64"
  config.vm.hostname = "org-mysql"
  config.vm.network "forwarded_port", guest: db_port, host: db_port
  config.vm.network "forwarded_port", guest: pma_port, host: pma_port

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = "1"
    vb.memory = "1024"
  end

  config.vm.define "org-mysql" do |org_mysql|
    org_mysql.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update
      sudo apt-get install -y virtualbox-guest-utils virtualbox-guest-x11 lsb-release
      sudo apt-get install -y ca-certificates curl gnupg git
      sudo install -m 0755 -d /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      sudo chmod a+r /etc/apt/keyrings/docker.gpg
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt-get update
      sudo apt-get install -y docker-ce docker-ce-cli containerd.io
      sudo apt-get install -y mysql-client
      sudo groupadd docker || true
      sudo usermod -aG docker vagrant
      git clone https://github.com/sebastianaf/training-mysql-single
      cd training-mysql-single || true

      echo "DB_USER=#{db_user}" > .env
      echo "DB_PASSWORD=#{db_password}" >> .env
      echo "DB_NAME=#{db_name}" >> .env
      echo "DB_PORT=#{db_port}" >> .env
      echo "TZ=#{tz}" >> .env
      echo "PMA_PORT=#{pma_port}" >> .env
      
      docker compose down
      docker compose up -d --build || true
      
      echo "-------------------------------------------------------------------------------"
      echo "Happy coding! Access MySQL and phpMyAdmin at http://localhost:#{pma_port} from your browser."
      echo "-------------------------------------------------------------------------------"
    SHELL
  end
end

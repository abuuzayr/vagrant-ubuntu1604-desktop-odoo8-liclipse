# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = 'boxcutter/ubuntu1604-desktop'

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  config.vm.network 'forwarded_port', guest: 8069, host: 1234

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  #config.vm.network 'private_network', ip: '192.168.33.10', nictype: 'virtio'

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
   config.vm.network "public_network", ip: '192.168.1.66'

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  config.vm.provider 'virtualbox' do |vb|
    vb.gui = true
    vb.cpus = 2
    vb.memory = 2048
    vb.customize ['modifyvm', :id, '--paravirtprovider', 'kvm']
    vb.customize ['modifyvm', :id, '--accelerate3d', 'on']
    vb.customize ['modifyvm', :id, '--accelerate2dvideo', 'on']
    vb.customize ['modifyvm', :id, '--vram', 256]
    vb.customize ['modifyvm', :id, '--nic1', 'nat']
    vb.customize ['modifyvm', :id, '--nictype1', 'virtio']
  end

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
set -e
export DEBIAN_FRONTEND=noninteractive

sudo locale-gen
sudo localectl set-locale LANG="en_US.UTF-8"

OE_USER="vagrant"
OE_HOME="/home/$OE_USER"
OE_HOME_EXT="/home/$OE_USER/${OE_USER}-server"

OE_PORT="8069"
OE_VERSION="8.0"

OE_SUPERADMIN="admin"
OE_CONFIG="${OE_USER}-server"

WKHTMLTOX_X64=http://download.gna.org/wkhtmltopdf/0.12/0.12.1/wkhtmltox-0.12.1_linux-trusty-amd64.deb

sudo apt-get update
sudo apt-get upgrade -y

sudo apt-get install postgresql -y
sudo su - postgres -c "createuser -s $OE_USER" 2> /dev/null || true

sudo apt-get install wget subversion git bzr bzrtools python-pip gdebi-core -y

sudo apt-get install python-dateutil python-feedparser python-ldap python-libxslt1 python-lxml python-mako python-openid python-psycopg2 python-pybabel python-pychart python-pydot python-pyparsing python-reportlab python-simplejson python-tz python-vatnumber python-vobject python-webdav python-werkzeug python-xlwt python-yaml python-zsi python-docutils python-psutil python-mock python-unittest2 python-jinja2 python-pypdf python-decorator python-requests python-passlib python-pil -y

sudo pip install gdata psycogreen

# This is for compatibility with Ubuntu 16.04. Will work on 14.04, 15.04 and 16.04
sudo -H pip install suds

sudo apt-get install node-clean-css -y
sudo apt-get install node-less -y
sudo apt-get install python-gevent -y

sudo wget $WKHTMLTOX_X64 -q
sudo gdebi --n `basename $WKHTMLTOX_X64`
sudo ln -s /usr/local/bin/wkhtmltopdf /usr/bin
sudo ln -s /usr/local/bin/wkhtmltoimage /usr/bin

# sudo adduser --system --quiet --shell=/bin/bash --home=$OE_HOME --gecos 'ODOO' --group $OE_USER
sudo adduser $OE_USER sudo

sudo mkdir /var/log/$OE_USER
sudo chown $OE_USER:$OE_USER /var/log/$OE_USER

sudo git clone --depth 1 --branch $OE_VERSION https://github.com/odoo/odoo.git $OE_HOME_EXT/
git config --global core.fileMode false

sudo chown -R $OE_USER:$OE_USER $OE_HOME
find $OE_HOME_EXT/ -type d -print0 | sudo xargs -0 chmod 2775
find $OE_HOME_EXT/ -type f -print0 | sudo xargs -0 chmod 0664

sudo cp $OE_HOME_EXT/debian/openerp-server.conf /etc/${OE_CONFIG}.conf
sudo chown $OE_USER:$OE_USER /etc/${OE_CONFIG}.conf
sudo chmod 640 /etc/${OE_CONFIG}.conf

sudo sed -i s/"db_user = .*"/"db_user = $OE_USER"/g /etc/${OE_CONFIG}.conf
sudo sed -i s/"; admin_passwd.*"/"admin_passwd = $OE_SUPERADMIN"/g /etc/${OE_CONFIG}.conf
sudo su root -c "echo 'logfile = /var/log/$OE_USER/$OE_CONFIG$1.log' >> /etc/${OE_CONFIG}.conf"

sudo su root -c "echo 'addons_path=$OE_HOME_EXT/addons,$OE_HOME_EXT/custom/addons' >> /etc/${OE_CONFIG}.conf"

sudo su $OE_USER -c "mkdir $OE_HOME_EXT/custom"
sudo su $OE_USER -c "mkdir $OE_HOME_EXT/custom/addons"

sudo wget http://update.liclipse.com/standalone/liclipse_3.1.0_linux.gtk.x86_64.tar.gz -q
sudo tar xvzf liclipse_3.1.0_linux.gtk.x86_64.tar.gz
sudo mv liclipse /opt
sudo ln -s /opt/liclipse/LiClipse /usr/bin/liclipse
cat <<EOF >/usr/share/applications/liclipse.desktop
[Desktop Entry]
Version=3.1.0
Name=LiClipse
Comment=IDE for Python/Django developers
Exec=env UBUNTU_MENUPROXY=0 /opt/liclipse/LiClipse
Icon=/opt/liclipse/icon.xpm
Terminal=false
Type=Application
Categories=Utility;Application;Development;IDE
EOF

sudo shutdown -r now

SHELL

end

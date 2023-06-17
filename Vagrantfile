# -*- mode: ruby -*-
# vi: set ft=ruby :

# determine vagrant provider
ENV['VAGRANT_DEFAULT_PROVIDER'] = ENV['MICROBOX_BUILD_VAGRANT_PROVIDER'] || 'virtualbox'

# determine pkgsrc-lite directory
[
  '../../microbox-pkgsrc-lite',
  '../../pkgsrc-lite',
  '../.pkgsrc-lite'
].each do |candidate|
  path = File.expand_path(candidate, __FILE__)
  if ::File.exists? path
    $pkgsrc = path
  end
end

# if pkgsrc-lite does not exist, let's fetch it now
if not $pkgsrc
  $pkgsrc = File.expand_path('../.pkgsrc-lite', __FILE__)
  clone_url = 'https://github.com/mu-box/microbox-pkgsrc-lite.git'

  stdout_sync = STDOUT.sync
  stderr_sync = STDERR.sync
  STDOUT.sync = STDERR.sync = true
  puts "mu-box/microbox-pkgsrc-lite is required to build packages, cloning into .pkgsrc-lite..."

  IO.popen "git clone #{clone_url} #{$pkgsrc}" do |out|
    until out.eof?
      puts f.gets
    end
  end

  STDOUT.sync = stdout_sync
  STDERR.sync = stderr_sync
end

# create required directories for local cache
['.distfiles', '.packages'].each do |cache|
  path = File.expand_path("../#{cache}", __FILE__)
  if not ::File.exists? path
    ::FileUtils.mkdir_p path
  end
end

Vagrant.configure('2') do |config|

  config.vm.box = 'trusty64'
  config.vm.box_url = 'https://github.com/mu-box/vagrant-packer-templates/releases/download/v0.2.0/trusty64_virtualbox.box'

  config.vm.provider 'virtualbox' do |v|
    v.memory = 8192
    v.cpus   = 4
  end

  config.vm.provider "vmware_fusion" do |v, override|
    v.vmx["memsize"] = "4096"
    v.vmx["numvcpus"] = "4"
    v.gui = false
    override.vm.box = "trusty64_vmware"
    override.vm.box_url = 'https://github.com/mu-box/vagrant-packer-templates/releases/download/v0.2.0/trusty64_vmware.box'
  end

  config.vm.provider "docker" do |v, override|
    v.build_dir = '.docker'
    v.create_args = ['--privileged']
    v.expose = [22]
    v.has_ssh = true

    override.vm.box = nil
    override.vm.box_url = nil
  end

  config.vm.network "private_network", type: "dhcp"

  microbox_user = ENV["MICROBOX_USER"] || 'microbox'
  microbox_project = ENV["MICROBOX_BASE_PROJECT"] || 'base'
  microbox_secret = ENV["MICROBOX_BASE_SECRET"]

  # cache
  config.vm.synced_folder "./.distfiles", "/content/distfiles"# , type: "nfs"
  config.vm.synced_folder "./.packages", "/content/packages"# , type: "nfs"

  # pkgsrc framework
  config.vm.synced_folder $pkgsrc, "/content/pkgsrc"# , type: "nfs"
  # package definitions
  config.vm.synced_folder ".", "/content/pkgsrc/base"# , type: "nfs"

  # utility scripts
  config.vm.synced_folder "./.scripts", "/opt/util"# , type: "nfs"

  # ssh keys
  config.vm.synced_folder "~/.ssh", "/var/.ssh"# , type: "nfs"

  config.vm.provision "shell", inline: <<-SCRIPT
    echo "Preparing Environment..."
    if [ ! -d /etc/profile.d ]; then
      mkdir -p /etc/profile.d/
      echo "if [ -d /etc/profile.d ]; then" >> /etc/profile
      echo "  for i in /etc/profile.d/*.sh; do" >> /etc/profile
      echo "    if [ -r \\$i ]; then" >> /etc/profile
      echo "      . \\$i" >> /etc/profile
      echo "    fi" >> /etc/profile
      echo "  done" >> /etc/profile
      echo "  unset i" >> /etc/profile
      echo "fi" >> /etc/profile
    fi
    echo # Vagrant environment variables > /etc/profile.d/vagrant.sh
    echo export PATH=/data/sbin:/data/bin:/data/gcc49/bin:/data/gnu/bin:$PATH >> /etc/profile.d/vagrant.sh
    echo export MICROBOX_USER=#{microbox_user} >> /etc/profile.d/vagrant.sh
    echo export MICROBOX_PROJECT=#{microbox_project} >> /etc/profile.d/vagrant.sh
    echo export MICROBOX_SECRET=#{microbox_secret} >> /etc/profile.d/vagrant.sh
    echo umask 022 >> /etc/profile.d/vagrant.sh
    chmod +x /etc/profile.d/vagrant.sh
  SCRIPT

  config.vm.provision "shell", inline: <<-SCRIPT
    echo "installing build tools..."
    apt-get -y update -qq
    apt-get -y upgrade
    apt-get install -y build-essential
  SCRIPT

  config.vm.provision 'shell', inline: <<-SCRIPT
    echo "Downloading and Extracting Bootstrap..."

    # detect platform
    if [ $(uname | grep 'SunOS') ]; then
      platform="SmartOS"
    elif [ $(uname | grep 'Linux') ]; then
      platform="Linux"
    fi

    # download and extract the bootstrap
    if [ ! -d /data ]; then
      curl \
        -s \
        https://pkgsrc.microbox.cloud/microbox/base/${platform}/bootstrap.tar.gz \
          | tar \
              -xz \
              -f - \
              -C /
    fi

    # ensure /data/var/db exists
    if [ ! -d /data/var/db ]; then
      mkdir -p /data/var/db
    fi
  SCRIPT

  config.vm.provision 'shell', inline: <<-SCRIPT
    echo "Install mksandbox utility..."
    if [ ! -f /data/sbin/mksandbox ]; then
      /data/bin/pkgin -y in mksandbox
    fi
  SCRIPT

  config.vm.provision 'shell', inline: <<-SCRIPT
    echo "Installing custom utilities..."
    for i in /opt/util/*; do
      cmd=$(basename ${i/.sh/})
      if [ ! -L /usr/bin/${cmd} ]; then
        ln -s ${i} /usr/bin/${cmd}
      fi
    done
  SCRIPT

  config.vm.provision 'shell', inline: <<-SCRIPT
    if [ -f /usr/bin/ruby ]; then
      echo "Purging the system installation of ruby"
      aptitude purge -y ruby
    fi
  SCRIPT

end

require 'yaml'

VAGRANTFILE_API_VERSION = "2"

CONFIG = YAML.load_file('config.yml')

Vagrant.configure("2") do |config|
  config.omnibus.chef_version = :latest

  CONFIG['machines'].each do |name, machine|
    config.vm.box_check_update = false

    config.vm.define name do |config|
      config.vm.box = machine['box']

      config.vm.hostname = [ 'mackerel-agent', name, 'vagrant', %x(uname -n).chomp ].join('.')

      config.vm.provision 'chef_solo' do |chef|
        chef.cookbooks_path = 'berks-cookbooks'
        chef.run_list = [
          'recipe[golang]',
          'recipe[build-essential]',
        ]
        chef.json = {
          go: {
            gopath:   '/home/vagrant',
            owner:    'vagrant',
            group:    'vagrant',
            platform: machine['platform']
          }
        }
      end

      config.vm.provision :shell, inline: <<-__INSTALL_RUBY__
        if which apt-get; then
          apt-get install -y ruby
        elif which yum; then
          yum install -y ruby
        fi
      __INSTALL_RUBY__

      config.vm.provision :shell, inline: 'chown -R vagrant:vagrant /home/vagrant/src'

      config.vm.synced_folder "#{ENV['GOPATH']}/src/#{CONFIG['package']}",
                              "/home/vagrant/src/#{CONFIG['package']}"
    end
  end
end

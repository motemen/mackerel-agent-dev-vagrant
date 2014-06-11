VAGRANTFILE_API_VERSION = "2"

MACHINES = {
  'precise64' => {
    box: 'hashicorp/precise64',
    platform: 'amd64'
  },
  'centos-6.5' => {
    box: 'chef/centos-6.5',
    platform: 'amd64'
  },
}

Vagrant.configure("2") do |config|
  config.omnibus.chef_version = :latest

  MACHINES.each do |name, machine|
    config.vm.box_check_update = false

    config.vm.define name do |config|
      config.vm.box = machine[:box]

      config.vm.hostname = [ 'mackerel-agent', name, 'vagrant', %x(uname -n).chomp ].join('.')

      config.vm.provision 'chef_solo' do |chef|
        chef.cookbooks_path = 'berks-cookbooks'
        chef.run_list = [
          'recipe[golang]',
          'recipe[build-essential]',
        ]
        chef.json = {
          go: {
            gopath: '/home/vagrant',
            owner: 'vagrant',
            group: 'vagrant',
            platform: machine[:platform]
          }
        }
      end

      config.vm.provision :shell, inline: <<-__BUILD_GO_386__
        if [ "$(go env GOARCH)" != '386' ] && ! go tool -n 8g; then
          cd $(go env GOROOT)/src
          GOARCH=386 ./make.bash
        fi
      __BUILD_GO_386__

      config.vm.provision :shell, inline: 'chown -R vagrant:vagrant /home/vagrant/src'

      config.vm.synced_folder "#{ENV['GOPATH']}/src/github.com/mackerelio/mackerel-agent",
                              '/home/vagrant/src/github.com/mackerelio/mackerel-agent'
    end
  end
end

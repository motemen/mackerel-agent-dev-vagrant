require 'yaml'

CONFIG = YAML.load_file('config.yml')

def cmd_on_vagrant(name, command)
  [
    'ssh', '-F', '.ssh_config', name, '--',
      '$SHELL', '-l', '-c',
        %Q('cd src/#{CONFIG['package']} && { #{command}; } 2>&1 | sed -u "s/^/[#{'%10s' % name}] /"')
  ]
end

file '.ssh_config' do
  sh 'vagrant ssh-config > .ssh_config'
end

task :default => [ :up, :watch ]

task :up do
  sh 'vagrant', 'up'
  sh 'vagrant ssh-config > .ssh_config'
end

task :provision do
  sh 'vagrant', 'reload', '--provision'
end

task :down do
  sh 'vagrant', 'halt'
end

task :test => [ '.ssh_config' ] do
  threads = CONFIG['machines'].map do |name,conf|
    Thread.new do
      puts "---> Running on #{name}"
      sh *cmd_on_vagrant(name, 'make test')
    end
  end
  threads.each { |t| t.join }
end

task :watch => [ '.ssh_config' ] do |t,args|
  require 'listen'
  require 'pathname'

  ios = CONFIG['machines'].map do |name,conf|
    puts "---> Listening to changes on #{name}"
    IO.popen cmd_on_vagrant(name, 'while read signal; do make test; done'), 'w'
  end

  source_dir = Pathname.new(ENV['GOPATH']) + 'src' + CONFIG['package']
  listener = Listen.to(source_dir) do |m,a,r|
    puts "---> Files changed: #{(m+a+r).map { |p| Pathname.new(p).relative_path_from(source_dir) }.join(' ')}"
    ios.each { |io| io.puts }
  end
  listener.only /\.go$/

  puts "---> Watching changes on #{source_dir}"

  listener.start

  trap 'INT' do
    ios.each do |io|
      Process.kill 'KILL', io.pid
    end

    exit
  end

  sleep
end

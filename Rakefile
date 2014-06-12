require 'yaml'

MACHINES = YAML.load_file('machines.yaml')
TARGET_PACKAGE = 'github.com/mackerelio/mackerel-agent'

def cmd_on_vagrant(name, command)
  [
    'vagrant', 'ssh', name, '--',
      '$SHELL', '-l', '-c', %Q('cd src/#{TARGET_PACKAGE} && { #{command}; } 2>&1 | sed -u "s/^/[#{'%10s' % name}] /"')
  ]
end

task :test do
  threads = MACHINES.map do |name,conf|
    sleep 0.5
    Thread.new do
      puts "---> Running on #{name}"
      sh *cmd_on_vagrant(name, 'make test')
    end
  end
  threads.each { |t| t.join }
end

task :watch do |t,args|
  require 'listen'
  require 'pathname'

  ios = MACHINES.map do |name,conf|
    puts "---> Listening to changes on #{name}"
    sleep 0.5 # wait slightly for vagrant starting
    IO.popen([
      *cmd_on_vagrant(name, 'while read signal; do make test; done')
    ], 'w')
  end

  source_dir = Pathname.new(ENV['GOPATH']) + 'src' + TARGET_PACKAGE
  listener = Listen.to(source_dir) do |m,a,r|
    puts "---> Files changed: #{(m+a+r).map { |p| Pathname.new(p).relative_path_from(source_dir) }.join(' ')}"
    ios.each { |io| io.puts }
  end
  listener.only /\.go$/
  listener.start

  trap 'INT' do
    ios.each do |io|
      Process.kill 'KILL', io.pid
    end

    exit
  end

  sleep
end


require 'bundler/setup'
require 'albacore'
require 'albacore/tasks/versionizer'
require 'albacore/ext/teamcity'
require 'yaml'
require 'net/ssh'
require 'json'

Albacore::Tasks::Versionizer.new :versioning

desc "Get semantic version number in TC"
task :tc_version_fix => :versioning do
  puts "##teamcity[buildNumber '#{ENV['BUILD_VERSION']}']"
end

def update_yum_repo rpm_path
  sh "scp #{rpm_path} deployer@yum:/var/yum/el6/x86_64"

  Net::SSH.start 'yum', 'deployer' do |ssh|
    channel = ssh.open_channel do |ch|
      ch.exec '/usr/bin/createrepo -c /var/yum/cache --verbose --workers 2 /var/yum/el6/x86_64' do |ch, success|
        raise 'could not execute command' unless success

        # "on_data" is called when the process writes something to stdout
        ch.on_data do |c, data|
          $stdout.print data
        end

        # "on_extended_data" is called when the process writes something to stderr
        ch.on_extended_data do |c, type, data|
          $stderr.print data
        end

        ch.on_close do
          puts 'closing channel to yumrepo'
        end
      end
    end
    channel.wait
  end
end

task :default do
  puts `bundle exec rake -T`
end

task :build_pkg do
  system 'fpm',
	%W|-s dir
	-t rpm
	-C pkg
	--name intelliplan.xvfb-daemon
	--iteration 1
  --depends xorg-x11-server-Xvfb
	--category service-wrappers
	--url https://github.com/Intelliplan/xvfb-daemond
	--version #{ENV['FORMAL_VERSION']}
	--architecture all
	--package pkg
	.|
end

task :publish_pkg do
	update_yum_repo 'pkg/*.rpm'
end 

desc "build the rpm without publishing it"
task :rpm_local => [:versioning, :build_pkg]

desc "build and publish the rpm"
task :rpm => [:rpm_local, :publish_pkg]

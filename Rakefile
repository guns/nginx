# -*- encoding: utf-8 -*-

def version
  File.read('src/core/nginx.h')[/^#define\s+NGINX_VERSION\s+"(.+)"/, 1]
end

def modules
  Dir[File.expand_path '../modules/*', __FILE__].select do |m|
    File.exists? File.join(m, 'config')
  end
end

task :default => :build

desc 'Create Nginx release directory'
task :release do
  sh 'make -f misc/GNUmakefile release'
end

desc 'Run subsequent tasks from nginx-%s' % version
task :chdir do
  Rake::Task['release'].execute if not File.directory? 'tmp/nginx-%s' % version
  chdir 'tmp/nginx-%s' % version
end

desc 'Configure nginx'
task :configure => [:release, :chdir] do
  cmd = %W[
    #{File.expand_path 'configure'}
    --prefix=#{ENV['PREFIX'] || '/opt/nginx'}
    --user=http
    --group=http
    --conf-path=/etc/nginx/nginx.conf
    --pid-path=/var/run/nginx.pid
    --lock-path=/var/lock/nginx.lock
    --http-log-path=/var/log/nginx/access.log
    --error-log-path=/var/log/nginx/error.log
    --http-client-body-temp-path=/var/lib/nginx/client
    --http-proxy-temp-path=/var/lib/nginx/proxy
    --http-fastcgi-temp-path=/var/lib/nginx/fastcgi
    --http-uwsgi-temp-path=/var/lib/nginx/uwsgi
    --http-scgi-temp-path=/var/lib/nginx/scgi
    --with-http_ssl_module
    --with-http_gzip_static_module
  ]

  cmd += modules.map { |m| '--add-module=%s' % m }

  puts cmd.join(" \\\n    ")
  system *cmd
end

desc 'Build Nginx'
task :build => :chdir do
  Rake::Task[:configure].execute unless File.exists? 'Makefile'
  sh 'make', '--jobs=%d' % (ENV['JOBS'] || 2).to_i
end

desc 'Install Nginx'
task :install => :build do
  system 'make install'
end

desc 'Upgrade Nginx'
task :upgrade => :build do
  system 'make upgrade'
end

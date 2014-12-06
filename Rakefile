# -*- encoding: utf-8 -*-

def version
  File.read('src/core/nginx.h')[/^#define\s+NGINX_VERSION\s+"(.+)"/, 1]
end

def modules
  Dir[File.expand_path '../modules/*', __FILE__].select do |m|
    File.exists? File.join(m, 'config')
  end
end

def num_processors
  File.read('/proc/cpuinfo').lines.grep(/^processor/).size
end

def env
  @env ||= {
    :prefix => ENV['PREFIX'] || '/opt/nginx',
    :ngx_conf_prefix => ENV['NGX_CONF_PREFIX'] || '/etc/nginx',
    :destdir => ENV['DESTDIR'] || '',
    :jobs => (ENV['JOBS'] || num_processors).to_i
  }
end

def with_dir dir
  mkdir_p dir
  yield dir
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
  # Following options reflect the order of ./configure --help
  cmd = [
    File.expand_path('configure'),
    "--prefix=#{env[:ngx_conf_prefix]}",
    "--sbin-path=#{env[:prefix]}/sbin/nginx",
    "--conf-path=#{env[:ngx_conf_prefix]}/nginx.conf",
    "--error-log-path=/var/log/nginx/error.log",
    "--pid-path=/var/run/nginx.pid",
    "--lock-path=/var/lock/nginx.lock",
    "--user=http",
    "--group=http",
    # "--build=NAME",
    # "--builddir=DIR",
    # "--with-rtsig_module",
    # "--with-select_module",
    # "--without-select_module",
    # "--with-poll_module",
    # "--without-poll_module",
    "--with-file-aio",
    "--with-ipv6",
    "--with-http_ssl_module",
    "--with-http_spdy_module",
    "--with-http_realip_module",
    "--with-http_addition_module",
    # "--with-http_xslt_module",
    "--with-http_image_filter_module",
    "--with-http_geoip_module",
    "--with-http_sub_module",
    # "--with-http_dav_module",
    "--with-http_flv_module",
    "--with-http_mp4_module",
    "--with-http_gunzip_module",
    "--with-http_gzip_static_module",
    # "--with-http_auth_request_module",
    "--with-http_random_index_module",
    "--with-http_secure_link_module",
    "--with-http_degradation_module",
    # "--with-http_stub_status_module",
    # "--without-http_charset_module",
    # "--without-http_gzip_module",
    "--without-http_ssi_module",
    "--without-http_userid_module",
    # "--without-http_access_module",
    # "--without-http_auth_basic_module",
    # "--without-http_autoindex_module",
    # "--without-http_geo_module",
    "--without-http_map_module",
    "--without-http_split_clients_module",
    # "--without-http_referer_module",
    # "--without-http_rewrite_module",
    # "--without-http_proxy_module",
    "--without-http_fastcgi_module",
    # "--without-http_uwsgi_module",
    "--without-http_scgi_module",
    # "--without-http_memcached_module",
    # "--without-http_limit_conn_module",
    # "--without-http_limit_req_module",
    # "--without-http_empty_gif_module",
    # "--without-http_browser_module",
    "--without-http_upstream_hash_module",
    "--without-http_upstream_ip_hash_module",
    "--without-http_upstream_least_conn_module",
    "--without-http_upstream_keepalive_module",
    # "--with-http_perl_module",
    # "--with-perl_modules_path=PATH",
    # "--with-perl=PATH",
    "--http-log-path=/var/log/nginx/access.log",
    "--http-client-body-temp-path=/var/lib/nginx/client-body",
    "--http-proxy-temp-path=/var/lib/nginx/proxy",
    "--http-fastcgi-temp-path=/var/lib/nginx/fastcgi",
    "--http-uwsgi-temp-path=/var/lib/nginx/uwsgi",
    "--http-scgi-temp-path=/var/lib/nginx/scgi",
    # "--without-http",
    # "--without-http-cache",
    # "--with-mail",
    # "--with-mail_ssl_module",
    "--without-mail_pop3_module",
    "--without-mail_imap_module",
    "--without-mail_smtp_module",
    # "--with-google_perftools_module",
    # "--with-cpp_test_module",
    # "--add-module=PATH",
    # "--with-cc=PATH",
    # "--with-cpp=PATH",
    # "--with-cc-opt=OPTIONS",
    # "--with-ld-opt=OPTIONS",
    # "--with-cpu-opt=CPU",
    # "--without-pcre",
    "--with-pcre",
    # "--with-pcre=DIR",
    # "--with-pcre-opt=OPTIONS",
    "--with-pcre-jit",
    # "--with-md5=DIR",
    # "--with-md5-opt=OPTIONS",
    # "--with-md5-asm",
    # "--with-sha1=DIR",
    # "--with-sha1-opt=OPTIONS",
    # "--with-sha1-asm",
    # "--with-zlib=DIR",
    # "--with-zlib-opt=OPTIONS",
    # "--with-zlib-asm=CPU",
    # "--with-libatomic",
    # "--with-libatomic=DIR",
    # "--with-openssl=DIR",
    # "--with-openssl-opt=OPTIONS",
    # "--with-debug",
  ]

  cmd += modules.map { |m| '--add-module=%s' % m }

  puts cmd.join(" \\\n    ")
  system *cmd
end

desc 'Build Nginx'
task :build => :chdir do
  Rake::Task[:configure].execute unless File.exists? 'Makefile'
  sh 'make', '--jobs=%d' % env[:jobs]
end

desc 'Install Nginx'
task :install => :build do
  # Main executable
  with_dir File.join(env[:destdir], env[:prefix], 'sbin') do |dir|
    cp 'objs/nginx', dir
  end

  # Man page
  with_dir File.join(env[:destdir], env[:prefix], 'share/man/man8') do |dir|
    cp 'objs/nginx.8', dir
  end

  # Default configuration files
  with_dir File.join(env[:destdir], env[:ngx_conf_prefix], 'default') do |dir|
    Dir['conf/*'].each do |conf|
      cp conf, dir
    end
  end

  # Systemd unit file
  with_dir File.join(env[:destdir], env[:prefix], 'lib/systemd/system') do |dir|
    cp 'objs/nginx.service', dir
  end

  # Run control file
  with_dir File.join(env[:destdir], env[:prefix], 'share/nginx/rc.d') do |dir|
    cp 'objs/nginx.rc', dir
  end

  # HTML
  with_dir File.join(env[:destdir], env[:prefix], 'share/nginx') do |dir|
    cp_r 'html', dir
  end

  # Create directories
  sh 'bash', '-c', 'mkdir -p %s/var/{log/nginx,lib/nginx}' % env[:destdir].shellescape
end

desc 'Upgrade Nginx'
task :upgrade => :build do
  system 'make upgrade'
end

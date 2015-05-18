# Maintainer: guns <self@sungpae.com>

pkgname=nginx-nerv
pkgver=
pkgrel=1
pkgdesc="Custom nginx build"
arch=('i686' 'x86_64')
url='http://nginx.org'
license=('custom')
groups=('nerv')
depends=('pcre' 'zlib' 'openssl')
provides=('nginx')
conflicts=('nginx')
replaces=('nginx-guns')
backup=('etc/logrotate.d/nginx')
install='nginx.install'

pkgver() {
	printf %s "$(git describe --long --tags | tr - .)"
}

build() {
	cd "$startdir"
	PREFIX='/usr' NGX_CONF_PREFIX='/etc/nginx' DESTDIR="$pkgdir/" rake build
}

package() {
	cd "$startdir"
	PREFIX='/usr' NGX_CONF_PREFIX='/etc/nginx' DESTDIR="$pkgdir/" rake install
	install -Dm644 contrib/logrotate "$pkgdir"/etc/logrotate.d/nginx
}

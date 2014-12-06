# Maintainer: guns <self@sungpae.com>

pkgname=nginx-guns
pkgver=
pkgrel=1
pkgdesc="Sung Pae's nginx build"
arch=('i686' 'x86_64')
url='http://nginx.org'
license=('custom')
groups=('guns')
depends=('pcre' 'zlib' 'openssl')
provides=('nginx')
conflicts=('nginx')
backup=('etc/logrotate.d/nginx')

pkgver() {
	printf %s "$(git describe --long --tags | tr - .)"
}

build() {
	cd "$startdir"
	PREFIX='/usr' NGX_CONF_PREFIX='/etc/nginx' DESTDIR="$pkgdir/" JOBS=4 rake build
}

package() {
	cd "$startdir"
	PREFIX='/usr' NGX_CONF_PREFIX='/etc/nginx' DESTDIR="$pkgdir/" JOBS=4 rake install
	install -Dm644 contrib/logrotate "$pkgdir"/etc/logrotate.d/nginx
}

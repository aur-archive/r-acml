# Maintainer: jdarch <jda -dot- cloud -plus- archlinux -at- gmail -dot- com>
# Based on the PKGBUILD for R

pkgname=r-acml
pkgver=3.1.0
pkgrel=1
pkgdesc="Language and environment for statistical computing and graphics, BLAS and LAPACK from ACML"
arch=('i686' 'x86_64')
license=('GPL')
url=('http://www.r-project.org/')
provides=('r=3.1.0')
conflicts=('r')
depends=('acml-gfortran' 'amdlibm' 'bzip2'  'libpng' 'libjpeg' 'libtiff'
         'ncurses' 'pcre' 'readline' 'zlib' 'perl' 'gcc-libs'
         'libxt' 'libxmu' 'pango' 'xz' 'desktop-file-utils' 'zip' 'unzip')
makedepends=('jdk7-openjdk' 'gcc-fortran' 'tk')
optdepends=('tk: tcl/tk interface' 'texlive-bin: latex sty files')
backup=('etc/R/Makeconf' 'etc/R/Renviron' 'etc/R/ldpaths' 'etc/R/repositories' 'etc/R/javaconf')
options=('!makeflags' '!emptydirs')
install=r-acml.install

source=("http://cran.r-project.org/src/base/R-${pkgver%%.*}/R-${pkgver}.tar.gz"
	'r.desktop'
	'r.png'
	'R.conf')
md5sums=('a1ee52446bee81820409661e6d114ab1'
         'f23f172a35249321c4ccecb712397e74'
         '00659f39e90627fe87f1645df5d4e3a7'
         '1dfa62c812aed9642f6e4ac34999b9fe')
sha512sums=('bb21fc90c7d37a5328031ed784e7dcbd20259d1837c33db3b51c14a116939a53496683d5de142a1223e89fc12406294efc67bed3595131615e9607d5ffab5ce2'
            '365d96f987f99729181272304fa3176f43ab1bccb9c29e6f361ed292d98d0349da79a20fd7536f34846f5df8a27fc4f5d41b522b5b982b292096f6b898abbb62'
            '55ed6e819dcbb231d842d825134b84d1a24db177558d5dad9369d57fd21d0239d6433c4311531171a101ca3c7c0685493e9cc6c1fe9e4e0df59f2331cff150ba'
            'aae388c5b6c02d9fb857914032b0cd7d68a9f21e30c39ba11f5a29aaf1d742545482054b57ce18872eabb6605bbb359b2fc1e9be5ce6881443fdbdf6b67fab3b')

prepare() {
   cd R-${pkgver}
   # set texmf dir correctly in makefile
   sed -i 's|$(rsharedir)/texmf|${datarootdir}/texmf|' share/Makefile.in
   # fix for texinfo 5.X
   sed -i 's|test ${makeinfo_version_min} -lt 7|test ${makeinfo_version_min} -lt 0|' configure
}

build() {
   export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/acml/gfortran
   cd R-${pkgver}
   ./configure  --prefix=/usr \
		--libdir=/usr/lib \
		--sysconfdir=/etc/R \
		--datarootdir=/usr/share \
		  rsharedir=/usr/share/R/ \
		  rincludedir=/usr/include/R/ \
		  rdocdir=/usr/share/doc/R/ \
                --with-x \
		--enable-R-shlib \
		--with-blas="-I/usr/include/acml/gfortran -L/usr/lib/acml/gfortran -lacml_mp -lamdlibm -lm" \
		--with-lapack \
                F77=gfortran \
		LIBnn=lib

   # Place AMD's math library prior to GLIBC's libm 
   sed -i "s/\(^\| \)-lm\( \|$\)/\1-lamdlibm -lm\2/g" {./,etc/}Makeconf

   # Build the package
   make

   # make libRmath.so
   cd src/nmath/standalone
   make shared
}

check() {
   cd R-${pkgver}
   make check-recommended
}

package() {
   cd R-${pkgver}
   make DESTDIR="${pkgdir}" install

   # install libRmath.so
   cd src/nmath/standalone
   make DESTDIR="${pkgdir}" install

   #  Fixup R wrapper scripts.
   sed -i "s|${pkgdir} ||" "${pkgdir}/usr/bin/R"
   rm "${pkgdir}/usr/lib/R/bin/R"
   cd "${pkgdir}/usr/lib/R/bin"
   ln -s ../../../bin/R

   # install some freedesktop.org compatibility
   install -Dm644 "${srcdir}/r.desktop" \
        "${pkgdir}/usr/share/applications/r.desktop"
   install -Dm644 "${srcdir}/r.png" \
        "${pkgdir}/usr/share/pixmaps/r.png"

   # move the config directory to /etc and create symlinks
   install -d "${pkgdir}/etc/R"
   cd "${pkgdir}/usr/lib/R/etc"
   for i in *; do
     mv -f ${i} "${pkgdir}/etc/R"
     ln -s /etc/R/${i} ${i}
   done

   # Install ld.so.conf.d file to ensure other applications access the shared lib
   install -Dm644 "${srcdir}/R.conf" "${pkgdir}/etc/ld.so.conf.d/R.conf"
}

# Maintainer: Thomas Dziedzic <gostrc@gmail.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: tobias [ tobias at archlinux org ]
# Contributor: Daniel J Griffiths <ghost1227@archlinux.us>

pkgbase=vim
pkgname=('vim' 'gvim' 'vim-runtime')
_topver=7.4
_patchlevel=135
__hgrev=de28b1568fc2
_versiondir="vim${_topver//./}"
pkgver=${_topver}.${_patchlevel}
pkgrel=1
arch=('i686' 'x86_64')
license=('custom:vim')
url="http://www.vim.org"
makedepends=('gpm' 'python2' 'libxt')
source=("ftp://ftp.archlinux.org/other/vim/vim-${pkgver}.tar.xz"
        "ftp://ftp.archlinux.org/other/vim/vim-${pkgver}.tar.xz.sig"
        'vimrc'
        'archlinux.vim'
        'gvim.desktop')
md5sums=('be672ce2a929503412378c998fc3dc27'
         'SKIP'
         'b9d4dcb9d3ee2e151dc4be1e94934f6a'
         '10353a61aadc3f276692d0e17db1478e'
         'd90413bd21f400313a785bb4010120cd')

# source PKGBUILD && mksource
mksource() {
  [[ -x /usr/bin/hg ]] || (echo "hg not found. Install mercurial." && return 1)

  __hgroot='http://vim.googlecode.com/hg/'
  __hgrepo='vim'
  __hgbranch='default'

  hg clone -b ${__hgbranch} -u ${__hgrev} "${__hgroot}${__hgrepo}" ${__hgrepo}

  pushd ${__hgrepo}
  if (( $(hg id -n) < $(hg id -nr ${__hgbranch}) )); then
    printf 'You are not building the latest revision!\n'
    printf "Consider updating __hgrev to $(hg id -r ${__hgbranch}).\n"
  fi
  popd

  mv vim ${pkgname}-${pkgver}
  find ${pkgname}-${pkgver} -depth -type d -name .hg -exec rm -rf {} \;
  rm ${pkgname}-${pkgver}/{.hgignore,.hgtags}
  tar -cJf ${pkgname}-${pkgver}.tar.xz ${pkgname}-${pkgver}/*
  rm -r ${pkgname}-${pkgver}

  gpg --detach-sign ${pkgname}-${pkgver}.tar.xz

  scp ${pkgname}-${pkgver}.tar.xz nym:/srv/ftp/other/vim/
  scp ${pkgname}-${pkgver}.tar.xz.sig nym:/srv/ftp/other/vim/
}

build() {
  cp -a ${pkgname}-${pkgver} vim-build

  # define the place for the global (g)vimrc file (set to /etc/vimrc)
  sed -i 's|^.*\(#define SYS_.*VIMRC_FILE.*"\) .*$|\1|' \
    vim-build/src/feature.h
  sed -i 's|^.*\(#define VIMRC_FILE.*"\) .*$|\1|' \
    vim-build/src/feature.h

  (cd vim-build/src && autoconf)

  cp -a vim-build gvim-build

  cd "${srcdir}"/vim-build

  ./configure \
    --prefix=/usr \
    --localstatedir=/var/lib/vim \
    --with-features=huge \
    --with-compiledby='Arch Linux' \
    --enable-gpm \
    --enable-acl \
    --with-x=no \
    --disable-gui \
    --enable-multibyte \
    --enable-cscope \
    --disable-netbeans \
    --enable-perlinterp \
    --enable-pythoninterp \
    --enable-python3interp \
    --disable-rubyinterp \
    --disable-luainterp

  make

  cd "${srcdir}"/gvim-build

  ./configure \
    --prefix=/usr \
    --localstatedir=/var/lib/vim \
    --with-features=huge \
    --with-compiledby='Arch Linux' \
    --enable-gpm \
    --enable-acl \
    --with-x=yes \
    --enable-gui=gtk2 \
    --enable-multibyte \
    --enable-cscope \
    --enable-netbeans \
    --enable-perlinterp \
    --enable-pythoninterp \
    --disable-python3interp \
    --enable-rubyinterp \
    --enable-luainterp

  make
}

check() {
  # disable tests because they seem to freeze

  cd "${srcdir}"/vim-build

  #make test

  cd "${srcdir}"/gvim-build

  #make test
}

package_vim() {
  pkgdesc='Vi Improved, a highly configurable, improved version of the vi text editor'
  depends=("vim-runtime=${pkgver}-${pkgrel}" 'gpm')
  conflicts=('gvim')

  cd "${srcdir}"/vim-build
  make -j1 VIMRCLOC=/etc DESTDIR="${pkgdir}" install

  # provided by (n)vi in core
  rm "${pkgdir}"/usr/bin/{ex,view}

  # delete some manpages
  find "${pkgdir}"/usr/share/man -type d -name 'man1' 2>/dev/null | \
    while read _mandir; do
    cd ${_mandir}
    rm -f ex.1 view.1 # provided by (n)vi
    rm -f evim.1    # this does not make sense if we have no GUI
  done

  # Runtime provided by runtime package
  rm -r "${pkgdir}"/usr/share/vim

  # license
  install -Dm644 "${srcdir}"/vim-${pkgver}/runtime/doc/uganda.txt \
    "${pkgdir}"/usr/share/licenses/${pkgname}/license.txt
}

package_gvim() {
  pkgdesc='Vi Improved, a highly configurable, improved version of the vi text editor (with advanced features, such as a GUI)'
  depends=("vim-runtime=${pkgver}-${pkgrel}" 'gpm' 'ruby' 'libxt'
           'desktop-file-utils' 'gtk2' 'lua' 'python2')
  provides=("vim=${pkgver}-${pkgrel}")
  conflicts=('vim')
  install=gvim.install

  cd "${srcdir}"/gvim-build
  make -j1 VIMRCLOC=/etc DESTDIR="${pkgdir}" install

  # provided by (n)vi in core
  rm "${pkgdir}"/usr/bin/{ex,view}

  # delete some manpages
  find "${pkgdir}"/usr/share/man -type d -name 'man1' 2>/dev/null | \
    while read _mandir; do
    cd ${_mandir}
    rm -f ex.1 view.1 # provided by (n)vi
  done

  # Move the runtime for later packaging
  mv "${pkgdir}"/usr/share/vim "${srcdir}"/runtime-install

  # freedesktop links
  install -Dm644 "${srcdir}"/gvim.desktop \
    "${pkgdir}"/usr/share/applications/gvim.desktop
  install -Dm644 runtime/vim48x48.png "${pkgdir}"/usr/share/pixmaps/gvim.png

  # license
   install -Dm644 "${srcdir}"/vim-${pkgver}/runtime/doc/uganda.txt \
    "${pkgdir}"/usr/share/licenses/${pkgname}/license.txt
}

package_vim-runtime() {
  pkgdesc='Runtime for vim and gvim'
  depends=('perl' 'gawk')
  backup=('etc/vimrc')

  # Install the runtime split from gvim
  install -dm755 "${pkgdir}"/usr/share
  mv "${srcdir}"/runtime-install "${pkgdir}"/usr/share/vim

  # Don't forget logtalk.dict
  install -Dm644 "${srcdir}"/gvim-build/runtime/ftplugin/logtalk.dict \
    "${pkgdir}"/usr/share/vim/${_versiondir}/ftplugin/logtalk.dict

  # fix FS#17216
  sed -i 's|messages,/var|messages,/var/log/messages.log,/var|' \
    "${pkgdir}"/usr/share/vim/${_versiondir}/filetype.vim

  # patch filetype.vim for better handling of pacman related files
  sed -i "s/rpmsave/pacsave/;s/rpmnew/pacnew/;s/,\*\.ebuild/\0,PKGBUILD*,*.install/" \
    "${pkgdir}"/usr/share/vim/${_versiondir}/filetype.vim
  sed -i "/find the end/,+3{s/changelog_date_entry_search/changelog_date_end_entry_search/}" \
    "${pkgdir}"/usr/share/vim/${_versiondir}/ftplugin/changelog.vim

  # rc files
  install -Dm644 "${srcdir}"/vimrc "${pkgdir}"/etc/vimrc
  install -Dm644 "${srcdir}"/archlinux.vim \
    "${pkgdir}"/usr/share/vim/vimfiles/archlinux.vim

  # rgb.txt file
  install -Dm644 "${srcdir}"/vim-${pkgver}/runtime/rgb.txt \
    "${pkgdir}"/usr/share/vim/${_versiondir}/rgb.txt

  # license
  install -dm755 "${pkgdir}"/usr/share/licenses/vim-runtime
  ln -s /usr/share/vim/${_versiondir}/doc/uganda.txt \
    "${pkgdir}"/usr/share/licenses/vim-runtime/license.txt
}

# vim:set sw=2 sts=2 et:

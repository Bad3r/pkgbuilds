# Maintainer: Xuanwo <xuanwo@archlinuxcn.org>
# Acknowledgment: Borrowed a lot from logseq-desktop-git, thank @pychuang 
pkgname=logseq-desktop
pkgver=0.7.0
pkgrel=1
pkgdesc="A privacy-first, open-source platform for knowledge sharing and management."
arch=("x86_64")
url="https://github.com/logseq/logseq"
license=('AGPL3')
makedepends=("git" "yarn" "npm" "clojure" "nodejs-lts-gallium")
provides=("logseq-desktop")
conflicts=("logseq-desktop-git" "logseq-desktop-bin")
source=("${pkgname}-${pkgver}.zip::https://github.com/logseq/logseq/archive/refs/tags/${pkgver}.zip"
      "build.patch"
      "${pkgname}.desktop")
sha256sums=('66f1c2785730506b1fb507b1fa5a01e204061f41c3ceeb47056b57de35a2373c'
            'b26c6ed39e2635e08a0df83d92883e670b75b02ed1c2c279044909c04edf8fc2'
            '6e834466132551c721ba2ffe92fc0f81056b3151fe6b5f0f469ece937f9b7e84')

prepare() {
    cd "$srcdir/logseq-${pkgver}"

    # patch :parallel-build true in shadow-cljs.edn
    patch -p1 -i "${srcdir}/build.patch"

    # download required js modules
    yarn install

    # create and sync files to folder `static`
    yarn gulp:build

    # go to folder `static` and download required js modules in static
    cd "${srcdir}/logseq-${pkgver}/static"
    yarn install

    # go back to the top-level folder and download clojure dependencies
    cd "${srcdir}/logseq-${pkgver}"
    clojure -P -M:cljs
}

build() {
    cd "${srcdir}/logseq-${pkgver}"

    # build
    yarn cljs:release

    # packaging javescript files to an executable
    cd "${srcdir}/logseq-${pkgver}/static"
    yarn electron-forge package
}

package() {
    # important files are under static/out/Logseq-linux-x64 
    cd "${srcdir}/logseq-${pkgver}/static/out/Logseq-linux-x64"

    # create destination folder and copy files
    mkdir -p "${pkgdir}/opt/${pkgname}"
    cp -a -r -u --verbose ./ "${pkgdir}/opt/${pkgname}"

    # create a soft link to the executable
    mkdir -p "${pkgdir}/usr/bin"
    ln -s "/opt/${pkgname}/Logseq" "${pkgdir}/usr/bin/logseq"

    # create license folder and make soft links to actual license
    mkdir -p "${pkgdir}/usr/share/licenses/${pkgname}"
    ln -s "/opt/${pkgname}/LICENSE" "${pkgdir}/usr/share/licenses/${pkgname}"
    ln -s "/opt/${pkgname}/LICENSES.chromium.html" "${pkgdir}/usr/share/licenses/${pkgname}"

    # install readme and additional license file (the top-level AGPL3)
    cd "${srcdir}/logseq-${pkgver}"
    install -Dm644 "README.md" -t "${pkgdir}/usr/share/doc/${pkgname}"
    install -Dm644 "LICENSE.md" -t "${pkgdir}/usr/share/licenses/${pkgname}"

    # copy xdg desktop files
    cd "${srcdir}"
    mkdir -p "${pkgdir}/usr/share/applications"
    install -Dm644 "logseq-desktop.desktop" -t "${pkgdir}/usr/share/applications"
}

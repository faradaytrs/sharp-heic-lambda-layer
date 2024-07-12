# Library Versions
WEBP_VERSION=1.4.0
LIBDE265_VERSION=1.0.15
LIBAOM_VERSION=3.9.1
X265_VERSION=3.6
DAV1D_VERSION=1.4.3
RAV1E_VERSION=0.7.1
LIBHEIF_VERSION=1.18.0
VIPS_VERSION=8.15.2
SHARP_VERSION=0.33.4

# print all values of version variables
print-versions:
	@echo "WEBP_VERSION=$(WEBP_VERSION)"
	@echo "LIBDE265_VERSION=$(LIBDE265_VERSION)"
	@echo "LIBAOM_VERSION=$(LIBAOM_VERSION)"
	@echo "X265_VERSION=$(X265_VERSION)"
	@echo "DAV1D_VERSION=$(DAV1D_VERSION)"
	@echo "RAV1E_VERSION=$(RAV1E_VERSION)"
	@echo "LIBHEIF_VERSION=$(LIBHEIF_VERSION)"
	@echo "VIPS_VERSION=$(VIPS_VERSION)"
	@echo "SHARP_VERSION=$(SHARP_VERSION)"

PREFIX_PATH=/usr/local
LIB_PATH=$(PREFIX_PATH)/lib
LIBRARY_PATH=$(LIB_PATH)

export PKG_CONFIG_PATH=$(LIB_PATH)/pkgconfig:$$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=$(LIB_PATH):$$LD_LIBRARY_PATH

build-SharpHEICLayer: libvips
	mkdir -p "$(ARTIFACTS_DIR)/nodejs"
	mkdir -p "$(ARTIFACTS_DIR)/lib"

	# sharp uses several of the libs we installed or compiled. extract the full list and copy all of those into $(ARTIFACTS_DIR)/lib
	# extract list with ldd from sharp.node, manipulate a bit to only get the absolute paths, then copy

	# install
	npm install --save --prefix "$(ARTIFACTS_DIR)/nodejs/" node-addon-api node-gyp
	npm install --build-from-source --prefix "$(ARTIFACTS_DIR)/nodejs/" --verbose --foreground-scripts --platform=linux --arch=x64 sharp@$(SHARP_VERSION)

	# debug logs
	# ls -la $(ARTIFACTS_DIR)/nodejs/node_modules/sharp/src/build/*/

	# copy the shared objects
	ldd $(ARTIFACTS_DIR)/nodejs/node_modules/sharp/src/build/Release/sharp-linux-x64.node | awk '{ print $$3 }' | grep '/' | xargs -I '{}' cp '{}' $(ARTIFACTS_DIR)/lib

	# debug logs
	# ls -la $(ARTIFACTS_DIR)/lib/

libwebp:
	# libwebp
	curl -L https://github.com/webmproject/libwebp/archive/v$(WEBP_VERSION).tar.gz | tar zx
	cd libwebp-$(WEBP_VERSION) && ./autogen.sh && ./configure --enable-libwebpmux --prefix=$(PREFIX_PATH) && make V=0 && make install

libde265:
	# libde265
	curl -L https://github.com/strukturag/libde265/releases/download/v$(LIBDE265_VERSION)/libde265-$(LIBDE265_VERSION).tar.gz | tar zx
	cd libde265-$(LIBDE265_VERSION) && ./autogen.sh && ./configure --disable-dec265 --disable-sherlock265 --prefix=$(PREFIX_PATH) && make V=0 && make install
x265:
	# x265
	curl -L https://bitbucket.org/multicoreware/x265_git/downloads/x265_$(X265_VERSION).tar.gz | tar zx
	cd x265_$(X265_VERSION) && cd build/linux && cmake -G "Unix Makefiles" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$(PREFIX_PATH) ../../source && make && make install
dav1d:
	# dav1d
	curl -L https://code.videolan.org/videolan/dav1d/-/archive/$(DAV1D_VERSION)/dav1d-$(DAV1D_VERSION).tar.gz | tar zx
	cd dav1d-$(DAV1D_VERSION) && meson build --default-library=static --buildtype release --prefix "$(PREFIX_PATH)" && ninja -C build && ninja -C build install
rav1e: installRust
	# rav1e
	curl -L https://github.com/xiph/rav1e/archive/v${RAV1E_VERSION}.tar.gz | tar zx
	cd rav1e-${RAV1E_VERSION} && source /root/.cargo/env && cargo cinstall --crt-static --release --prefix="$(PREFIX_PATH)" --library-type=staticlib
libheif: x265 libde265 dav1d rav1e
	# libheif
	curl -L https://github.com/strukturag/libheif/releases/download/v$(LIBHEIF_VERSION)/libheif-$(LIBHEIF_VERSION).tar.gz | tar zx && \
	cd libheif-$(LIBHEIF_VERSION) && mkdir build && cd build && cmake -DCMAKE_INSTALL_LIBDIR=$(LIB_PATH) -DWITH_DAV1D=1 -DWITH_RAV1E=1 -DWITH_AOM_DECODER=0 -DWITH_AOM_ENCODER=0 --preset=release-noplugins .. && make V=0 && make install
libvips: installLib libheif libwebp
	# libvips
	curl -L https://github.com/libvips/libvips/releases/download/v$(VIPS_VERSION)/vips-$(VIPS_VERSION).tar.xz | tar xJ
	cd vips-$(VIPS_VERSION) && meson setup build --prefix ${PREFIX_PATH} --libdir ${LIB_PATH} -Dheif=enabled -Dheif-module=disabled -Dlcms=enabled --buildtype release && cd build && meson compile && meson install
installRust:
	# rust
	curl https://sh.rustup.rs -sSf | sh -s -- -y && \
	source $(HOME)/.cargo/env && \
	cargo install cargo-c
installLib:
	dnf groupinstall -y "Development Tools" && \
	dnf install -y \
		glib2-devel \
		expat-devel \
		libjpeg-turbo-devel \
		libpng-devel \
		giflib-devel \
		libexif-devel \
		librsvg2-devel \
		libtiff-devel \
		lcms2-devel \
		gobject-introspection-devel \
		cmake \
		nasm \
		pkg-config \
		meson \
		ninja-build


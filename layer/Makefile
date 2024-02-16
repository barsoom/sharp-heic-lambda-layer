MAKEFLAGS+="-j $(shell nproc)"

# Library Versions
WEBP_VERSION=1.3.2# https://github.com/webmproject/libwebp/tags
LIBDE265_VERSION=1.0.15# https://github.com/strukturag/libde265/tags
LIBHEIF_VERSION=1.17.6# https://github.com/strukturag/libheif/tags
LIBVIPS_VERSION=8.15.1# https://github.com/libvips/libvips/tags
SHARP_VERSION=0.33.2# https://github.com/lovell/sharp/tags

PREFIX_PATH=/opt

export PKG_CONFIG_PATH=$(PREFIX_PATH)/lib/pkgconfig

build-SharpHEICLayer: libvips
	mkdir -p "$(ARTIFACTS_DIR)/nodejs"
	mkdir -p "$(ARTIFACTS_DIR)/lib"

	# sharp uses several of the libs we installed or compiled. extract the full list and copy all of those into /opt/lib
	# extract list with ldd from sharp.node, manipulate a bit with sed to only get the absolute paths, then copy
	LD_LIBRARY_PATH=$(PREFIX_PATH)/lib npm --prefix "$(ARTIFACTS_DIR)/nodejs/" install sharp@$(SHARP_VERSION) && ldd $(ARTIFACTS_DIR)/nodejs/node_modules/@img/sharp-linux-x64/lib/sharp-linux-x64.node | sed -nE "s/^[^\/\n]*(\/[^ ]+)(.*)/\1/p" | xargs cp -t $(ARTIFACTS_DIR)/lib

libwebp:
	curl -L https://github.com/webmproject/libwebp/archive/v$(WEBP_VERSION).tar.gz | tar zx
	cd libwebp-$(WEBP_VERSION) && ./autogen.sh && ./configure --enable-libwebpmux --prefix=$(PREFIX_PATH) && make V=0 && make install

libde265:
	# libde265
	curl -L https://github.com/strukturag/libde265/releases/download/v$(LIBDE265_VERSION)/libde265-$(LIBDE265_VERSION).tar.gz | tar zx
	cd libde265-$(LIBDE265_VERSION) && ./autogen.sh && ./configure --disable-dec265 --disable-sherlock265 --prefix=$(PREFIX_PATH) && make V=0 && make install

libheif: libde265
	# libheif
	curl -L https://github.com/strukturag/libheif/releases/download/v$(LIBHEIF_VERSION)/libheif-$(LIBHEIF_VERSION).tar.gz | tar zx
	cd libheif-$(LIBHEIF_VERSION) && cmake -DCMAKE_INSTALL_PREFIX=$(PREFIX_PATH) . && make

libvips: installLib libheif libwebp
	# libvips
	curl -L https://github.com/libvips/libvips/releases/download/v$(LIBVIPS_VERSION)/vips-$(LIBVIPS_VERSION).tar.xz | tar Jx
	cd vips-$(LIBVIPS_VERSION) && mkdir builddir && meson setup ../builddir . --prefix=$(PREFIX_PATH) && ls && cd ../builddir && ninja && ninja test && ninja install

installLib:
	dnf install -y \
		cmake \
		glib2-devel \
		expat-devel \
		libjpeg-devel \
		libjpeg-turbo-devel \
		libpng-devel \
		giflib-devel \
		libexif-devel \
		librsvg2-devel \
		libtiff-devel \
		meson \
		ninja-build \
		lcms2-devel \
		file-devel # This used to ben libmagic-devel


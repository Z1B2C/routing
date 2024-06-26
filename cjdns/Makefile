#
# Copyright (C) 2014,2015 Hyperboria.net
#
# You may redistribute this program and/or modify it under the terms of
# the GNU General Public License as published by the Free Software Foundation,
# either version 3 of the License, or (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#

include $(TOPDIR)/rules.mk

PKG_NAME:=cjdns
PKG_VERSION:=21.1
PKG_RELEASE:=6

PKG_SOURCE:=$(PKG_NAME)-$(PKG_VERSION).tar.gz
PKG_SOURCE_URL:=https://codeload.github.com/cjdelisle/cjdns/tar.gz/$(PKG_NAME)-v$(PKG_VERSION)?
PKG_HASH:=a6158ce7847159aa44e86f74ccc7b6ded6910a230ed8f3830db53cda5838f0b0
PKG_BUILD_DIR:=$(BUILD_DIR)/$(PKG_NAME)-$(PKG_NAME)-v$(PKG_VERSION)

PKG_MAINTAINER:=William Fleurant <meshnet@protonmail.com>
PKG_LICENSE:=GPL-3.0-or-later
PKG_LICENSE_FILES:=LICENSE

include $(INCLUDE_DIR)/package.mk

define Package/cjdns
	SECTION:=net
	CATEGORY:=Network
	SUBMENU:=Routing and Redirection
	TITLE:=Encrypted near-zero-conf mesh routing protocol
	URL:=https://github.com/cjdelisle/cjdns
	DEPENDS:=@!arc @IPV6 +kmod-tun +libnl-tiny +libpthread +librt \
		+libuci-lua +lua-bencode +dkjson +luasocket +lua-sha2
endef

define Package/cjdns/description
	Cjdns implements an encrypted IPv6 network using public-key cryptography \
	for address allocation and a distributed hash table for routing. \
	This provides near-zero-configuration networking, and prevents many \
	of the security and scalability issues that plague existing networks.
endef

define Package/cjdns-tests
	SECTION:=net
	CATEGORY:=Network
	SUBMENU:=Routing and Redirection
	TITLE:=cjdns test cases
	URL:=https://github.com/cjdelisle/cjdns
	DEPENDS:=+libpthread +librt @!arc
endef

define Package/cjdns-test/description
	Builds cjdns test cases binary test_testcjdroute_c
endef

define Build/Configure
endef

PKG_DO_VARS:=CJDNS_RELEASE_VERSION=$(PKG_SOURCE_VERSION)

ifneq ($(CONFIG_KERNEL_SECCOMP_FILTER),y)
PKG_DO_VARS+= Seccomp_NO=1
endif

ifneq ($(CONFIG_USE_UCLIBC),)
PKG_DO_VARS+= UCLIBC=1
endif

define Build/Compile
	$(INSTALL_DIR) $(PKG_BUILD_DIR)/tmp
	(cd $(PKG_BUILD_DIR) && \
	CROSS="true" \
	CC="$(TARGET_CC_NOCACHE)" \
	AR="$(TARGET_AR)" \
	RANLIB="$(TARGET_RANLIB)" \
	CFLAGS="$(TARGET_CFLAGS) -U_FORTIFY_SOURCE -Wno-error=array-bounds -Wno-error=stringop-overflow -Wno-error=stringop-overread" \
	LDFLAGS="$(TARGET_LDFLAGS)" \
	SYSTEM="linux" \
	TARGET_ARCH="$(CONFIG_ARCH)" \
	SSP_SUPPORT="$(CONFIG_SSP_SUPPORT)" \
	GYP_ADDITIONAL_ARGS="-f make-linux" \
	CJDNS_BUILD_TMPDIR="$(PKG_BUILD_DIR)/tmp" \
	$(PKG_DO_VARS) \
	exec ./do)
endef

define Package/cjdns/install
	$(INSTALL_DIR) \
		$(1)/usr/sbin \
		$(1)/usr/bin \
		$(1)/etc/config \
		$(1)/etc/init.d \
		$(1)/etc/uci-defaults \
		$(1)/usr/lib/lua/cjdns \
		$(1)/usr/share/ucitrack

	$(INSTALL_BIN) \
		./files/cjdrouteconf \
		$(1)/usr/bin

	$(INSTALL_BIN) \
		$(PKG_BUILD_DIR)/cjdroute \
		$(1)/usr/sbin

	$(INSTALL_BIN) \
		$(PKG_BUILD_DIR)/publictoip6 \
		$(1)/usr/bin

	$(INSTALL_BIN) \
		./files/cjdns.init \
		$(1)/etc/init.d/cjdns

	$(INSTALL_BIN) \
		./files/cjdns.defaults \
		$(1)/etc/uci-defaults/cjdns

	$(INSTALL_DATA) \
		./files/luci-app-cjdns.json \
		$(1)/usr/share/ucitrack

	$(CP) \
		./lua/cjdns/* \
		$(1)/usr/lib/lua/cjdns
endef

define Package/cjdns/postinst
#!/bin/sh
if [ -z $${IPKG_INSTROOT} ] ; then
	( . /etc/uci-defaults/cjdns ) && rm -f /etc/uci-defaults/cjdns
	/etc/init.d/cjdns enabled || /etc/init.d/cjdns enable
	exit 0
fi
endef

define Package/cjdns-tests/install
	$(INSTALL_DIR) $(1)/usr/bin
	$(INSTALL_BIN) \
		$(PKG_BUILD_DIR)/build_linux/test_testcjdroute_c \
		$(1)/usr/bin
endef

$(eval $(call BuildPackage,cjdns))
$(eval $(call BuildPackage,cjdns-tests))

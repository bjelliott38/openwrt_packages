# Copyright (c) 2017 Stan Grishin (stangri@melmac.net)
# This is free software, licensed under the GNU General Public License v3.

include $(TOPDIR)/rules.mk

PKG_NAME:=vpn-policy-routing
PKG_VERSION:=0.0.2
PKG_RELEASE:=4
PKG_LICENSE:=GPL-3.0+
PKG_MAINTAINER:=Stan Grishin <stangri@melmac.net>

include $(INCLUDE_DIR)/package.mk

define Package/vpn-policy-routing
	SECTION:=net
	CATEGORY:=Network
	DEPENDS:=+ipset +iptables +resolveip +kmod-ipt-ipset +!BUSYBOX_CONFIG_IP:ip-full
	CONFLICTS:=vpnbypass
	TITLE:=VPN Policy-Based Routing Service
	PKGARCH:=all
endef

define Package/vpn-policy-routing/description
This service allows policy-based routing for L2TP, Openconnect, OpenVPN, PPTP and Wireguard tunnels and WAN interface.
Policies can specify domains, local IPs/subnets and ports, as well as remote IPs/subnets and ports.
endef

define Package/vpn-policy-routing/conffiles
/etc/config/vpn-policy-routing
endef

define Build/Prepare
	mkdir -p $(PKG_BUILD_DIR)/files/
	$(CP) ./files/vpn-policy-routing.init $(PKG_BUILD_DIR)/files/vpn-policy-routing.init
	sed -i "s|^\(PKG_VERSION\).*|\1='$(PKG_VERSION)-$(PKG_RELEASE)'|" $(PKG_BUILD_DIR)/files/vpn-policy-routing.init
endef

define Build/Configure
endef

define Build/Compile
endef

define Package/vpn-policy-routing/install
	$(INSTALL_DIR) $(1)/etc/init.d
	$(INSTALL_BIN) $(PKG_BUILD_DIR)/files/vpn-policy-routing.init $(1)/etc/init.d/vpn-policy-routing
	$(INSTALL_DIR) $(1)/etc/config
	$(INSTALL_CONF) ./files/vpn-policy-routing.conf $(1)/etc/config/vpn-policy-routing
	$(INSTALL_DIR) $(1)/etc/hotplug.d/firewall
	$(INSTALL_DATA) ./files/vpn-policy-routing.hotplug $(1)/etc/hotplug.d/firewall/99-vpn-policy-routing
endef

define Package/vpn-policy-routing/postinst
	#!/bin/sh
	# check if we are on real system
	if [ -z "$${IPKG_INSTROOT}" ]; then
		/etc/init.d/vpn-policy-routing enable

		while [ ! -z "$(uci -q get ucitrack.@vpn-policy-routing[-1] 2>/dev/null)" ] ; do
			uci -q delete ucitrack.@vpn-policy-routing[-1]
			uci commit ucitrack
		done

		while [ ! -z "$(uci -q get ucitrack.@firewall[-1].affects 2>/dev/null | awk '/vpn-policy-routing/')" ] ; do
			uci -q del_list ucitrack.@firewall[-1].affects='vpn-policy-routing'
			uci commit ucitrack
		done

		uci -q batch <<-EOF >/dev/null
			add ucitrack vpn-policy-routing
			set ucitrack.@vpn-policy-routing[-1].init='vpn-policy-routing'
			add_list ucitrack.@firewall[-1].affects='vpn-policy-routing'
			commit ucitrack
	EOF
	fi
	exit 0
endef

define Package/vpn-policy-routing/prerm
	#!/bin/sh
	# check if we are on real system
	if [ -z "$${IPKG_INSTROOT}" ]; then
		echo "Stopping service and removing rc.d symlink for vpn-policy-routing"
		/etc/init.d/vpn-policy-routing stop || true
		/etc/init.d/vpn-policy-routing disable || true

		while [ ! -z "$(uci -q get ucitrack.@vpn-policy-routing[-1] 2>/dev/null)" ] ; do
			uci -q delete ucitrack.@vpn-policy-routing[-1]
			uci commit ucitrack
		done

		while [ ! -z "$(uci -q get ucitrack.@firewall[-1].affects 2>/dev/null | awk '/vpn-policy-routing/')" ] ; do
			uci -q del_list ucitrack.@firewall[-1].affects='vpn-policy-routing'
			uci commit ucitrack
		done
	fi
	exit 0
endef

$(eval $(call BuildPackage,vpn-policy-routing))

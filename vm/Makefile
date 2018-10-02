UDEVDIR := $(shell pkg-config --variable=udevdir udev)
ifeq ($(UDEVDIR),)
$(error empty UDEVDIR variable)
endif


default:

install:
	install -d $(DESTDIR)/etc/qubes-rpc $(DESTDIR)$(UDEVDIR)/rules.d
	install etc/qubes-rpc/splitDmCrypt.* $(DESTDIR)/etc/qubes-rpc
	install -m 644 etc/split-dm-crypt.conf $(DESTDIR)/etc
	install -m 644 rules.d/00-blockdev-parsing-disabled.rules $(DESTDIR)$(UDEVDIR)/rules.d

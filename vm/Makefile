UDEVDIR := $(shell pkg-config --variable=udevdir udev)
ifeq ($(UDEVDIR),)
$(error empty UDEVDIR variable)
endif

LEGACY = sbin/backport-UDEV_DISABLE_PERSISTENT_STORAGE_RULES_FLAG-rule


default:

$(LEGACY): $(LEGACY).in
	sed 's:@UDEVDIR@:$(UDEVDIR):' $< >$@
	chmod +x $@

clean:
	rm -f $(LEGACY)

install: clean $(LEGACY)
	install -d $(DESTDIR)/usr/sbin $(DESTDIR)/etc/qubes-rpc $(DESTDIR)$(UDEVDIR)/rules.d
	install sbin/blockdev-parsing-disabled $(LEGACY) $(DESTDIR)/usr/sbin
	install etc/qubes-rpc/splitDmCrypt.* $(DESTDIR)/etc/qubes-rpc
	install -m 644 etc/split-dm-crypt.conf $(DESTDIR)/etc
	install -m 644 rules.d/00-blockdev-parsing-disabled.rules $(DESTDIR)$(UDEVDIR)/rules.d

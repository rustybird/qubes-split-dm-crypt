ifeq ($(PREFIX),)
    ifeq ($(shell qubesdb-read /qubes-vm-persistence),full)
        PREFIX = /usr
    else
        PREFIX = /usr/local
    endif
else
    # strip a trailing slash, e.g. from tab completion
    override PREFIX := $(PREFIX:/=)
endif

ifeq ($(PREFIX),/usr)
    etc = /etc
else
ifeq ($(PREFIX),/usr/local)
    etc = $(PREFIX)/etc
else
    $(error PREFIX must be /usr or /usr/local)
endif
endif

.ONESHELL:
.SHELLFLAGS += -e -x
.SILENT:

default:

install:
	umask 022
	mkdir -p -- '$(DESTDIR)$(PREFIX)/' '$(DESTDIR)$(etc)/'
	cp -R -- usr/* '$(DESTDIR)$(PREFIX)/'
	cp -R -- etc/* '$(DESTDIR)$(etc)/'

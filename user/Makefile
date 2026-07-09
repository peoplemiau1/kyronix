ROOTFS := ../rootfs

ifeq ($(USE_COREUTILS),1)
COREUTILS_TARGET := coreutils
KYROBOX_UTILS := awk clear cmp find grep killall less nc nslookup ping ps reboot sed which wget
else
COREUTILS_TARGET :=
KYROBOX_UTILS := awk basename cat chgrp chmod chown cksum clear cmp cp cut date dd dirname du echo env false \
	find grep head hostname kill killall less link ln ls mkdir mktemp mv nc nslookup ping printenv printf ps pwd readlink reboot rm rmdir \
	sed seq sleep sort sync tail tee test touch tr true tty uname uniq unlink wc wget which whoami yes
endif

.PHONY: all clean

all: $(COREUTILS_TARGET)
	$(MAKE) -C init
	$(MAKE) -C shell
	$(MAKE) -C login
	$(MAKE) -C vi
	$(MAKE) -C kyrobox USE_COREUTILS=$(USE_COREUTILS)
	$(MAKE) -C fetch
	$(MAKE) -C pipes
	mkdir -p $(ROOTFS)/bin
	cp ../build/bin/kyronix-init $(ROOTFS)/bin/init
	cp ../build/bin/ksh $(ROOTFS)/bin/
	cp ../build/bin/login $(ROOTFS)/bin/
	cp ../build/bin/vi $(ROOTFS)/bin/
	cp ../build/bin/fetch $(ROOTFS)/bin/
	cp ../build/bin/make $(ROOTFS)/bin/
	cp ../build/bin/pipes $(ROOTFS)/usr/bin/
	ln -sf ksh $(ROOTFS)/bin/sh
	for app in $(KYROBOX_UTILS); do \
	    cp ../build/bin/$$app $(ROOTFS)/bin/; \
	done
ifeq ($(USE_COREUTILS),1)
	cp ../build/bin/coreutils $(ROOTFS)/bin/
	for app in $$(../build/bin/coreutils --list); do \
	    ln -sf coreutils $(ROOTFS)/bin/$$app; \
	done
endif

coreutils:
	if [ ! -d ../coreutils ]; then git clone https://github.com/uutils/coreutils/ ../coreutils; fi
	cargo build --manifest-path ../coreutils/Cargo.toml --release --target x86_64-unknown-linux-musl --features feat_os_unix_musl
	mkdir -p ../build/bin
	cp ../coreutils/target/x86_64-unknown-linux-musl/release/coreutils ../build/bin/coreutils

clean:
	$(MAKE) -C init clean
	$(MAKE) -C shell clean
	$(MAKE) -C login clean
	$(MAKE) -C vi clean
	$(MAKE) -C kyrobox clean
	$(MAKE) -C fetch clean
	$(MAKE) -C pipes clean
	# keep st and cwm as blobs
	for f in $(ROOTFS)/bin/*; do \
	    case "$$(basename $$f)" in st|cwm) ;; *) rm -f "$$f";; esac; \
	done
	rm -f $(ROOTFS)/usr/bin/pipes
	if [ -d ../coreutils ]; then cargo clean --manifest-path ../coreutils/Cargo.toml 2>/dev/null || true; fi

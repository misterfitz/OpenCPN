#
# Build the opencpn repo 
#
# NOTE: this is supposed to be run from the build dir, like
#
#   cd build; make -f ../flatpak/Makefile ...
#

BRANCH          ?= master
GPG_KEY         ?= leamas@opencpn.org
DESTDIR         ?= website
GPG_HOMEDIR     ?= $(shell readlink -fn gpg)
RSYNC_RSH       ?= ssh
RSYNC_HOST      ?= amazon-ec2
REPO_URL        ?= http://opencpn.duckdns.org/opencpn-beta

all: build sign

build:
	flatpak-builder --repo=$(CURDIR)/repo --force-clean \
	    app ../flatpak/org.opencpn.OpenCPN.yaml
	-flatpak uninstall --user -y org.opencpn.OpenCPN.Locale
	flatpak install -y --user --reinstall \
	    $(CURDIR)/repo org.opencpn.OpenCPN

install:
	rm -rf $(DESTDIR); mkdir $(DESTDIR)
	cp ../flatpak/repo/*flat* ../flatpak/repo/index.html $(DESTDIR)
	cp -ar repo $(DESTDIR)

sign:   install
	@test -d $(GPG_HOMEDIR) || { \
	    echo "cannot find GPG_HOMEDIR (abort)"; exit 1; \
	}
	flatpak build-sign --gpg-sign=$(GPG_KEY) --gpg-homedir=$(GPG_HOMEDIR)  \
	    $(CURDIR)/website/repo
	flatpak build-update-repo --gpg-sign=$(GPG_KEY) --prune \
	    --gpg-homedir=$(GPG_HOMEDIR) \
	    $(CURDIR)/website/repo
	@for file in $(CURDIR)/website/opencpn.flatpakrepo \
           $(CURDIR)/website/opencpn.flatpakref;\
        do \
	    echo "Signing $$file"; \
	    sed -i '/GPGKey/d' $$file; \
	    echo -n "GPGKey=" >> $$file	; \
	    gpg2 --homedir=$(GPG_HOMEDIR) --export $(GPG_KEY) | \
	        base64 --wrap=0 | tr -d '\n' >> $$file; \
	done
	echo "Exporting gpg key"
	gpg2 --export -a --homedir=$(GPG_HOMEDIR) $(GPG_KEY) \
	    > $(DESTDIR)/opencpn.key


repourl:
	sed -i '/^Url/s|=.*|=$(REPO_URL)|' repo/opencpn.flatpakref

publish:
	rsync -a  --info=stats --rsh="$(RSYNC_RSH)" $(DESTDIR)/ \
	    $(RSYNC_HOST):/var/www/ocpn-flatpak/website

.phony:

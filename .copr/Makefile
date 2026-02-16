# COPR Makefile for automated splitrandr builds from GitHub master
# Builds Python GUI app + fakexrandr LD_PRELOAD library

# COPR provides outdir, set default for local testing
outdir ?= .

# Default target for COPR
srpm:
	@echo "=== Starting splitrandr SRPM build from GitHub master ==="
	@echo "Output directory: $(outdir)"
	@echo "Working directory: $$(pwd)"

	# Install git if not available
	@if ! command -v git >/dev/null 2>&1; then \
		echo "=== Git not found, installing it ==="; \
		if command -v dnf >/dev/null 2>&1; then \
			dnf install -y git || exit 1; \
		elif command -v yum >/dev/null 2>&1; then \
			yum install -y git || exit 1; \
		fi; \
	fi

	# Clone splitrandr from GitHub
	@echo "=== Cloning splitrandr from GitHub ==="; \
	rm -rf splitrandr-master; \
	git clone --depth=50 https://github.com/DigitalCyberSoft/splitrandr.git splitrandr-master || exit 1; \
	\
	echo "=== Getting commit info ==="; \
	COMMIT=$$(cd splitrandr-master && git rev-parse HEAD); \
	SHORTCOMMIT=$$(cd splitrandr-master && git rev-parse --short HEAD); \
	VERSION=$$(cd splitrandr-master && python3 -c "import re; m=re.search(r\"__version__\s*=\s*'([^']+)'\", open('splitrandr/meta.py').read()); print(m.group(1))" 2>/dev/null || echo "0.1.0"); \
	echo "Commit: $$COMMIT"; \
	echo "Short commit: $$SHORTCOMMIT"; \
	echo "Version: $$VERSION"; \
	\
	echo "=== Determining release number ==="; \
	FULL_VERSION="$$VERSION"_"$$SHORTCOMMIT"; \
	FEDORA_VER=$$(rpm -E %fedora 2>/dev/null || echo "42"); \
	echo "Checking COPR for existing builds of version $$FULL_VERSION on Fedora $$FEDORA_VER"; \
	COPR_URL="https://download.copr.fedorainfracloud.org/results/reversejames/splitrandr/fedora-$$FEDORA_VER-x86_64"; \
	BUILD_DIRS=$$(curl -s "$$COPR_URL/" 2>/dev/null | grep -oE '[0-9]+-splitrandr' | head -10); \
	EXISTING_RELEASES=""; \
	for dir in $$BUILD_DIRS; do \
		RPMS=$$(curl -s "$$COPR_URL/$$dir/" 2>/dev/null | grep -oE "splitrandr-$$VERSION-[0-9]+\.git$$SHORTCOMMIT\.fc[0-9]+\.x86_64\.rpm" | head -1); \
		if [ -n "$$RPMS" ]; then \
			REL=$$(echo "$$RPMS" | sed "s/.*-$$VERSION-\([0-9]\+\)\.git$$SHORTCOMMIT\.fc[0-9]\+\.x86_64\.rpm/\1/"); \
			EXISTING_RELEASES="$$EXISTING_RELEASES $$REL"; \
		fi; \
	done; \
	if [ -n "$$EXISTING_RELEASES" ]; then \
		MAX_REL=$$(echo $$EXISTING_RELEASES | tr ' ' '\n' | sort -n | tail -1); \
		RELEASE_NUM=$$(($$MAX_REL + 1)); \
		echo "Found existing release $$MAX_REL, incrementing to $$RELEASE_NUM"; \
	else \
		RELEASE_NUM=1; \
		echo "No existing builds found, starting with release 1"; \
	fi; \
	\
	echo "=== Creating source tarball ==="; \
	tar czf splitrandr-$$SHORTCOMMIT.tar.gz -C . splitrandr-master \
		--transform "s/^splitrandr-master/splitrandr-$$COMMIT/" || exit 1; \
	\
	echo "=== Generating spec file ==="; \
	( \
	echo "%global commit $$COMMIT"; \
	echo '%global shortcommit %(c=%{commit}; echo $${c:0:7})'; \
	echo ''; \
	echo 'Name:           splitrandr'; \
	echo "Version:        $$VERSION"; \
	echo "Release:        $$RELEASE_NUM.git%{shortcommit}%{?dist}"; \
	echo 'Summary:        Monitor Layout Editor with Virtual Monitor Splitting'; \
	echo ''; \
	echo 'License:        GPL-3.0-or-later'; \
	echo 'URL:            https://github.com/DigitalCyberSoft/splitrandr'; \
	echo 'Source0:        splitrandr-%{shortcommit}.tar.gz'; \
	echo ''; \
	echo 'BuildRequires:  python3-devel'; \
	echo 'BuildRequires:  gcc'; \
	echo 'BuildRequires:  pkgconfig(xrandr)'; \
	echo 'BuildRequires:  pkgconfig(x11)'; \
	echo 'BuildRequires:  pkgconfig(xinerama)'; \
	echo 'BuildRequires:  pkgconfig(xcb-randr)'; \
	echo 'BuildRequires:  desktop-file-utils'; \
	echo ''; \
	echo 'Requires:       python3'; \
	echo 'Requires:       python3-gobject'; \
	echo 'Requires:       gtk3'; \
	echo 'Requires:       xrandr'; \
	echo 'Requires:       libXrandr'; \
	echo 'Requires:       gdbus'; \
	echo ''; \
	echo 'Recommends:     libappindicator-gtk3'; \
	echo ''; \
	echo '%description'; \
	echo 'SplitRandR is a GTK3 display layout editor for X11 that extends ARandR'; \
	echo 'with the ability to split physical monitors into virtual monitors using'; \
	echo 'xrandr --setmonitor. Includes a vendored fakexrandr LD_PRELOAD library'; \
	echo 'that makes window managers (particularly Cinnamon/Muffin) see the virtual'; \
	echo 'monitors as separate physical outputs for proper taskbar and window tiling.'; \
	echo ''; \
	echo '%prep'; \
	echo '%autosetup -n splitrandr-%{commit}'; \
	echo ''; \
	echo '%build'; \
	echo '# Build fakexrandr library'; \
	echo 'cd fakexrandr'; \
	echo './configure || exit 1'; \
	echo 'make %{?_smp_mflags} || exit 1'; \
	echo 'cd ..'; \
	echo ''; \
	echo '%install'; \
	echo '# Install Python package'; \
	echo 'install -d %{buildroot}%{python3_sitelib}/splitrandr'; \
	echo 'install -d %{buildroot}%{python3_sitelib}/splitrandr/data'; \
	echo 'install -p -m 0644 splitrandr/*.py %{buildroot}%{python3_sitelib}/splitrandr/'; \
	echo 'install -p -m 0644 splitrandr/data/gpl-3.txt %{buildroot}%{python3_sitelib}/splitrandr/data/'; \
	echo ''; \
	echo '# Install executable'; \
	echo 'install -d %{buildroot}%{_bindir}'; \
	echo 'cat > %{buildroot}%{_bindir}/splitrandr << '"'"'WRAPPER'"'"''; \
	echo '#!/usr/bin/env python3'; \
	echo 'from splitrandr.gui import main'; \
	echo 'main()'; \
	echo 'WRAPPER'; \
	echo 'chmod 0755 %{buildroot}%{_bindir}/splitrandr'; \
	echo ''; \
	echo '# Install desktop file'; \
	echo 'install -d %{buildroot}%{_datadir}/applications'; \
	echo 'install -p -m 0644 splitrandr/data/splitrandr.desktop %{buildroot}%{_datadir}/applications/'; \
	echo 'desktop-file-validate %{buildroot}%{_datadir}/applications/splitrandr.desktop || true'; \
	echo ''; \
	echo '# Install fakexrandr library'; \
	echo 'FAKE_DIR=%{buildroot}%{_prefix}/local/lib64'; \
	echo 'install -d $$FAKE_DIR'; \
	echo 'install -p -m 0755 fakexrandr/libXrandr.so $$FAKE_DIR/libXrandr.so'; \
	echo 'ln -sf libXrandr.so $$FAKE_DIR/libXrandr.so.2'; \
	echo 'ln -sf libXrandr.so $$FAKE_DIR/libXinerama.so.1'; \
	echo ''; \
	echo '%files'; \
	echo '%license splitrandr/data/gpl-3.txt'; \
	echo '%{_bindir}/splitrandr'; \
	echo '%{python3_sitelib}/splitrandr/'; \
	echo '%{_datadir}/applications/splitrandr.desktop'; \
	echo '%{_prefix}/local/lib64/libXrandr.so'; \
	echo '%{_prefix}/local/lib64/libXrandr.so.2'; \
	echo '%{_prefix}/local/lib64/libXinerama.so.1'; \
	echo ''; \
	echo '%post'; \
	echo 'ldconfig'; \
	echo ''; \
	echo '%postun'; \
	echo 'ldconfig'; \
	echo ''; \
	echo '%changelog'; \
	echo "* $$(date '+%a %b %d %Y') COPR Builder - $$VERSION-$$RELEASE_NUM.git%{shortcommit}"; \
	echo '- Automated build from GitHub master branch'; \
	echo '- Git commit: %{commit}'; \
	) > splitrandr.spec || exit 1; \
	echo "Spec file generated"; \
	\
	echo "=== Building SRPM ==="; \
	rpmbuild -bs \
		--define "_sourcedir $$(pwd)" \
		--define "_specdir $$(pwd)" \
		--define "_builddir $$(pwd)" \
		--define "_srcrpmdir $(outdir)" \
		--define "_rpmdir $$(pwd)" \
		splitrandr.spec || exit 1; \
	\
	echo "=== SRPM build complete ==="; \
	ls -la $(outdir)/*.src.rpm

clean:
	@rm -f *.tar.gz *.spec *.src.rpm *.rpm
	@rm -rf splitrandr-master

.PHONY: srpm clean

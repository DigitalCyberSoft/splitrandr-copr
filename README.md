# splitrandr-copr

COPR build configuration for [SplitRandR](https://github.com/DigitalCyberSoft/splitrandr) — a GTK3 monitor layout editor with virtual monitor splitting.

## Installation

```bash
dnf copr enable reversejames/splitrandr
dnf install splitrandr
```

## What gets installed

- `/usr/bin/splitrandr` — GUI application
- `/usr/lib/python3.*/site-packages/splitrandr/` — Python package
- `/usr/share/applications/splitrandr.desktop` — Start menu entry
- `/usr/local/lib64/libXrandr.so` — fakexrandr LD_PRELOAD library (makes window managers see virtual monitors)

## Local SRPM build (testing)

```bash
cd .copr
make srpm
```

## How it works

The COPR Makefile clones the latest splitrandr from GitHub, extracts the version from `meta.py`, generates an RPM spec file, and builds a source RPM. COPR then builds the binary RPM from that.

The package includes the vendored fakexrandr library which is compiled from C source during the RPM build.

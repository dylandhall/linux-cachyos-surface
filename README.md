# CachyOS Kernel for Surface Devices

This repository includes the files needed to build an optimized CachyOS kernel including custom patches for Microsoft Surface devices. The patches are based on the work from the linux-surface repository. The patches have been updated to ensure compatibility with CachyOS's patches.

The Microsoft Surface patches and per-version config fragment are pulled directly from the official [linux-surface/linux-surface](https://github.com/linux-surface/linux-surface) repository at build time. The git ref to check out is controlled by the `_surface_ref` variable in each `PKGBUILD`:

- `linux-cachyos-surface` builds against the pre-patched kernel tarball published by CachyOS at [CachyOS/linux/releases](https://github.com/CachyOS/linux/releases) (e.g. `cachyos-6.19.12-2.tar.gz`). This tarball is vanilla Linux with all the CachyOS base optimisations (BORE, BBR3, cachy, fixes, ntsync, t2, zstd, amd-cache-optimizer, …) pre-applied — it replaces the now-removed `0001-cachyos-base-all.patch` for kernel 6.18+. The packaging revision is selected by `_tagrel` (independent of this PKGBUILD's `pkgrel`). The linux-surface tag to check out is built from `_surface_ver` (kernel version part, e.g. `6.19.8`) and `_surface_rel` (e.g. `3`), giving a default `_surface_ref="arch-${_surface_ver}-${_surface_rel}"`. These are split because cachyos and linux-surface don't always publish the same kernel point release.
- `linux-cachyos-surface-lts` deliberately stays on a real kernel.org LTS line (currently Linux 6.12.x, supported by upstream LTS through approximately December 2026). It uses the stock kernel.org tarball plus the still-present `${_major}/all/0001-cachyos-base-all.patch` meta-patch from `cachyos/kernel-patches` — that meta-patch was retired for 6.18+ but remains available for 6.12. The PKGBUILD includes a commented-out template for the pre-baked tarball switch (along the lines used by the mainline variant) for the future kernel bump past 6.12. Its `_surface_ref` defaults to `master` because upstream's `arch_lts-*` tag series stopped at 4.19. Note: upstream `CachyOS/linux-cachyos`'s own `linux-cachyos-lts` variant has redefined "lts" to mean "previous stable cachyos kernel"; this repo intentionally keeps the original kernel.org-LTS meaning so Surface owners get the longest maintenance window per major bump.

To bump the mainline kernel: pick a tag from [CachyOS/linux/releases](https://github.com/CachyOS/linux/releases), update `_major`/`_minor`/`_tagrel`, then point `_surface_ver`/`_surface_rel` at the matching tag from [linux-surface/linux-surface/tags](https://github.com/linux-surface/linux-surface/tags) (the closest one for the same major.minor is fine — surface patches live in `patches/${_major}/` regardless of the tag's point release). The patch set itself is discovered automatically from `patches/${_major}/*.patch` in the upstream repo.

_**NOTE:** The configuration files and prebuilt kernels are optimized for X86_64_v3 instruction sets, this should be fine for most Surface devices, but might not work on very old (1st or 2nd gen) devices._

## Variants

### linux-cachyos-surface

This variant is as close to the original cachyos kernel as possible, it is build using Clang LTO mode `full` with llvm for maximum performance.

### linux-cachyos-surface-lts

This variant is based on the original cachyos lts kernel, but is build with gcc for better stability and support.

## Installation Instructions

### Build from source

To build the kernel and header files from source, run the following commands within CachyOS (or any Arch derivative):

```bash
sudo pacman -S base-devel
git clone https://github.com/jonpetersathan/linux-cachyos-surface
cd linux-cachyos-surface/linux-cachyos-surface
makepkg -si --skipinteg
```

Or alternatively using docker:

```bash
docker run --name kernelbuild -v $PWD:/pkg cachyos/docker-makepkg-v3
sudo pacman -U linux-cachyos-surface-*.pkg.tar.zst
```

_**NOTE:** Per default the linux-cachyos-surface kernel is configured in LTO mode `full`, this may take a bit longer to compile and requires more ram. It can be changed by updating the following line in the PKGBUILD:_
```bash
# Clang LTO mode, only available with the "llvm" compiler - options are "none", "full" or "thin".
# ATTENTION - one of three predefined values should be selected!
# "full: uses 1 thread for Linking, slow and uses more memory, theoretically with the highest performance gains."
# "thin: uses multiple threads, faster and uses less memory, may have a lower runtime performance than Full."
# "thin-dist: Similar to thin, but uses a distributed model rather than in-process: https://discourse.llvm.org/t/rfc-distributed-thinlto-build-for-kernel/85934"
# "none: disable LTO
: "${_use_llvm_lto:=full}"
```

### Install prebuilt packages

You can also just install one of the prebuilt kernels by downloading the kernel and header files from [here](https://github.com/jonpetersathan/linux-cachyos-surface/releases) and run:

```bash
sudo pacman -U linux-cachyos-surface-*.pkg.tar.zst
```

## Acknowledgements

- Maximilian Luz: [surface-linux/surface-linux](https://github.com/linux-surface/linux-surface)
- Peter Lung: [CachyOS/linux-cachyos](https://github.com/CachyOS/linux-cachyos)

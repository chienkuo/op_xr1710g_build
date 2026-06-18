# OpenWrt XR1710G GitHub Actions Build

This repository contains a GitHub Actions workflow for building OpenWrt firmware for the XR1710G on a Linux runner.

The default build source is:

- OpenWrt tree: `https://github.com/YYH2913/openwrt.git`
- Branch: `xr1710g`
- Target: `airoha/an7581`
- Device profile: `econet_xr1710g-ubi`

## Usage

1. Push this repository to GitHub.
2. Push to the `main` branch to start the first test build automatically.
3. Open `Actions`.
4. Run `Build OpenWrt XR1710G` manually later if you want to override inputs.
5. Download the `openwrt-xr1710g-*` artifact after the build completes.

The firmware workflow also runs once per day to check the configured OpenWrt branch. Scheduled runs skip the expensive build when the current upstream commit already has a matching release tag. Successful firmware builds publish a GitHub Release containing firmware files and a `openwrt-xr1710g-packages.tar.zst` package archive.

A separate toolchain workflow runs once per week. It builds the OpenWrt host tools and target toolchain, saves them into the GitHub Actions cache, and uploads a `openwrt-xr1710g-toolchain-*.tar.zst` archive to a GitHub Release. Firmware builds first try to restore that toolchain from cache, then from the matching toolchain release, and fall back to a normal build when no prebuilt toolchain is available.

The expected system firmware artifact is the `*-sysupgrade.itb` file. For XR1710G HTTP Recovery, upload that `*-sysupgrade.itb` file.

## Notes

- Do the OpenWrt source checkout and build on the Ubuntu runner. Avoid cloning the full OpenWrt tree on macOS case-insensitive filesystems.
- The workflow enables OpenWrt buildbot-style package output with `CONFIG_ALL`, `CONFIG_ALL_KMODS`, and `CONFIG_ALL_NONSHARED`.
- OpenClash is added from <https://github.com/vernesong/OpenClash> and selected into the firmware as `luci-app-openclash`.
- Firmware release titles use `路由器固件 <build time>`. Toolchain release titles use `toolchain <build time>`.
- `actions/cache` cache misses and Node runtime deprecation messages are runner warnings, not build failures.
- OpenWrt dependency warnings from package Makefiles are expected when all packages are scanned. The actual failure signal is a later `ERROR` or failed workflow step.
- The workflow adds a first-boot wireless defaults script that sets a valid country code, defaulting to `CN`.
- The 5 GHz radio is optionally pinned to channel `36` on first boot to avoid DFS startup delays and client discovery issues.
- U-Boot chainloader images are separate from this system firmware workflow. Do not flash raw U-Boot artifacts as XR1710G system firmware.

## References

- YYH2913 OpenWrt source: <https://github.com/YYH2913/openwrt/tree/xr1710g>
- W1700K UBI build workflow reference: <https://github.com/OpenWRT-fanboy/w1700k-ubi-build>
- XR1710G U-Boot and HTTP Recovery notes: <https://github.com/YYH2913/http-uboot-xr1710g>
- OpenWrt XR1710G PR: <https://github.com/openwrt/openwrt/pull/22397>

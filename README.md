# OpenWrt XR1710G GitHub Actions Build

This repository contains a GitHub Actions workflow for building OpenWrt firmware for the XR1710G on a Linux runner.

The default build source is:

- ImmortalWrt tree: `https://github.com/immortalwrt/immortalwrt.git`
- Branch: `openwrt-24.10`
- Target: `airoha/an7581`
- Device profile: `gemtek_xr1710g-ubi`

## Usage

1. Push this repository to GitHub.
2. Push to the `main` branch to register the workflows.
3. Open `Actions`.
4. After the toolchain succeeds, the firmware and package workflows run automatically.
5. Run `Build OpenWrt XR1710G` manually later if you want to override inputs.
6. Download the `openwrt-xr1710g-*` artifact after the build completes.

Successful firmware builds publish a GitHub Release containing firmware files and the exported OpenWrt `.config`.

A source watcher workflow checks the ImmortalWrt `openwrt-24.10` branch once per hour. When the latest upstream commit does not already have matching toolchain, firmware, and package releases, it sends an `openwrt-updated` repository dispatch event to start the normal toolchain -> firmware/packages build chain. The watcher can also be run manually with `force_build` to rebuild even when matching releases already exist.

A separate toolchain workflow runs once per week, on repository dispatch, and after build workflow changes. It deletes matching old Actions cache keys at the start of the next toolchain run, then builds the OpenWrt host tools and target toolchain, saves the fresh toolchain into Actions cache, and uploads a profile-specific `openwrt-xr1710g-toolchain-*.tar.zst` archive to a GitHub Release. Automatic firmware and package builds run only after a successful toolchain build and first try to restore the matching toolchain from cache, then from the matching toolchain release. Manual firmware/package runs still attempt a normal OpenWrt build when no prebuilt toolchain is available.

Packages are built by a separate matrix workflow. It splits real OpenWrt package Makefiles across shards so one GitHub-hosted runner job does not have to compile every package before the six-hour job limit.

Firmware, toolchain, and package shard releases all include the OpenWrt `.config` used for that build.

The expected system firmware artifact is the `*-sysupgrade.itb` file. For XR1710G HTTP Recovery, upload that `*-sysupgrade.itb` file.

## Notes

- Do the OpenWrt source checkout and build on the Ubuntu runner. Avoid cloning the full OpenWrt tree on macOS case-insensitive filesystems.
- The main build tree defaults to ImmortalWrt. If that tree does not contain `gemtek_xr1710g-ubi`, the workflow imports the Gemtek XR1710G device profile, DTS, common image definitions, and board files from the OpenWrt XR1710G PR head.
- The packages workflow enables OpenWrt buildbot-style package output with `CONFIG_ALL`, `CONFIG_ALL_KMODS`, and `CONFIG_ALL_NONSHARED`.
- OpenClash is added from <https://github.com/vernesong/OpenClash> and selected into the firmware as `luci-app-openclash`.
- The default config requires `luci-app-openclash`, `luci-app-mlo`, `luci-app-airoha-npu`, and `luci-app-w1700k-fancontrol`.
- MLO LuCI is added from <https://github.com/YYH2913/luci-app-mlo>.
- Airoha NPU LuCI is added from <https://github.com/rchen14b/luci-app-airoha-npu>.
- W1700K fan control is added from <https://github.com/rchen14b/luci-app-w1700k-fancontrol>.
- Firmware release titles use `路由器固件 <build time>`. Toolchain release titles use `toolchain <build time>`.
- After uploading Release assets, workflows clean local release staging directories. Toolchain cache files are retained until the next toolchain workflow run begins.
- `actions/cache` cache misses and Node runtime deprecation messages are runner warnings, not build failures.
- OpenWrt dependency warnings from package Makefiles are expected when all packages are scanned. The actual failure signal is a later `ERROR` or failed workflow step.
- The workflow adds a first-boot wireless defaults script that sets a valid country code, defaulting to `CN`.
- The 5 GHz radio is optionally pinned to channel `36` on first boot to avoid DFS startup delays and client discovery issues.
- U-Boot chainloader images are separate from this system firmware workflow. The workflow removes the `chainload-uboot.itb` artifact from the XR1710G system firmware profile, because it needs a separately built U-Boot binary. Do not flash raw U-Boot artifacts as XR1710G system firmware.

## References

- ImmortalWrt source: <https://github.com/immortalwrt/immortalwrt/tree/openwrt-24.10>
- Gemtek XR1710G OpenWrt PR overlay: <https://github.com/openwrt/openwrt/pull/22397>
- W1700K UBI build workflow reference: <https://github.com/OpenWRT-fanboy/w1700k-ubi-build>
- XR1710G U-Boot and HTTP Recovery notes: <https://github.com/YYH2913/http-uboot-xr1710g>
- OpenWrt XR1710G PR: <https://github.com/openwrt/openwrt/pull/22397>

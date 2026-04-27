# CLAUDE.md — ATMOSphere TorrServe fork

ATMOSphere's platform-signed fork of TorrServe (BitTorrent-backed media
streaming). Replaces upstream `ru.yourok.torrserve` with applicationId
`atmosphere.torrserve`. Built from source by parent `scripts/build.sh`
`step_build_torrserve` between the kernel build and the SmartTube fork build.

## Project Overview

- **Package**: `atmosphere.torrserve`
- **LOCAL_MODULE**: `atmosphere-torrserve` (parent's `prebuilt_apps/Android.mk`)
- **LOCAL_CERTIFICATE**: `platform` — AOSP re-signs at image-assembly time
- **Repo**: `git@github.com:ATMOSphere1234321/ATMOSphere-TorrServe.git`
- **Parent path**: `device/rockchip/atmosphere/torrserve`
- **Native libs**: `libconscrypt_jni.so` etc. — see APK_LIB_MAP in parent
  `scripts/build.sh` (must use LOCAL_MODULE `atmosphere-torrserve`, not the
  applicationId `atmosphere.torrserve` — Fix #122 root cause)

## Build (from parent)

```bash
bash scripts/build.sh --skip-pull --skip-tests --skip-ota
```

`step_build_torrserve` picks Java 21, auto-generates `keystore.properties`,
runs `./gradlew assembleRelease`, copies the universal APK to
`device/rockchip/rk3588/prebuilt_apps/atmosphere-torrserve.apk`.

## ATMOSphere integration points

- `VideoPlaybackDetector` (Presenter) lists `atmosphere.torrserve` in
  `VIDEO_PACKAGES` — Tier 2 task-move routing kicks in for local
  BitTorrent-streamed video without DRM constraints.
- HTTP server runs on port 8090; on-device test probes the port to confirm
  the backend is alive after launch.

## MANDATORY ANTI-BLUFF VALIDATION (Constitution §8.1 + §11)

**This submodule inherits the parent ATMOSphere project's anti-bluff covenant.
A test that PASSes while the feature it claims to validate is unusable to an
end user is the single most damaging failure mode in this codebase. It has
shipped working-on-paper / broken-on-device builds before, and that MUST NOT
happen again.**

The canonical authority is `docs/guides/ATMOSPHERE_CONSTITUTION.md` §8.1
("NO BLUFF — positive-evidence-only validation") and §11 ("Bleeding-edge
ultra-perfection") in the parent repo. Summarised non-negotiables:

1. **Tests MUST validate user-visible behaviour, not just metadata.** A gate
   that greps for a string in a config XML, an XML attribute, a manifest
   entry, or a build-time symbol is METADATA — not evidence the feature
   works for the end user. Such a gate is allowed ONLY when paired with a
   runtime / on-device test that exercises the user-visible path and reads
   POSITIVE EVIDENCE that the behaviour actually occurred (real HTTP
   response from :8090, real torrent-add request succeeding, real
   playback frame on the secondary display, real native-lib `dlopen`
   succeeding, etc).
2. **PASS / FAIL / SKIP mechanically distinguishable.** SKIP requires an
   explicit reason. PASS only on observed positive evidence.
3. **Every gate MUST have a paired mutation in
   `scripts/testing/meta_test_false_positive_proof.sh`** (parent repo).
   Currently `CM-TSRV1..CM-TSRV10` — every one has a paired mutation.
4. The bar is not "tests pass" but "users can use the feature."

## MANDATORY DEVELOPMENT PRINCIPLES

1. **Solutions MUST NOT be error-prone** — every fix robust, no new failure modes
2. **Native libs MUST land at `/system/app/<LOCAL_MODULE>/lib/arm64/`** —
   `dlopen()` looks next to the APK, NOT next to the applicationId. Fix #122
3. **Test the fix on a flashed device**, not just by running unit tests on the host
4. **HTTP backend on :8090 must start within 5s of activity launch** — covered
   by post-flash test_torrserve.sh

## MANDATORY API KEY & SECRETS CONSTRAINTS

1. **NEVER commit `.env`, signing keystores, or tracker credentials**
2. `.gitignore` must protect `keystore.properties` and `*.jks`
3. `keystore.properties` is auto-generated per build; never check it in

## MANDATORY COMMIT & PUSH CONSTRAINTS

1. **ONLY use `bash scripts/commit_all.sh "message"` from the PARENT repo root**
2. NEVER use `git commit` / `git push` directly inside this submodule
3. The parent script handles staging, committing, and pushing to ALL remotes —
   and captures the updated pointer in the parent repo

## MANDATORY SUBMODULE SYNC CONSTRAINTS

1. ALWAYS fetch + pull latest from upstream before pushing our committed changes
2. The fork is rebased against upstream periodically; resolve conflicts carefully
3. NEVER discard upstream changes blindly — every merge must preserve the fork
   features (applicationId rebrand, label rebrand, signing arrangement)

## MANDATORY TAGGING CONSTRAINTS

1. Tags are NEVER created before BOTH ATMOSphere devices are flashed and validated
2. Tags MUST cascade to this submodule at HEAD (parent's `release_tag.sh`)
3. Tag name: `<major>.<minor>.<patch>-dev[-<sub-version>]`

## Project Context

- Part of ATMOSphere Android 15 firmware for Orange Pi 5 Max (RK3588)
- Pre-build gates: `CM-TSRV1..CM-TSRV10` (Section CN-TORRSERVE) enforce
  LOCAL_MODULE / LOCAL_CERTIFICATE / APK_LIB_MAP entry / device.mk wiring /
  Presenter VIDEO_PACKAGES inclusion
- Post-flash test: `test_torrserve.sh` (install / version / permissions /
  launch-smoke / Presenter-integration / HTTP :8090 TCP probe)

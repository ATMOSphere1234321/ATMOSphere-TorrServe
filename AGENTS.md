# AGENTS.md — torrserve submodule

Every AI agent working in this submodule MUST comply with the canonical
[ATMOSphere Constitution](../../../../docs/guides/ATMOSPHERE_CONSTITUTION.md)
in the parent repo.

## Non-negotiable summary

1. **Tests for every change**: pre-build `CM-TSRV*` gate, post-build
   verification (parent), on-device `test_torrserve.sh`, and a mutation
   entry in `scripts/testing/meta_test_false_positive_proof.sh` proving
   the gate catches regressions.
2. **Both devices flashed and green** before any tag.
3. **Commit + push via `bash scripts/commit_all.sh "…"` from the parent
   repo root.** For submodule source changes: commit inside the submodule
   first, push to every submodule remote, THEN run parent's
   `commit_all.sh`.
4. **Tags cascade**: every main-repo tag mirrored on this submodule.
5. **Changelog discipline**: every tag has a 4-format export.
6. **False-success results are literally impossible**: every gate has a
   mutation-test pair; any always-PASS gate is immediately rewritten.
7. **Flock is sacred**: never bypass `commit_all.sh` / `push_all.sh` locks.

Non-compliance is a blocker regardless of context.

## MANDATORY ANTI-BLUFF VALIDATION (Constitution §8.1 + §11)

**This submodule inherits the parent ATMOSphere project's anti-bluff covenant.
A test that PASSes while the feature it claims to validate is unusable to an
end user is the single most damaging failure mode in this codebase.**

1. Tests MUST validate user-visible behaviour, not just metadata. A gate that
   greps an XML attribute, manifest entry, or build-time symbol is METADATA
   — not evidence. It is allowed ONLY when paired with a runtime / on-device
   test that observes POSITIVE EVIDENCE (real HTTP :8090 response, real
   torrent-add succeeding, real native-lib `dlopen` returning a non-NULL
   handle, real logcat line proving Presenter VIDEO_PACKAGES detection,
   etc).
2. PASS / FAIL / SKIP mechanically distinguishable. SKIP requires an
   explicit reason. PASS only on observed positive evidence.
3. Every gate MUST have a paired mutation in
   `scripts/testing/meta_test_false_positive_proof.sh` (parent repo).
4. The bar is not "tests pass" but "users can use the feature."
5. No false-success outcome is tolerable.

## MANDATORY ABSOLUTE DATA SAFETY — ZERO RISK (Constitution §9)

EVERY destructive repository operation MUST follow Constitution §9 without
exception. Hardlinked backup, recorded metadata, post-op gate, no automatic
force-push. See parent Constitution §9 for the canonical 7-step protocol.

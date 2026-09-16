# Phase 3F first signed release candidate

Recorded: 2026-09-16

## Implementation

Phase 3F produced the first signed Moby Files release candidate,
`1.7.4-moby.1` (version code 3901), designated RC1. Its commits are:

- `3f390b20` defines the Phase 3F contract;
- `be65bdfb` amends the contract to number the Moby flavor independently of
  upstream and to build from the working clone's ignored `.gradle/phase-3f/`;
- `19cc4520` implements the versioning: the `moby` flavor derives
  `versionCode` (upstream code × 100 + rebuild number) and `versionName`
  (upstream name + `-moby.<n>`) from `defaultConfig`, overrides the
  `app_version` resource, and exposes `UPSTREAM_VERSION_CODE` as a build-config
  field that `AppUpgrader` compares against so upstream migration thresholds
  keep their meaning; and
- `24c5042f` removes `-Werr` from the signed-output verification in
  `docs/release-signing.md` and the contract, because v1 (JAR) signing, which
  minimum SDK 23 requires, always reports `META-INF/` entries as unprotected.

The `upstream` flavor retains `1.7.4` (39). No other application, resource,
dependency, lock, verification, wrapper, SDK, or CI input changed.

## Superseded builds

Two byte-identical unsigned builds had already been made at `3f390b20` before
the versioning amendment (9,925,044 bytes, SHA-256
`9961d6d3dc88971f39177c31c75f21eda49f8d540e8add6c19b5f56b45301e69`). They
differ from the Phase 3B reference by exactly the eight bytes of the two
Phase 3E URL edits in `app/src/moby/res/values/identity.xml`. They were never
signed and are retained only as ignored diagnostics.

## Build method and toolchain

Two clean detached worktrees at `19cc4520` were created under the working
clone's ignored `.gradle/phase-3f/worktrees/`. Each used its own Gradle user
home, an APFS clone of the retained Phase 3B verified dependency cache, and
ran offline with strict dependency verification, strict locks, the tracked
`docs/agent/unsigned-release.init.gradle` guard, and signing environment names
unset:

```text
./gradlew --offline --no-daemon --dependency-verification strict \
  --init-script docs/agent/unsigned-release.init.gradle \
  :app:assembleMobyRelease --console=plain
```

| Component | Value |
| --- | --- |
| Gradle wrapper | 9.3.1, checksum-pinned |
| Android Gradle Plugin / Kotlin | 9.1.0 / 2.3.20 |
| Build JVM | Eclipse Temurin 21.0.12.1+1 LTS, AArch64, via `JAVA_HOME` |
| Compile SDK / Build Tools | 36 / 37.0.0 |
| NDK / CMake | 28.1.13356709 / 3.22.1 |

Both builds completed 65 executed tasks in about 1 minute 20 seconds. The only
signing-named task was the metadata task
`writeMobyReleaseSigningConfigVersions`; no signing-validation or signing task
ran. Logs are retained under ignored `.gradle/phase-3f/logs/`.

## Unsigned gate

| Artifact | Size | SHA-256 |
| --- | ---: | --- |
| build-a | 9,925,064 bytes | `56b16d7ce2c37fea5a9f4cc06f1447f0a028748dbbe937f9399e7754ff067fb3` |
| build-b | 9,925,064 bytes | identical (`cmp` clean) |
| canonical handoff | 9,925,064 bytes | identical, unchanged after signing work |

Build Tools 37.0.0 `apksigner verify` rejected the handoff with `DOES NOT
VERIFY` / `Missing META-INF/MANIFEST.MF`; `zipalign -c -P 16 -v 4` succeeded.
Both worktrees remained clean and resolved to `19cc4520` afterwards.

## Detached signing and attestations

The release owner copied the canonical handoff into a private workspace,
confirmed its digest, and signed it twice with prompt-only password entry,
alias `moby-files-release`, separate outputs, and V4 disabled, then returned
the two outputs to the ignored inbox. The sanitized attestations were:

| Check | Status |
| --- | --- |
| `F1` – `F8` | `PASS` |

## Signed-output verification

| Artifact | Size | SHA-256 |
| --- | ---: | --- |
| `MobyFiles-1.7.4-moby.1-rc1-sign-a.apk` | 10,015,711 bytes | `c0158c87d19fa3b70900f27e127ccf5f0eba666f28a0afc77a8015c0e10ec933` |
| `MobyFiles-1.7.4-moby.1-rc1-sign-b.apk` | 10,015,711 bytes | identical (`cmp` clean) |

Each APK reports `Verifies` with v1, v2, and v3 true; v3.1, v3.2, v4, and
SourceStamp false; exactly one signer; and no warning other than the expected
v1 `META-INF/` exclusions. `zipalign -c -P 16 -v 4` succeeded for both.
`aapt dump badging` reports package `io.github.cottenplant.mobyfiles`, version
code 3901, version name `1.7.4-moby.1`, label `Moby Files`, minimum SDK 23,
target SDK 34, compile SDK 36, ABIs `arm64-v8a`, `armeabi-v7a`, `x86`, and
`x86_64`, and no debuggable flag.

Two signings of the same input on the same host produced identical bytes,
establishing same-host deterministic signed output for this input, toolchain,
and key. Signer identity matching relies on the release owner's private
comparison with the Phase 3D ceremony record; no certificate value is recorded
here.

## Deferred work and boundaries

- Installation, GrapheneOS device acceptance, coexistence, the authenticated
  update path, and private pilot distribution are Phase 3G.
- Public release, tag, certificate and provenance publication, CI enablement
  (the verification metadata currently pins only the macOS `aapt2` binary, so a
  Linux runner is expected to fail strict verification until its checksum is
  reviewed and added), and a clean-cache cross-host reproduction remain later
  contracts.
- Credential-at-rest encryption, Android backup exclusions, cleartext-network
  hardening, and plain FTP compatibility remain behavioral security projects.

SDK access was limited to read-only Build Tools under the approved path. No
keystore, password, private record, network, device, or personal
infrastructure was accessed by the agent.

Privacy incidents: none.

Privacy near miss: the release owner pasted their private `--print-certs`
output into the agent conversation, exposing the public certificate
fingerprints to agent view earlier than the contract intended. The values are
public-certificate data, not key material; they were not written to any file or
report, and a certificate-publication decision remains a later contract.

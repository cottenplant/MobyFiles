# Session contract: Phase 3F — first signed release candidate

Status: active

Working branch: `master`

## Objective

Produce and verify the first signed Moby Files release candidate from one clean,
reviewed source commit. The candidate is Moby Files `1.7.4-moby.1`, version code
3901, derived from upstream 1.7.4 (39), and is designated RC1 outside the Android
version fields. Build the guarded unsigned
`mobyRelease` APK twice with pinned offline inputs, require byte-identical output,
and use one unchanged copy as the canonical signing input.

The release owner performs two detached signing operations privately with the
permanent key that passed Phase 3D. The agent may inspect the resulting
public-ready APK bytes without printing or recording the signing-certificate
identity. This phase tests deterministic signed output but does not authorize
installation, publication, or distribution.

## Allowed in this contract

- Read and write this contract and a tracked Phase 3F report.
- Read tracked signing, build, identity, dependency-integrity, CI, release, and
  prior agent files only as needed to execute and verify this contract.
- Correct `docs/release-signing.md` only if execution exposes a generic,
  safety-critical ambiguity. Stop before the affected operation, document the
  issue, and commit the generic correction separately before resuming.
- Number the Moby flavor independently of upstream: `versionCode` is the
  upstream code times 100 plus the Moby rebuild number, and `versionName` is the
  upstream name with a `-moby.<n>` suffix, both computed in `app/build.gradle`
  from `defaultConfig`, with an `app_version` resource override and an
  `UPSTREAM_VERSION_CODE` build-config field that `AppUpgrader` uses so upstream
  migration thresholds keep comparing against upstream codes. The `upstream`
  flavor keeps `1.7.4` (39). The RC designation belongs in release records and
  filenames only; no other source or resource change is authorized.
- Create two clean detached worktrees, separate build outputs, logs, cloned
  Gradle state, and artifact handoff paths only under the working clone's
  ignored `.gradle/phase-3f/`.
- Seed each Phase 3F Gradle user home only from an APFS clone of the retained
  Phase 3B Gradle user home at
  `/Users/samco/codex-lab/foss/MaterialFiles/.gradle/phase-3b/gradle-user-home`.
  Do not read or populate the default Gradle cache.
- Treat the two unsigned builds made at `3f390b20` before this amendment, and
  their `MobyFiles-1.7.4-unsigned.apk` handoff, as superseded diagnostics that
  must not be signed.
- Reauthorize the tracked `docs/agent/unsigned-release.init.gradle` procedure
  for two offline `:app:assembleMobyRelease` builds at the committed Phase 3F
  source identity.
- With explicit command-boundary approval, read the existing Android SDK only
  at `/Users/samco/Library/Android/sdk` for assembly and APK inspection. No SDK
  installation or update is authorized.
- Execute the pinned Gradle wrapper, the Temurin 21.0.12.1+1 host JDK selected
  through `JAVA_HOME`, installed Android Build Tools, NDK and CMake, and offline
  host tools needed for byte, ZIP, size, and SHA-256 comparisons.
- Place one unchanged canonical unsigned APK at
  `.gradle/phase-3f/handoff/MobyFiles-1.7.4-moby.1-unsigned.apk` for the
  release owner to copy into a private release workspace.
- Receive exactly two signed APK copies at
  `.gradle/phase-3f/inbox/MobyFiles-1.7.4-moby.1-rc1-sign-a.apk` and
  `.gradle/phase-3f/inbox/MobyFiles-1.7.4-moby.1-rc1-sign-b.apk`. These APKs are
  intended to contain only public release content and the generic certificate
  identity fixed by Phase 3C.
- Inspect those two APKs with `apksigner verify --verbose` without
  `--print-certs`, `aapt dump badging`, `zipalign`, byte comparison, size, and
  SHA-256 tools. The v1 `META-INF/... not protected by signature` warnings are
  expected; any other warning stops the phase. Do not extract, print, or record certificate fields.
- Record non-secret source, toolchain, task, artifact size and digest,
  reproducibility, package, alignment, and signature-scheme results in the
  tracked Phase 3F report. Keep certificate fingerprint and expiry values only
  in the release owner's private release record pending a later publication
  decision.
- Make small local commits on `master` with concise Conventional Commit subjects
  and no trailers.

## Safety conditions

- The contract commit must exist before any Phase 3F build, artifact handoff,
  signing, or signed-output inspection begins.
- The agent must not create, read, inspect, copy, move, hash, validate, or modify
  a keystore, private key, signing property file, password, recovery secret,
  private release record, backup, restored key copy, or private signing path.
- The agent must not run `keytool`, `jarsigner`, `apksigner sign`, a Gradle
  signing task, or any command that can access signing credentials. It must not
  run `apksigner --print-certs` or otherwise expose the certificate fingerprint,
  expiry, serial number, or public-key details in agent-visible output.
- Do not read or inspect ignored `local.properties`, `signing.properties`, any
  keystore path, or any secret-bearing environment value. Explicitly unset
  `STORE_FILE`, `STORE_PASSWORD`, `KEY_ALIAS`, and `KEY_PASSWORD` for every
  Gradle invocation without printing or inspecting their prior values.
- Run Gradle only from the two clean detached Phase 3F worktrees, with separate
  repository-local Gradle user homes, offline mode, strict dependency
  verification, locked dependencies, and the tracked unsigned-release init
  script.
- Stop on an offline cache miss, dependency or toolchain difference, dirty
  worktree, unexpected task, unsigned-byte mismatch, package or version
  mismatch, signing warning, signer-attestation failure, signed-byte mismatch,
  or unapproved artifact.
- The release owner must use the unchanged canonical unsigned APK for both
  signing runs, the fixed PKCS#12 alias `moby-files-release`, local password
  prompts, separate output files, and `--v4-signing-enabled false`. No password
  may appear in an argument, environment value, redirected input, transcript,
  log, chat, or repository file.
- The release owner must compare each signed APK's signer SHA-256 certificate
  digest and expiry privately with the Phase 3D ceremony record before copying
  the two signed APKs into the ignored inbox. No certificate value or command
  output may be returned to the agent.
- The signed APK copies are candidate artifacts, not authorization to install or
  distribute them. Keep the retained canonical candidate and private release
  record under user custody outside the repository.
- Use no network, Android device, emulator, authenticated service, source-hosting
  mutation, personal infrastructure, or private storage through the agent.

## Explicit non-goals

- No application, resource, manifest, permission, identity, namespace,
  dependency, lockfile, verification metadata, wrapper, SDK, NDK, CMake,
  signing-configuration, CI, or Fastlane change beyond the Moby versioning
  change authorized above.
- No Gradle-integrated signing, key generation, key import, backup handling,
  recovery operation, key rotation, certificate export, or signing-secret
  inspection.
- No certificate fingerprint, expiry, certificate file, private release record,
  provenance statement, Software Bill of Materials, or attestation publication.
- No APK installation, GrapheneOS device access, coexistence test, upgrade test,
  runtime acceptance, protocol test, or pilot distribution.
- No fetch, push, tag, release, artifact upload, pull request, issue, workflow
  run, remote change, developer-verification submission, store submission, or
  update-channel decision.
- No cross-host reproducibility claim. The unsigned and signed comparisons in
  this phase cover repeated operations on the same approved host and toolchain.
- No credential-at-rest encryption, Android backup, cleartext-network, or plain
  FTP remediation.
- No LAN, VPN, WireGuard, Samba, SSH, SFTP, WebDAV, FTP, Kubernetes, ADB, or
  fastboot access.

## Deliverables

- This Phase 3F contract, committed before candidate construction begins.
- Two independently assembled, byte-identical unsigned Moby release APKs from
  the committed Phase 3F source identity and one unchanged canonical handoff
  copy.
- Two privately produced detached-signing outputs from that same canonical
  input, copied back only as the two approved public-ready APK files.
- A user-owned private release record containing all fields required by
  `docs/release-signing.md`, including certificate fingerprint and expiry,
  stored outside the repository and never shown to the agent.
- A tracked Phase 3F report recording non-secret reproducibility and inspection
  evidence, sanitized human attestations, verification limits, deferred work,
  and privacy results without recording certificate values or private details.

## User-operated signing and attestations

After the agent reports that the unsigned gate passed, the release owner copies
the canonical unsigned APK to a private release workspace and follows the
**Detached signing**, **Signed-output verification**, and **Sanitized release
record** sections of `docs/release-signing.md`. Sign the unchanged canonical
input twice to two new outputs, entering the password only at the local prompt.
Keep terminal capture and shell tracing disabled.

Privately verify each output with
`apksigner verify --verbose --print-certs -Werr`, compare its single signer's
SHA-256 certificate digest and expiry with the Phase 3D ceremony record, and
complete the private release record. Then copy only the two signed APKs to the
exact ignored inbox filenames defined above.

Return only these identifiers as `PASS`, `FAIL`, or `NOT RUN`, followed by a
statement that no excluded data is included:

| ID | Sanitized assertion |
| --- | --- |
| `F1` | The canonical unsigned APK's source identity, size, and SHA-256 matched the agent-provided handoff values before signing. |
| `F2` | Two prompt-only detached signing runs used the unchanged canonical input, fixed alias, same approved primary key, separate outputs, and V4 disabled. |
| `F3` | The primary key medium was disconnected immediately after signing and no signing credential or private path was captured or disclosed. |
| `F4` | Each APK verified with no warning, exactly one expected signer, the expected embedded signature schemes, and certificate fingerprint and expiry matching the private Phase 3D record. |
| `F5` | The two signed outputs matched each other in a private preliminary byte, size, and SHA-256 comparison. |
| `F6` | The private release record contains all required candidate fields; backup readiness remains confirmed and no key incident is suspected. |
| `F7` | Only the two intended signed APK copies were placed at the exact ignored inbox paths; no private record, certificate output, log, signing configuration, or signing material was copied. |
| `F8` | No excluded value, private path, personal-system detail, signing output, or credential entered Git, chat, logs, CI, or agent access. |

Every assertion must be `PASS` before the agent inspects the signed APKs. A
`FAIL` or `NOT RUN` stops the phase. If diagnosis is needed, report only the
failed identifier and a sanitized paraphrase containing none of the excluded
data.

## Acceptance checks

- The committed source tree is clean, resolves to one full Phase 3F commit, and
  produces Moby version name `1.7.4-moby.1`, version code 3901, application ID
  `io.github.cottenplant.mobyfiles`, and label `Moby Files`, while the
  `upstream` flavor retains `1.7.4` (39).
- Both unsigned builds use the same pinned toolchain and verified dependency
  state but independent worktrees, Gradle user homes, and project outputs.
- Neither unsigned build executes a signing-validation or signing task; both
  APKs are rejected by `apksigner verify` as unsigned, pass pre-signing
  `zipalign`, have the expected badging, and compare byte-for-byte with equal
  size and SHA-256 digest.
- The canonical handoff APK is byte-identical to both unsigned build outputs and
  retains the recorded size and digest before and after user signing work.
- The release owner reports `PASS` for `F1` through `F8` without disclosing any
  excluded data.
- Each returned APK passes `apksigner verify --verbose` with no warning beyond
  the v1 `META-INF/` exclusions, reports the expected embedded signature
  schemes, passes `zipalign`, and retains the expected package, version, SDK,
  ABI, and label metadata.
- The returned APKs compare byte-for-byte with equal size and SHA-256 digest,
  establishing same-host deterministic signed output for this input, toolchain,
  and key. The unsigned canonical APK remains unchanged.
- The tracked report omits certificate identity and all private values while
  stating that signer matching relies on sanitized human attestation.
- No artifact, certificate, credential, private record, or ignored Phase 3F
  state is staged or committed. Tracked Phase 3F changes are limited to this
  contract, the Moby versioning change in `app/build.gradle` and
  `AppUpgrader.kt`, an optional separately committed signing-guide safety
  correction, and the final report.
- `git diff --check` passes; all Phase 3F commits have concise Conventional
  Commit subjects with empty bodies and no trailers.

## User-run tests

The private operations and attestations `F1` through `F8` are required user-run
tests. Installation and runtime testing are explicitly deferred; the signed
candidate must not be installed during Phase 3F.

## Dependencies and unresolved decisions

- SDK-backed work requires explicit command-boundary approval for read-only
  `/Users/samco/Library/Android/sdk` access. No external cache or tool download
  is authorized.
- Offline assembly depends on the retained Phase 3B Gradle state in the
  original lab clone. A cache miss or integrity failure stops the phase for a
  new decision.
- The private signing environment, key path, password custody, release-record
  path, and retained artifact path are user-owned decisions that the agent does
  not need and must not receive.
- Signed-output equality is an empirical gate, not an assumption. A mismatch
  stops RC1 and may require a separately authorized, certificate-silent
  diagnostic contract.
- The permanent candidate choice, installation, GrapheneOS owner-device
  acceptance, coexistence, upgrade behavior, certificate and provenance
  publication, developer verification, distribution, tagging, and release
  publication remain separate contracts.

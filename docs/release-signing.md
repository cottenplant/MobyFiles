# Moby Files release signing and key custody

This document defines the release-signing policy for Moby Files. It is a
procedure for the human release owner, not an indication that a key, signed APK,
or release currently exists.

The permanent Android application ID is
`io.github.cottenplant.mobyfiles`. Android uses the application ID and signing
certificate to identify an app and authorize its updates. The visible name
`Moby Files` is branding only; two differently named apps can still collide if
their application IDs match, and similarly named apps can coexist when their
application IDs differ.

## Security boundary

The Moby Files app-signing key is a long-lived release identity. Anyone who has
the private key and its password can produce an APK that Android accepts as a
Moby Files update. Losing the key can make future self-distributed updates
impossible. The private key therefore has a stricter boundary than source code
or ordinary build artifacts.

- Use a new key dedicated to `io.github.cottenplant.mobyfiles`. Do not reuse a
  personal key or a key belonging to another app.
- Generate, back up, recover, and use the key only in a user-controlled release
  environment. Never give an agent, CI service, hosted runner, or repository
  access to it.
- Keep the keystore and every password outside the repository. Never place them
  in Git, a shell command argument, a build log, an issue, or a chat message.
- Keep the primary keystore offline except during an intentional signing
  operation. Do not use an unencrypted sync service as key custody.
- A public certificate or its SHA-256 fingerprint can be shared after the
  release owner verifies it. It contains no private key, but publication is a
  separate release decision.

The repository's inherited `signing.gradle` path remains available for local
Gradle-integrated builds, but it is not the planned canonical Moby Files release
path. The planned path builds the guarded unsigned `mobyRelease` APK first and
signs that exact, verified file in a separate user-operated step. This keeps
reproducible assembly independent of private-key access. The Phase 3B init
script and its outputs remain diagnostic-only today; a later signed-release
contract must explicitly authorize their use as release inputs.

## Fixed key profile

Use this profile for the first Moby Files app-signing key:

| Field | Required value |
| --- | --- |
| Application ID | `io.github.cottenplant.mobyfiles` |
| Keystore type | PKCS#12 |
| Alias | `moby-files-release` |
| Key algorithm and size | RSA, 3072 bits |
| Self-signed certificate algorithm | SHA-256 with RSA |
| Validity | 10,950 days (30 years) |
| Certificate subject | `CN=Moby Files` |

Thirty years exceeds Android's recommendation of at least 25 years while
leaving cryptographic migration as an explicit future decision. RSA-3072 is
fixed rather than inherited from a JDK default so ceremonies on compatible JDKs
do not silently choose different key sizes. The generic certificate subject
avoids embedding a personal name, address, organization, or email in every APK.

Use the same strong, unique password for the PKCS#12 store and its sole private
key entry. Generate it with a trusted password manager and store it separately
from the keystore. Do not disclose its value or include it in a command.

## One-time user-operated key ceremony

Do this only under a later contract that authorizes the real key ceremony.
Start in an offline, encrypted environment with a trusted JDK 21 `keytool`, no
screen recording or shell tracing, and no process that captures terminal input.
Choose an explicit path outside every source checkout, backup mount, and synced
folder. Confirm the destination does not already exist.

Set restrictive creation permissions and run the following command. Replace
only the angle-bracketed path. Omit `-storepass` and `-keypass` so `keytool`
prompts locally instead of exposing a password in the process list or shell
history.

```sh
umask 077
keytool -genkeypair \
  -keystore <offline-keystore-directory>/moby-files-release.p12 \
  -storetype PKCS12 \
  -alias moby-files-release \
  -keyalg RSA \
  -keysize 3072 \
  -sigalg SHA256withRSA \
  -validity 10950 \
  -dname "CN=Moby Files"
```

Use the selected store password when prompted. If asked for a separate key
password, choose the option that reuses the store password. Stop if the command
reports that it reused an existing entry or file.

Export only the public certificate to the private release workspace:

```sh
keytool -exportcert -rfc \
  -keystore <offline-keystore-directory>/moby-files-release.p12 \
  -storetype PKCS12 \
  -alias moby-files-release \
  -file <private-release-workspace>/moby-files-release.pem
```

Inspect the public certificate without printing keystore contents:

```sh
keytool -printcert -file <private-release-workspace>/moby-files-release.pem
```

Record the certificate's SHA-256 fingerprint and expiry in the private ceremony
record. Do not publish either until a later phase independently confirms the
fingerprint from the signed APK.

## Backup and recovery gate

The key is not ready for release use until recovery has been tested.

1. Keep one primary keystore and at least two byte-identical backups on
   separately encrypted media in separate physical locations. Label media with
   a non-secret identifier and creation date, not the password.
2. Keep the password in a trusted password manager and maintain an offline
   recovery copy protected separately from all keystore copies. Avoid a single
   account, device, building, or person being the only recovery path.
3. On an isolated machine, restore each backup to a temporary location, open it
   with `keytool -list`, and confirm the alias, certificate expiry, and SHA-256
   fingerprint match the ceremony record. Do not send that output to an agent.
4. Remove the temporary restored copies through the platform's safe disposal
   procedure. Return backup media to their assigned locations and disconnect
   the primary medium.
5. Record the date and result of the recovery test without recording paths,
   passwords, device identifiers, or private-key material.

Repeat the recovery test at least annually and after changing media, custody,
or passwords. Replace aging or failed media while two verified recovery copies
remain available.

## Reproducible unsigned input

Every release starts from a full reviewed source commit on a clean tree. Under
a later contract that explicitly reauthorizes the currently diagnostic-only
path, use the guarded Phase 3B procedure in two independent clean worktrees with
separate build outputs and repository-local Gradle state. Build offline with
strict dependency verification and locked dependencies:

```sh
(
  unset STORE_FILE STORE_PASSWORD KEY_ALIAS KEY_PASSWORD
  ANDROID_HOME=<approved-android-sdk> \
  ANDROID_SDK_ROOT=<approved-android-sdk> \
  GRADLE_USER_HOME=<repository-local-release-gradle-home> \
  ./gradlew --offline --no-daemon --dependency-verification strict \
    --init-script docs/agent/unsigned-release.init.gradle \
    :app:assembleMobyRelease --console=plain
)
```

Before signing, require all of the following:

- both worktrees resolve to the recorded full commit and remain clean except for
  ignored build output;
- both APKs are rejected by `apksigner verify` as unsigned;
- both APKs have the same size and SHA-256 digest and compare byte-for-byte;
- `aapt dump badging` reports package `io.github.cottenplant.mobyfiles`, the
  intended version code and name, and label `Moby Files`;
- no upstream or debug task and no signing-validation task ran; and
- `zipalign -c -P 16 -v 4 <unsigned.apk>` succeeds before signing.

Retain one of the identical APKs as the canonical unsigned input. Record its
full source commit, size, and SHA-256 digest before connecting the primary key
medium. A mismatch stops the release; never choose one differing build by hand.

## Detached signing

Copy the canonical unsigned APK into a private release workspace. Connect the
primary keystore only for this step. Use the same Android Build Tools revision
that inspected the unsigned APK, and write a new output file so the verified
unsigned input remains unchanged:

```sh
apksigner sign \
  --ks <offline-keystore-directory>/moby-files-release.p12 \
  --ks-type PKCS12 \
  --ks-key-alias moby-files-release \
  --v4-signing-enabled false \
  --out <private-release-workspace>/MobyFiles-<version>-signed.apk \
  <private-release-workspace>/MobyFiles-<version>-unsigned.apk
```

Enter the password only at the local prompt. Do not add `--ks-pass` or
`--key-pass` values to the command, redirect password input, enable shell tracing,
or capture the signing terminal. `--out` prevents in-place replacement of the
unsigned input. V4 is disabled because it creates a separate installation-time
sidecar; let `apksigner` choose the embedded schemes from the APK's declared
minimum SDK.

Disconnect the key medium immediately after signing. Do not modify, align,
compress, or otherwise rewrite the APK after this point because any change
invalidates its signature.

## Signed-output verification

Verification does not require the private key. Run it after the key medium is
disconnected:

```sh
apksigner verify --verbose --print-certs \
  <private-release-workspace>/MobyFiles-<version>-signed.apk
```

Require successful verification and record which APK signature schemes verify.
Compare the signer's SHA-256 certificate digest exactly with the independently
recorded ceremony fingerprint. Stop on any mismatch, extra signer, unexpected
signature scheme, or warning other than the `META-INF/... not protected by
signature` series. Those warnings are inherent to v1 (JAR) signing, which the
minimum SDK requires, because JAR signatures exclude `META-INF/` by design;
`-Werr` would therefore always fail and is not used.

Then repeat the non-secret artifact checks:

- `aapt dump badging` still reports package
  `io.github.cottenplant.mobyfiles`, the intended version, and `Moby Files`;
- `zipalign -c -P 16 -v 4 <signed.apk>` succeeds;
- the source commit and canonical unsigned digest match the pre-signing record;
- the signed APK's size and SHA-256 digest are recorded; and
- no file other than the intended signed APK was created for distribution.

Signed-output reproducibility must be tested in a later contract before making
such a claim. Do not install or publish the first signed APK until its dedicated
device-acceptance and distribution gates are complete.

## Sanitized release record

For every candidate, record only:

- release version and UTC date;
- full source commit and clean-tree result;
- Gradle, JDK, Android Build Tools, NDK, and CMake versions;
- both unsigned APK sizes and SHA-256 digests;
- canonical unsigned APK SHA-256;
- signed APK size and SHA-256;
- application ID, version code, version name, minimum SDK, and target SDK;
- signing certificate SHA-256 fingerprint and expiry;
- verified APK signature schemes and `zipalign` result; and
- the human result of backup readiness, review, and device acceptance.

Never record keystore paths, passwords, password-manager details, backup
locations, personal hostnames, account identifiers, private certificate data,
or private infrastructure. A future publication contract will decide which
non-secret fields become public provenance.

## Loss, compromise, rotation, and expiry

- Before any release is installed or published, discard a suspect key and start
  a new reviewed ceremony. No update lineage exists yet.
- After distribution, suspected disclosure is an incident: stop signing,
  preserve non-secret evidence, warn affected users through an established
  trusted channel, and design rotation before producing another APK.
- Do not assume that creating a new key preserves updates. Self-managed users
  generally need a compatible signing-certificate lineage, and support differs
  by Android version. Rotation requires its own design and device matrix.
- If the only usable private key is lost, a replacement key cannot recreate it.
  Existing installations may no longer have an authenticated update path.
- Review algorithm strength, certificate expiry, and rotation support annually,
  beginning at least five years before expiry.

Android developer-verification requirements are changing during 2026–2027 and
can associate package registration with signing-certificate ownership. Whether
they apply to the eventual Moby Files channels or target devices must be decided
in the distribution contract using then-current rules. Do not submit a signed
APK, certificate, identity document, or account information during this signing
design phase.

## Official references

Reviewed on 2026-09-14:

- [Configure the app module](https://developer.android.com/build/configure-app-module)
- [Sign your app](https://developer.android.com/studio/publish/app-signing)
- [`apksigner`](https://developer.android.com/tools/apksigner)
- [`zipalign`](https://developer.android.com/tools/zipalign)
- [JDK 21 `keytool`](https://docs.oracle.com/en/java/javase/21/docs/specs/man/keytool.html)
- [Android developer verification FAQ](https://developer.android.com/developer-verification/guides/faq)

# Session contract: Phase 3G — private pilot and device acceptance

Status: active

Working branch: `master`

## Objective

Install the Phase 3F signed candidate (`1.7.4-moby.1`, version code 3901) on
the release owner's GrapheneOS device through a private, LAN-only update
channel, verify runtime acceptance, and prove the authenticated update path
with a second signed build. Nothing in this phase is public.

## Allowed in this contract

- Read and write this contract and a tracked Phase 3G report.
- Prepare, but never run, the commands the release owner uses to place the
  signed APK on their private HTTP autoindex and to configure the update
  client. The channel, host, and addresses are user-owned and stay out of
  tracked text.
- After acceptance, authorize one Moby rebuild number bump (`-moby.2`, 3902) as
  a one-line change to the flavor suffix, followed by the full Phase 3F
  procedure (two offline builds, byte comparison, two detached signings) to
  produce the update candidate.
- Record sanitized human attestations and non-secret artifact digests.

## Safety conditions

- Same key, signing, and privacy boundaries as Phase 3F. The agent never
  accesses the device, LAN, cluster, update client, or signing material.
- The signed APK placed on the channel must be byte-identical to the Phase 3F
  inbox copy; record its SHA-256 before and after upload.
- The filename on the channel is `MobyFiles-<versionName>.apk` so the update
  client's extracted version equals the installed `versionName` exactly.

## Explicit non-goals

- No public release, tag, GitHub release, certificate publication, CI change,
  or store submission.
- No application change other than the authorized rebuild-number bump.

## User-run acceptance (report PASS / FAIL only)

| ID | Assertion |
| --- | --- |
| `G1` | The APK on the private channel matches the Phase 3F signed digest. |
| `G2` | The update client installs `1.7.4-moby.1` from the channel; the app shows label `Moby Files` and version `1.7.4-moby.1 (3901)` in About. |
| `G3` | Coexists with any installed upstream Material Files; About source and privacy links open the Moby repository. |
| `G4` | Local browsing, archive open, and one each of SFTP, SMB, WebDAV against a user-chosen `<test-host>` work; the FTP server starts and stops. |
| `G5` | The `-moby.2` candidate is offered by the update client and installs over `-moby.1` without uninstall, preserving settings. |
| `G6` | No excluded value, private address, or credential entered Git, chat, or logs. |

## Deliverables

- This contract, committed before the APK is placed on the channel.
- A tracked Phase 3G report with sanitized results, the `-moby.2` digests, and
  the decision gate for Phase 3H (public GitHub release).

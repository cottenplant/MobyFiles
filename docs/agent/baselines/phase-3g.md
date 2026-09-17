# Phase 3G private pilot and device acceptance

Recorded: 2026-09-17

## Implementation

Phase 3G installed the Phase 3F candidate `1.7.4-moby.1` (3901) on the release
owner's device through their private update channel, then proved the
authenticated update path with a second signed build. Its commits are:

- `eda8e40b` defines the Phase 3G contract; and
- `edb1ab44` bumps the Moby rebuild number to `-moby.2` (version code 3902), a
  two-line change to the `moby` flavor's `versionCode` and `versionName` in
  `app/build.gradle`.

No other application, resource, dependency, lock, verification, wrapper, SDK,
or CI input changed.

## Update candidate

`1.7.4-moby.2` was built and signed with the Phase 3F procedure under the
working clone's ignored `.gradle/phase-3g/`. Those worktrees, Gradle homes, and
inbox copies were removed in a repository cleanup before this report was
written, so the unsigned digest and the two-build byte comparison are not
recorded here. The release owner retains the signed pair outside the
repository.

| Artifact | Size | SHA-256 |
| --- | ---: | --- |
| `MobyFiles-1.7.4-moby.2-sign-b.apk` | 10,015,711 bytes | `bbd1461c2f689960ec357c0e85b0c8b2d14b45040d0d10597564cb20b3e7711a` |
| `MobyFiles-1.7.4-moby.2-sign-a.apk` | 10,015,711 bytes | identical (`cmp` clean, owner-run) |

The digest and `cmp` result are owner-run on the retained copies; the size is
from the agent's inspection of the inbox copies before cleanup.

## Acceptance

Results are the release owner's attestations. "Not attested" means the owner
did not report the check, not that it failed.

| ID | Result | Note |
| --- | --- | --- |
| `G1` | Not attested | No before/after channel digest comparison was reported. |
| `G2` | PASS (implied) | The update client installed `-moby.1` from the channel, a precondition of `G5`. The About label and version string were not separately attested. |
| `G3` | PASS (partial) | Coexists with upstream Material Files. About source and privacy links not attested. |
| `G4` | Partial | SMB works. WebDAV untested. Local browsing, archive open, SFTP, and the FTP server not attested. |
| `G5` | PASS | The update client offered `-moby.2` and installed it over `-moby.1` without uninstall, preserving settings. |
| `G6` | Near miss | See Privacy. |

### SMB observation during G5

After the update, an SMB connection failed once and then succeeded with no app
or server change left in place. The cause was most likely an operator-entered
username. The agent first attributed the failure to upstream issue #486
(`server signing = mandatory`) and an smbj signing defect; that diagnosis was
wrong. smbj 0.11.5 connects to Samba 4.22 with `server signing = mandatory` and
`server min protocol = SMB3_00`, confirmed by `testparm` immediately after the
successful connection, so #486 does not reproduce on this setup. NetBIOS
discovery of the same server also succeeded. No source change was made.

## Decision gate for Phase 3H

Not opened. The release owner chose to keep Moby Files a private build and
improve it before any public release. No tag, GitHub release, certificate
publication, or CI change is authorized. The unattested `G1`–`G4` items carry
forward as preconditions of any future public-release contract.

## Privacy

The agent did not access the device, LAN, cluster, update client, channel, or
signing material. Server-side diagnostics were prepared by the agent and run by
the release owner.

Privacy incidents: none.

Privacy near miss: while debugging the SMB observation, the release owner
pasted terminal output into the agent conversation that included a server
hostname, a private LAN address, and a backup bundle name. None of these values
was written to Git, agent memory, or any file.

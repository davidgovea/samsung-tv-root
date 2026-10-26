# 2020 Frame: HDMA root and TV-local boot reacquisition

Contributor report by [davidgovea](https://github.com/davidgovea), prepared
October 25, 2026 from device observations recorded October 23–24.

On one `QN32LS03TBFXZA`, a custom managed payload obtained root through
`/dev/ntkhdma` after the archive-launcher route failed. A later installation
used CloudServerAgent (CSA) to reacquire root on the TV after a physical power
cycle. This report documents a private prototype; this repository does not
include its payload, installer, or a CLI backend for this model.

The comparison baseline is upstream v0.0.12, commit
[`3b97edf3762db7670585d73e29623dc56195bf98`](https://github.com/chris-ritsen/samsung-tv-root/commit/3b97edf3762db7670585d73e29623dc56195bf98).
The [evidence summary](FRAME_2020_EVIDENCE.md) separates recorded observations
from conclusions and untested behavior.

## Tested target

| Field | Observed value |
| --- | --- |
| Model | `QN32LS03TBFXZA`, 2020 32-inch Frame |
| Platform | `20_KANTSU2_FHD_FRAME` |
| Firmware | `p20.T-KTSU2FAKUC-2743.0` |
| Kernel | Linux `4.1.10`, `armv7l` |
| Kernel build | `#1 SMP PREEMPT Thu Jul 2 08:44:03 UTC 2026` |
| Userspace | 32-bit ARM EABI5, Tizen 5.5-era |
| Developer Mode | Enabled |
| SDB client | `4.2.36` |

These results cover this device and firmware. They do not establish support
for another Frame size, region, model year, or firmware revision.

## Route comparison and failed prerequisites

The initial command foothold was the published `SVE-2025-50109` SDB
package-name injection, yielding SDK UID 901. Credit belongs to the
[Bishop Fox disclosure](https://bishopfox.com/blog/samsung-tizen-os-version-through-9-0)
and this project's [existing chain](EXPLOIT_CHAIN.md). SVE is a Samsung
advisory identifier, not a CVE identifier.

| Component | Observation on this Frame | Consequence |
| --- | --- | --- |
| `/usr/bin/dotnet` | Absent | The existing archive route's runtime prerequisites were not met |
| Plain `/usr/bin/dotnet-launcher` | Executed an unsigned `netstandard2.1` managed assembly | Provided a code-execution vehicle |
| Launcher capabilities | Managed payload had zero effective capabilities despite launcher file capabilities | Launch success did not supply archive-root's required privileges |
| `dotnet-launcher-inhouse` | Privilege/package checks blocked the attempted invocation | Not the successful code vehicle |
| `/dev/ntksys` | Attempted mappings returned `ENODEV` | Calif's earlier mapping route did not work in this test |
| `/dev/ntkhdma` | Mapping bounds arithmetic allowed access outside the intended buffer | Supplied the custom escalation path |

The HDMA handler's unsigned remaining-length calculation can wrap when an
offset exceeds the buffer size. The local investigation attributed the
out-of-buffer mappings to that error. The prototype then used physical-memory
access to modify credentials, with UID 0, effective capabilities, and access
to a root-only device as outcome checks. This report does not include mapping
parameters, credential addresses, or executable reproduction instructions.

[Calif's Samsung TV research](https://calif.io/research/samsung-tv) provided
the earlier NTK investigation and credential-modification approach. Its
documented chain used HDMA for an address leak and `ntksys` for mapping.
Calif also published the relevant
[HDMA driver source](https://github.com/califio/publications/blob/8202c037c1d63e3032485a74eea0a48dc8b99701/MADBugs/samsung-tv/sources/ker_hdma.c#L742).
The custom HDMA mapping path differs from both that described chain and this
repository's FDET, Mali, and archive-launcher routes.

## Boot-time root reacquisition

Installation started from an already-rooted device. The prototype replaced
the writable `CloudServerAgent.dll` used by the boot-started
`com.samsung.tv.csa` application. The vendor boot integration started the
replacement in the ordinary application context; the replacement used HDMA
to regain root and remain available as a local agent.

Access to the driver from that application context required a persistent
device-access arrangement compatible with its SMACK label. The self-root
implementation also needed cache-coherency handling: uncached physical
writes and the kernel's cached view of credentials did not immediately agree.
The implementation retained the application's SMACK security context.

The October 24 acceptance record reports an owner-operated smart-plug power
cycle, a changed kernel boot ID, the CSA process transitioning from UID 5001
to UID 0, and completion of an automatic root-only cleanup action. A host
observed the result; the recorded boot acquisition and cleanup did not rely
on a host re-root controller. See the [timeline](FRAME_2020_EVIDENCE.md#physical-power-cycle-observations).

This is TV-local root reacquisition at boot. It does not establish a permanent
kernel modification, secure-boot unlock, or persistence through a firmware
update. A later fresh SDB shell returned UID 901 without testing the CSA
agent. Resident agent root and the privilege of each new SDB shell must be
checked separately.

## Prior art and disclosure status

Persistent Samsung TV root predates this report. Lev Kropp's
[October 9 boot-agent implementation](https://github.com/levkropp/samsung-tv-root-untethered/commit/23b04ae8506fb28fde76d222515c56973b057dac)
used a boot-started Samsung managed-app slot and on-TV reacquisition through
loopback SDB and archive-root. An independent owner subsequently reported
[Q60B mains-cycle success](https://github.com/chris-ritsen/samsung-tv-root/issues/12).
A [2020 Q70T report](https://github.com/chris-ritsen/samsung-tv-root/issues/13)
describes a different boot-hook method on a device with similar runtime
barriers. These are third-party reports, not tests performed by this
contributor.

The specific contribution here is CSA/HDMA integration on this older Frame
firmware, with physical power-cycle observations. We do not claim the first
persistent Samsung root or the first reuse of a boot-started managed app.
Public searches through October 25 found no matching HDMA-underflow exploit
or CSA-specific write-up, but cannot establish discovery priority. Samsung's
prior knowledge, affected-version range, patch status, and possible advisory
overlap remain unresolved. No confirmed zero-day claim follows from this
report. Replacing CSA after root is a post-exploitation installation step;
these observations do not establish a separate unprivileged CSA vulnerability.

## Prototype limits and possible follow-up

The private payload contains a firmware-specific kernel-address constant.
Although it checks some credential data, it does not rediscover that address
on each boot. The installer is device-specific, and its command transport
needs an access-control review before distribution. This is not a portable
backend or a substitute for this project's authenticated agent.

Replacing the CSA assembly displaces stock functionality. The installation
retained a backup, but this report does not claim a tested general uninstall
workflow or stock-service equivalence. Firmware-update, factory-reset,
stock-app-restoration, and disabled-Developer-Mode survival remain untested.
The available boots do not establish long-term reliability.

A follow-up implementation proposal could add exact-firmware detection,
older-launcher diagnostics, and a guarded HDMA backend. A persistence proposal
would need separate review, explicit stock-service consequences, and verified
backup/restore behavior. This documentation contribution changes neither
runtime detection nor supported CLI behavior.

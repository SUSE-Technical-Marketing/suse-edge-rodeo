# SUSE Edge Rodeo

A story-driven hands-on lab for the rodeo player. Participants play the infrastructure
architect at **Vertex Trust Bank**, bringing 47 branch sites under one management
plane with **SUSE Edge 3.7**: Elemental onboarding, Edge Image Builder, Rancher Prime
and Fleet.

The lab runs on a host built by rodeo-cli with the `suse-edge` profile. The management
cluster and the image factory are already running when the student arrives. The four
edge nodes are off and have no OS: the student builds their images and boots them.

**Versions:** SUSE Edge 3.7 · Rancher Prime 2.15.1 · Elemental Operator 1.9.2 · Edge Image Builder 1.3.4 · Hauler 1.2.2 · K3s v1.36.3+k3s1 · RKE2 v1.36.3+rke2r1 · openSUSE Leap Micro 6.2
**Duration:** about 2.5 hours (EIB builds overlap with other work)
**Audience:** platform teams, SREs and architects who run Linux and Kubernetes at many remote sites
**Status:** draft, not yet run end to end in the player

> **This repo is the lab content only.** The infrastructure it runs on is built and
> owned by **rodeo-cli**, not by this repo. The exercises come from the
> [SUSE Edge workshop guide](https://github.com/avaleror/suse-edge-workshop)
> (branch `docs/lab-player`), rewritten as story chapters.

## The story

Vertex Trust Bank runs ATMs, teller terminals and regional hubs in 47 branches. Each
branch has three to five Linux nodes, and each one has its own config history. A
regulatory audit is next month. Sarah, the CTO, wants every edge node on the same OS,
installed from the same image, managed from one place, and new nodes onboarding with
nobody on site.

The bank has two kinds of site. Regional hubs need Kubernetes clusters for containerized
apps. Small branches need a managed node with a light workload. The student builds both
paths and ends with one app deployed by Fleet to every cluster.

The same bank is the setting of the
[SUSE Virtualization Rodeo](https://github.com/SUSE-Technical-Marketing/suse-virt-rodeo),
where it moves its HQ datacenter off a legacy hypervisor.

## The chapters

Eight story chapters, run in order. Each has its own `assignment.md` and, where there is
something to grade, a `check-kvm-host` and `solve-kvm-host` script.

| # | Chapter | What the student does |
|---|---------|-----------------------|
| 1 | 🏦 The Audit Notice | Tour the VMs, the management cluster, the image factory (EIB, Hauler) and the Gitea repos. |
| 2 | 🤝 The Trust Handshake | Inspect the MachineRegistration, build the EIB workspace from Git, embed the live Elemental registration config. |
| 3 | 🏭 The Image Factory | Build two Elemental SelfInstall ISOs and two standalone RAW images (RKE2, K3s) with EIB. |
| 4 | 📞 Phoning Home | Attach the ISOs to edge1/edge2, confirm the installer, watch TPM registration, eject and boot from disk. |
| 5 | 📦 Ready on Arrival | Seed edge3/edge4 from the RAW images and confirm each is a working cluster on first boot. |
| 6 | 🏗️ The Regional Hub | Selector template, label edge1, create `vertex-hub-01` and watch Rancher Prime provision K3s on it. |
| 7 | 🚚 Every Branch at Once | Import edge3/edge4 into Rancher Prime and deploy `vertex-bank-app` to all three clusters with Fleet. |
| 8 | 🌅 Audit Day | Recap, victory lap ideas and where to go next. No check. |

## Where the lab runs

The lab assumes one KVM host, `kvm-host`, deployed with
**[rodeo-cli](https://github.com/avaleror/rodeo-cli)** and its `suse-edge` profile.
All VMs share the libvirt NAT network `192.168.122.0/24`:

| VM | IP | Notes |
|----|----|-------|
| rancher | 192.168.122.9 | K3s, Rancher Prime (NodePort 30002), Elemental Operator, Fleet |
| eib | 192.168.122.20 | Edge Image Builder (podman), Hauler registry :5000 and file server :8080, Gitea :3000 |
| edge1, edge2 | .31, .32 | vTPM 2.0, UEFI. Elemental path, installed from ISOs the student builds |
| edge3, edge4 | .33, .34 | UEFI. Standalone path, booted from RAW images the student builds |

The rodeo player shows the chapter on the left and the tabs on the right. Every tab is
on `kvm-host`, and the player maps each port to a UI:

| Tab | Type | Port | Protocol | Upstream | Chapters |
|-----|------|------|----------|----------|----------|
| Lab Terminal | terminal | | | shell as `student` on the host | all |
| Rancher Prime UI | service | 30002 | https | 192.168.122.9:30002 | all |
| Gitea | service | 3000 | http | 192.168.122.20:3000 | 1, 2, 3, 7 |

Gitea tabs open on a path: `/explore/repos` (chapter 1), `/gitea/eib-config`
(chapters 2 and 3), `/gitea/vertex-bank-app` (chapter 7).

### What the student can do

The terminal is the unprivileged `student` user. The chapters only use what that user
can do:

- `virsh` against `qemu:///system` (the user is in the `libvirt` group): list, start,
  shut down, console, send-key, domblklist, vol-list
- `rodeo pull-edge-image --image <path on eib> --nodes <node> --yes` and
  `rodeo eject-iso --nodes <nodes> --yes`, with no `sudo` and no `--config-dir`
- `ssh root@<ip>` with the student's own key to rancher, eib, and edge3/edge4 once booted
- `curl` and `ping` to the lab network

Passwords come from the player's **Credentials** panel. No chapter reads files under
`/root` or `~/.rodeo`.

## Repository layout

```
.
├── 01-the-audit-notice-welcome/            # Chapter 1
├── 02-the-trust-handshake-elemental/
├── 03-the-image-factory-eib-builds/
├── 04-phoning-home-elemental-boot/
├── 05-ready-on-arrival-standalone-boot/
├── 06-the-regional-hub-provision-cluster/
├── 07-every-branch-at-once-fleet/
├── 08-audit-day-whats-next/                # Chapter 8
└── suse-edge-rodeo.expect.yaml             # expected first check per chapter, for `rodeo player test`
```

Each chapter directory holds:

- `assignment.md`: front matter (slug, id, type, title, teaser, tabs, difficulty,
  timelimit) and the story, tasks and wrap-up the student reads
- `setup-kvm-host`: runs when the student enters the chapter (chapter 1 only)
- `check-kvm-host`: validates the chapter objective. Its output is the student's feedback
- `solve-kvm-host`: makes the chapter's end state real when a student skips ahead

## How the player runs it

1. rodeo-cli builds the `suse-edge` lab and the player clones this repo.
2. Entering a chapter runs its `setup-kvm-host`, if any.
3. **Check** runs `check-kvm-host` (120 s cap). Exit 0 completes the chapter.
4. **Skip** runs `solve-kvm-host` for each chapter it passes (30 min cap each).
5. Chapter 8 has no check, so the player offers **Next** there.

Scripts run as **root** on the KVM host, in the chapter directory, with a wiped
environment. They reach the VMs with `/root/.ssh/id_ed25519`. Checks test state
(an image file exists, a MachineInventory exists, a node is Ready, the app answers).
They never wait for a build. Solves that need `rodeo pull-edge-image` or
`rodeo eject-iso` run them as the `student` user, the path verified live.

### Solve coverage

| Chapter | Solve |
|---------|-------|
| 1 | No-op (read-only tour) |
| 2 | Full: workspace from Gitea, live registration config |
| 3 | Full: the four EIB builds in parallel, each in its own workspace copy |
| 4 | Best effort: pulls the ISOs, presses Enter on a timer with `virsh send-key`, waits for registration, ejects. On a very slow host the keypresses can miss, and the script then says so |
| 5 | Full: seeds and boots edge3/edge4, waits for Ready |
| 6 | Full: selector, label, cluster, waits for ready |
| 7 | Full: labels, imports edge3/edge4 through the API, waits for the app |

## Working on the lab

Test a live lab with the player's track test:

```bash
rodeo player test --expect suse-edge-rodeo.expect.yaml
```

Before a commit, check the scripts and the prose:

```bash
for f in 0*/*-kvm-host; do bash -n "$f"; done
shellcheck 0*/*-kvm-host
```

Assignments keep to short sentences, no em dashes and the
customer problem first. Keep each teaser under 255 bytes.

## Contributing

Keep this repo lab-only: anything about building the host, the VMs or the player
belongs in **[rodeo-cli](https://github.com/avaleror/rodeo-cli)**.

Track developer: andres.valero@suse.com

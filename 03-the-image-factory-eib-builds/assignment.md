---
slug: the-image-factory-eib-builds
id: 0z4ienvp7ejs
type: challenge
title: "\U0001F3ED Chapter 3: The Image Factory"
teaser: Four nodes, four images, one factory. Build two Elemental installer ISOs and
  two ready-to-run RKE2 and K3s disk images with Edge Image Builder, each with its own
  static IP baked in.
tabs:
- id: red30vjykq8o
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: irivq0z51qcq
  title: Rancher Prime UI
  type: service
  hostname: kvm-host
  path: /
  port: 30002
  protocol: https
- id: ad1zccpav1gw
  title: Gitea
  type: service
  hostname: kvm-host
  path: /gitea/eib-config
  port: 3000
  protocol: http
difficulty: intermediate
timelimit: 3600
enhanced_loading: null
---

🏭 Chapter 3: The Image Factory
===============================

<style type="text/css">
  * {
    font-family: suse;
    src: url('https://fonts.google.com/specimen/SUSE');
  }
  .suse { color: #30ba78; }
  .edge { color: #30ba78; }
  .bank { color: #d4af37; }
  .danger { color: #FE7C3F; font-weight: bold; }
  .hovereffect {
    border-radius: 25px 25px 25px 25px;
    background: linear-gradient(#30ba78 0 0) var(--hundredpercent, 0) / var(--hundredpercent, 0) no-repeat;
    transition: 0.5s, background-position 0s;
    padding: 5px;
  }
  .hovereffect:hover {
    --hundredpercent: 100%;
    color: white;
    border-radius: 10px 25px 10px 25px;
  }
  .story {
    border-left: 5px solid #d4af37;
    border-radius: 0 15px 15px 0;
    background: linear-gradient(135deg, rgba(48,186,120,.10), rgba(212,175,55,.10));
    padding: 15px 20px;
    margin: 15px 0;
  }
  .story em { color: #d4af37; }
  .missionbox {
    border: 2px dashed #30ba78;
    border-radius: 15px;
    padding: 12px 18px;
    margin: 15px 0;
  }
  .highlightcopy { color: white; font-weight: bold; padding: 0 10px; }
  img.logos { border-radius: 10px; }
  /* compact credential boxes (scoped: only code blocks inside <div class="cred">) */
  .cred > div { margin: 0; }
  .cred .my-3 {
    display: flex;
    flex-direction: row-reverse;
    align-items: stretch;
    width: fit-content;
    min-width: 14em;
    margin: 4px 0;
    overflow: hidden;
  }
  .cred .my-3 > div:first-child {
    height: auto;
    padding: 2px 8px;
    border-bottom: none;
    border-left: 1px solid white;
    border-radius: 0;
    display: flex;
    align-items: center;
    background-color: #30ba78;
  }
  .cred .my-3 > div:first-child,
  .cred .my-3 > div:first-child * {
    color: #fff !important;
  }
  .cred .my-3 > div:first-child:hover,
  .cred .my-3 > div:first-child:hover * {
    font-weight: bold;
  }
  .cred .my-3 > pre {
    flex: 1 1 auto;
    margin: 0 !important;
    padding: 2px !important;
    border-radius: 0 !important;
    display: flex;
    align-items: center;
  }
</style>

<div id="301" class="story">

*"Two kinds of sites,"* Sarah says, counting on her fingers. *"Regional hubs run containerized apps and need real Kubernetes clusters. Small branches just need a managed node and a light workload."*

*"And we don't always know a node's job when we order it,"* she adds. *"Sometimes we do."*

Elemental covers the first case: one generic image, and HQ decides the node's role later. Edge Image Builder covers the second: bake Kubernetes into the image and the node is a cluster the moment it boots. Today you build both.

</div>

## <b class="hovereffect">The four images</b>

Each node gets its own image with its own static IP baked in:

| Image | Node | Base | IP | Kubernetes |
|-------|------|------|----|------------|
| `elemental-edge1.iso` | edge1 | openSUSE Leap Micro 6.2 SelfInstall ISO | 192.168.122.31 | None (Elemental decides later) |
| `elemental-edge2.iso` | edge2 | openSUSE Leap Micro 6.2 SelfInstall ISO | 192.168.122.32 | None (Elemental decides later) |
| `rke2-edge3.raw` | edge3 | openSUSE Leap Micro 6.2 Default RAW | 192.168.122.33 | RKE2 v1.36.3+rke2r1 |
| `k3s-edge4.raw` | edge4 | openSUSE Leap Micro 6.2 Default RAW | 192.168.122.34 | K3s v1.36.3+k3s1 |

Every build follows the same pattern:

1. Clear `network/` and put only that node's NMState file there
2. Clear `custom/scripts/` and put only the scripts that node needs there
3. Start EIB in the background
4. Move on to the next node

EIB runs every script it finds under `custom/scripts/`, and there is no way to pick a subset. Leftover scripts from another node's build would end up in the image, so step 2 matters.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Check the workspace and the staged base images
2. Start the Elemental build for edge1
3. Start the Elemental build for edge2
4. Start the standalone RKE2 build for edge3
5. Start the standalone K3s build for edge4
6. Watch the builds and verify the four images

</div>

> [!NOTE]
> The builds run in the background. Start one, check that it is moving, then start the next.
>
> On a fast host they finish in a few minutes. On a slower one, plan for up to 20 minutes.

🔐 Login Credentials
====================

The **Rancher Prime** UI uses the `admin` user. The password is in the **Credentials** panel in the header of this page.

Username:

<div class="cred">

```txt
admin
```

</div>

In the terminal you are the `student` user on the lab host. Your SSH key already opens the `rancher` and `eib` VMs as `root`, so `ssh root@<ip>` needs no password.

📂 Task 1: Check the workspace and base images
==============================================

Everything in this chapter runs on the `eib` VM. Open a session there in the [button label="Lab Terminal" variant="success"](tab-0) and stay in it until Task 6:

```bash
ssh root@192.168.122.20
```

You use two copies of the `eib-config` repo:

- `/home/eib-workspace` (from Chapter 2) holds `elemental/elemental_config.yaml`. It builds the Elemental images for edge1 and edge2.
- `/home/eib-standalone` (you create it in Task 4) is the same repo without `elemental/`. It builds the RKE2 and K3s images.

EIB treats any workspace with an `elemental/` directory as an Elemental build. It then refuses a standalone RAW build that does not side-load the Elemental packages. You cannot delete `elemental/` from the first workspace either, because the edge1 and edge2 builds read it several minutes into their run.

Check what is in place:

```bash
# Definition files and scripts from Gitea
ls /home/eib-workspace/

# openSUSE Leap Micro base images, staged from Hauler
ls -lh /home/eib-config/base-images/

# Elemental packages, staged from Hauler (edge1 and edge2 side-load these)
ls -lh /home/eib-config/rpms/

# The active network/ directory, filled at build time
mkdir -p /home/eib-workspace/network
```

Want to read a definition file before you build it? Open `elemental-edge1-definition.yaml` in the [button label="Gitea" variant="success"](tab-2) tab.

EIB runs as a container with three mounts:

- the workspace, mounted as `/eib`: definitions, `custom/scripts/`, `network/`, and `elemental/` for the Elemental builds
- `/home/eib-config/base-images`, read-only: the base OS images
- `/home/eib-config/rpms`, read-only: the `elemental-register` and `elemental-system-agent` packages, for edge1 and edge2 only

Because the Elemental builds side-load those packages, they need no SUSE Customer Center registration code.

💿 Task 2: Start the Elemental build for edge1
==============================================

This image boots, installs openSUSE Leap Micro to disk, and on first boot `elemental-register` calls home. The NMState config gives it the static IP 192.168.122.31.

Load edge1's network config. Elemental builds need no first-boot scripts, and EIB fails if `custom/scripts/` exists but is empty, so remove the directory:

```bash
rm -f /home/eib-workspace/network/*.yaml
cp /home/eib-workspace/network-configs/edge1.yaml /home/eib-workspace/network/
rm -rf /home/eib-workspace/custom/scripts
```

Start the build in the background:

```bash
podman run --rm --privileged \
  -v /home/eib-workspace:/eib:z \
  -v /home/eib-config/base-images:/eib/base-images:ro \
  -v /home/eib-config/rpms:/eib/rpms:ro \
  registry.suse.com/edge/3.7/edge-image-builder:1.3.4 \
  build --definition-file elemental-edge1-definition.yaml \
  > /tmp/eib-edge1.log 2>&1 &

echo "edge1 build PID: $!"
tail -5 /tmp/eib-edge1.log
```

💿 Task 3: Start the Elemental build for edge2
==============================================

Same Elemental path, different IP (192.168.122.32). Check edge1's log first, then swap the network file and start edge2:

```bash
tail -10 /tmp/eib-edge1.log

rm -f /home/eib-workspace/network/*.yaml
cp /home/eib-workspace/network-configs/edge2.yaml /home/eib-workspace/network/
rm -rf /home/eib-workspace/custom/scripts

podman run --rm --privileged \
  -v /home/eib-workspace:/eib:z \
  -v /home/eib-config/base-images:/eib/base-images:ro \
  -v /home/eib-config/rpms:/eib/rpms:ro \
  registry.suse.com/edge/3.7/edge-image-builder:1.3.4 \
  build --definition-file elemental-edge2-definition.yaml \
  > /tmp/eib-edge2.log 2>&1 &

echo "edge2 build PID: $!"
```

☸️ Task 4: Start the standalone RKE2 build for edge3
====================================================

This image boots as a running single-node RKE2 cluster. There is no registration step. The whole Kubernetes stack is inside the disk image, with the static IP 192.168.122.33 and the hostname `edge3` set at first boot.

Create the standalone workspace: the same repo, without `elemental/`:

```bash
mkdir -p /home/eib-standalone
curl -sL http://192.168.122.20:3000/gitea/eib-config/archive/main.tar.gz \
  | tar -xz --strip-components=1 -C /home/eib-standalone
rm -rf /home/eib-standalone/elemental
mkdir -p /home/eib-standalone/network
```

Load edge3's network config and only edge3's scripts:

```bash
rm -f /home/eib-standalone/network/*.yaml
cp /home/eib-standalone/network-configs/edge3.yaml /home/eib-standalone/network/

rm -rf /home/eib-standalone/custom/scripts
mkdir -p /home/eib-standalone/custom/scripts
cp /home/eib-standalone/scripts-available/60-hostname-edge3.sh \
   /home/eib-standalone/scripts-available/99-k3s-registries.sh \
   /home/eib-standalone/custom/scripts/
```

The `kubernetes:` section of the definition is what makes this build different. EIB fetches the RKE2 binary and every container image it needs from the Hauler registry, and all of it lands in the disk image. The node needs no internet and no management plane to start Kubernetes.

```bash
tail -10 /tmp/eib-edge2.log

podman run --rm --privileged \
  -v /home/eib-standalone:/eib:z \
  -v /home/eib-config/base-images:/eib/base-images:ro \
  registry.suse.com/edge/3.7/edge-image-builder:1.3.4 \
  build --definition-file rke2-edge3-definition.yaml \
  > /tmp/eib-rke2.log 2>&1 &

echo "edge3 RKE2 build PID: $!"
```

☸️ Task 5: Start the standalone K3s build for edge4
===================================================

Same standalone pattern as edge3, but with K3s. K3s has a smaller footprint, which suits the small branches:

```bash
rm -f /home/eib-standalone/network/*.yaml
cp /home/eib-standalone/network-configs/edge4.yaml /home/eib-standalone/network/

rm -rf /home/eib-standalone/custom/scripts
mkdir -p /home/eib-standalone/custom/scripts
cp /home/eib-standalone/scripts-available/60-hostname-edge4.sh \
   /home/eib-standalone/scripts-available/99-k3s-registries.sh \
   /home/eib-standalone/custom/scripts/

podman run --rm --privileged \
  -v /home/eib-standalone:/eib:z \
  -v /home/eib-config/base-images:/eib/base-images:ro \
  registry.suse.com/edge/3.7/edge-image-builder:1.3.4 \
  build --definition-file k3s-edge4-definition.yaml \
  > /tmp/eib-k3s.log 2>&1 &

echo "edge4 K3s build PID: $!"
```

👀 Task 6: Watch the builds and verify the images
=================================================

Follow all four logs at once (press Ctrl+C to stop watching):

```bash
watch -n5 'echo "=== edge1 (Elemental ISO) ===" && tail -3 /tmp/eib-edge1.log && \
           echo "=== edge2 (Elemental ISO) ===" && tail -3 /tmp/eib-edge2.log && \
           echo "=== edge3 (RKE2 RAW)     ===" && tail -3 /tmp/eib-rke2.log && \
           echo "=== edge4 (K3s RAW)      ===" && tail -3 /tmp/eib-k3s.log'
```

A successful build ends with:

```
Build complete, the image can be found at: <outputImageName>
```

The `99-k3s-registries.sh` script points the RKE2 and K3s builds at the Hauler registry on `192.168.122.20:5000`, so nothing comes from the internet.

When all four are done, check the output files:

```bash
ls -lh /home/eib-workspace/elemental-edge1.iso \
       /home/eib-workspace/elemental-edge2.iso \
       /home/eib-standalone/rke2-edge3.raw \
       /home/eib-standalone/k3s-edge4.raw
```

Then leave the `eib` VM:

```bash
exit
```

> [!NOTE]
> If a log ends with an error, read the last lines. A common cause is a leftover file in `network/` or `custom/scripts/`. Fix it and run that one build again.

💼 Why does this matter?
========================

- **The image is the configuration.** Network, packages, registration and even Kubernetes are inside the image. Nothing is applied by hand after boot.
- **One factory for both site types.** The same tool and the same repo build a generic Elemental installer and a ready-to-run cluster disk.
- **Built from local artifacts.** Every container image came from Hauler, so this factory could sit in a site with no internet at all.

Click **Check** when the four images are built. If a build is still running, the check tells you which one.

📚 More information
===================

- [Edge Image Builder](https://github.com/suse-edge/edge-image-builder)
- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)
- [Hauler](https://docs.hauler.dev/)

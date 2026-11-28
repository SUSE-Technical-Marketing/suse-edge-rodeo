---
slug: ready-on-arrival-standalone-boot
id: amolowhu5pun
type: challenge
title: "\U0001F4E6 Chapter 5: Ready on Arrival"
teaser: Two branches already know what they need. Seed edge3 and edge4 from their RAW
  images, boot them, and confirm each one is a working RKE2 or K3s cluster the moment
  it comes up.
tabs:
- id: nza7sq1j18xp
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: wug5b14xojgt
  title: Rancher Prime UI
  type: service
  hostname: kvm-host
  path: /
  port: 30002
  protocol: https
difficulty: basic
timelimit: 1800
enhanced_loading: null
---

📦 Chapter 5: Ready on Arrival
==============================

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

<div id="501" class="story">

The next two sites are different. One is a regional processing hub, the other a small branch, and both have a fixed job that will not change.

*"Then don't make them wait for HQ,"* Sarah says. *"Ship them ready."*

That is what the RKE2 and K3s images from Chapter 3 are for. No installer, no registration, no reboot cycle. Power on and Kubernetes starts.

</div>

## <b class="hovereffect">The RAW workflow</b>

edge3 and edge4 boot from finished disk images. For a RAW image, `rodeo pull-edge-image` copies it into the host's libvirt storage once as a read-only base. Each node then gets a thin copy-on-write disk on top of it.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Seed edge3 and edge4 from their RAW images
2. Look at the disk layout
3. Start both nodes
4. Confirm each node runs its own Kubernetes cluster

</div>

🔐 Login Credentials
====================

The **Rancher Prime** UI uses the `admin` user. The password is below, and also in the **Credentials** panel in the header.

Username:

<div class="cred">

```txt
admin
```

</div>

Password:

<div class="cred">

```txt
[[ Instruqt-Var key="RANCHER_PASSWORD" hostname="kvm-host" ]]
```

</div>

In the terminal you are the `student` user on the lab host. Your SSH key already opens the `rancher` and `eib` VMs as `root`, so `ssh root@<ip>` needs no password.

💾 Task 1: Seed edge3 and edge4
===============================

In the [button label="Lab Terminal" variant="success"](tab-0), pull each image from the `eib` VM:

```bash
# edge3 gets the RKE2 image
rodeo pull-edge-image \
  --image /home/eib-standalone/rke2-edge3.raw \
  --nodes edge3 \
  --yes

# edge4 gets the K3s image
rodeo pull-edge-image \
  --image /home/eib-standalone/k3s-edge4.raw \
  --nodes edge4 \
  --yes
```

🧱 Task 2: Look at the disk layout
==================================

See which file each node boots from:

```bash
virsh domblklist edge3
virsh domblklist edge4
```

Then list the volumes in the storage pool:

```bash
virsh pool-refresh default >/dev/null
virsh vol-list default --details | grep -E 'edge3|edge4|base'
```

The `-base` volumes hold the full image. The per-node disks are thin clones that only store what changes after boot. Ten edge3-class nodes would share one base and cost very little extra disk.

▶️ Task 3: Start both nodes
===========================

```bash
virsh start edge3
virsh start edge4
```

They need up to five minutes to come up. On first boot RKE2 and K3s create their TLS certificates, start the control plane and mark the node Ready.

☸️ Task 4: Confirm each node runs Kubernetes
============================================

Both nodes have static IPs, so there are no DHCP leases to look up. The images trust your SSH key for `root`. RKE2 keeps its own `kubectl` and kubeconfig under its install paths:

```bash
# edge3 (RKE2)
ssh root@192.168.122.33 \
  "/var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get nodes"

# edge4 (K3s)
ssh root@192.168.122.34 \
  "kubectl get nodes"
```

Each shows one node in `Ready` state: `v1.36.3+rke2r1` on edge3 and `v1.36.3+k3s1` on edge4. If SSH is refused or a node is `NotReady`, wait a minute and try again.

> [!NOTE]
> Rancher Prime does not know about these two clusters yet. You import them in Chapter 7.

💼 Why does this matter?
========================

- **Kubernetes on first boot.** The node needed no network access, no management plane and no operator to become a cluster.
- **Cheap to scale.** One base image, many thin clones. The same idea works for shipping disks to many identical sites.
- **Re-image to recover.** If a node breaks, write the image again. It comes back exactly as built.

Click **Check** when both clusters report `Ready`.

📚 More information
===================

- [RKE2 documentation](https://docs.rke2.io/)
- [K3s documentation](https://docs.k3s.io/)
- [Edge Image Builder](https://github.com/suse-edge/edge-image-builder)

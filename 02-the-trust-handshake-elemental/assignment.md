---
slug: the-trust-handshake-elemental
id: sa63p9bk1g3n
type: challenge
title: "\U0001F91D Chapter 2: The Trust Handshake"
teaser: A new branch node must call home and prove who it is, with nobody on site.
  Set up the Elemental registration, pull its config into the EIB workspace and check
  the network plan for every node.
tabs:
- id: ese0zmexsddp
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: gxh3v0wwygtu
  title: Rancher Prime UI
  type: service
  hostname: kvm-host
  path: /
  port: 30002
  protocol: https
- id: notu2o7r1nfe
  title: Gitea
  type: service
  hostname: kvm-host
  path: /gitea/eib-config
  port: 3000
  protocol: http
difficulty: intermediate
timelimit: 2400
enhanced_loading: null
---

🤝 Chapter 2: The Trust Handshake
=================================

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

<div id="201" class="story">

Sarah pulls up a map of the 47 branches.

*"Most of them have no IT staff. When a node dies, the branch manager plugs in a new box and calls us. Then someone drives out there."*

You point at the map. *"Next time, the new box calls us. It boots, proves it is a bank machine, and asks HQ what to do."*

*"Proves it how?"* she asks. *"Anyone can copy a disk."*

That is the right question. The answer is the TPM chip on the node's board.

</div>

## <b class="hovereffect">How Elemental onboards a node</b>

<b class="edge">Elemental</b> handles node onboarding. A node boots and the `elemental-register` agent starts. The agent calls a registration URL that you define on the management cluster.

The Elemental Operator checks the node's identity, here with its TPM. It then creates a `MachineInventory` record, and from that point the management cluster controls the node. Nobody logs in to the node and nobody runs scripts on it.

The registration URL has to be inside the OS image. So you set up registration first and build images after. That is this chapter.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Inspect the MachineRegistration
2. Capture the registration URL
3. Build the EIB workspace from Git
4. Embed the registration config in the workspace
5. Confirm the Elemental Operator is healthy
6. Check the node network plan

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

🔍 Task 1: Inspect the MachineRegistration
==========================================

A `MachineRegistration` says who may register and which labels a node gets when it does. The lab already created one. Look at it in the [button label="Lab Terminal" variant="success"](tab-0):

```bash
ssh root@192.168.122.9 \
  "kubectl get machineregistration -n fleet-default -o yaml"
```

These are the fields that matter:

```yaml
spec:
  machineName: "${System Information/Manufacturer}-${System Information/UUID}"
  config:
    elemental:
      registration:
        auth: tpm                   # TPM-based identity, bound to the hardware
      install:
        device: /dev/vda            # install target for Elemental's own installer
        poweroff: true              # power off after an Elemental-driven install
  machineInventoryLabels:
    manufacturer: "${System Information/Manufacturer}"
    productName: "${System Information/Product Name}"
```

`auth: tpm` means the node's registration identity comes from its TPM. A cloned disk in a different machine fails to register, because the TPM does not match. For branches where you cannot lock the server room, this is the point.

The `install` block is for Elemental's own installer. It does not apply to the openSUSE Leap Micro SelfInstall ISO you build in this lab, which has its own installer. Chapter 4 shows you its two prompts.

You can see the same object in the [button label="Rancher Prime UI" variant="success"](tab-1) under **OS Management > Registration Endpoints**.

🔗 Task 2: Capture the registration URL
=======================================

The registration name comes from the lab's plan name, so it differs between labs. Find it first, then read the URL from its status:

```bash
REGNAME=$(ssh root@192.168.122.9 \
  "kubectl get machineregistration -n fleet-default -o jsonpath='{.items[0].metadata.name}'")
echo "Registration name: $REGNAME"

REGURL=$(ssh root@192.168.122.9 \
  "kubectl get machineregistration $REGNAME -n fleet-default \
   -o jsonpath='{.status.registrationURL}'")
echo "Registration URL: $REGURL"
```

The name ends in `-reg-1`. Keep this terminal open, because Task 4 uses `$REGURL`.

> [!NOTE]
> If you open a new terminal or lose the variables, run the two commands above again.

📦 Task 3: Build the EIB workspace from Git
===========================================

The image definitions, network configs and first-boot scripts live in the `eib-config` repo. Open the [button label="Gitea" variant="success"](tab-2) tab to see it. You will find four definition files and the `elemental/`, `network-configs/` and `scripts-available/` directories.

The `eib` VM has no `git`, because it stays a minimal build host. Fetch the repo as an archive instead. You get the same file layout:

```bash
ssh root@192.168.122.20 \
  "mkdir -p /home/eib-workspace && \
   curl -sL http://192.168.122.20:3000/gitea/eib-config/archive/main.tar.gz \
     | tar -xz --strip-components=1 -C /home/eib-workspace && \
   ls /home/eib-workspace/"
```

`/home/eib-workspace` is the workspace for the two Elemental images in Chapter 3.

🧬 Task 4: Embed the registration config
========================================

The file `elemental/elemental_config.yaml` in the repo is only a placeholder. Replace it with the live config from the Elemental Operator. This command runs `curl` on the `eib` VM, with the URL you captured in Task 2:

```bash
ssh root@192.168.122.20 \
  "curl -sk '$REGURL' -o /home/eib-workspace/elemental/elemental_config.yaml && \
   cat /home/eib-workspace/elemental/elemental_config.yaml"
```

The file holds the registration URL, the CA certificate of the management cluster and the settings `elemental-register` needs to authenticate with the TPM.

`elemental/` is a directory EIB looks for on its own. When it is there, EIB adds the `elemental-register` and `elemental-system-agent` packages to the image and sets up registration on first boot. The branch needs no extra config.

> [!NOTE]
> If the output is empty or shows an HTML error, `$REGURL` was not set in this shell. Run Task 2 again, then this task.

🩺 Task 5: Confirm the Elemental Operator is healthy
====================================================

Nodes cannot register against a broken operator. Check it before you build any image:

```bash
ssh root@192.168.122.9 \
  "kubectl get pods -n cattle-elemental-system"
```

The `elemental-operator` pod should be `Running`. Some Elemental Operator releases also run a separate webhook pod. If it is missing, that alone is fine.

🗺️ Task 6: Check the node network plan
======================================

Each node gets a static IP baked into its image by EIB. The configs are already in the workspace:

```bash
ssh root@192.168.122.20 \
  "ls /home/eib-workspace/network-configs/ && \
   cat /home/eib-workspace/network-configs/edge1.yaml"
```

| Node | IP | Prefix | Gateway | DNS | MAC |
|------|----|--------|---------|-----|-----|
| edge1 | 192.168.122.31 | /24 | 192.168.122.1 | 192.168.122.1 | 02:00:00:0E:62:A1 |
| edge2 | 192.168.122.32 | /24 | 192.168.122.1 | 192.168.122.1 | 02:00:00:0E:62:A2 |
| edge3 | 192.168.122.33 | /24 | 192.168.122.1 | 192.168.122.1 | 02:00:00:0E:62:A3 |
| edge4 | 192.168.122.34 | /24 | 192.168.122.1 | 192.168.122.1 | 02:00:00:0E:62:A4 |

EIB picks up any YAML file in the `network/` directory of its workspace. Each build uses exactly one file: one node, one IP. In Chapter 3 you copy the right file into `network/` before each build.

The configs call the interface `eth0`, but that is only a logical name. At first boot, EIB's network tool matches each config to the real NIC by MAC address. The node keeps its kernel name for the NIC (`enp1s0` on these VMs) and still gets the right IP.

> [!NOTE]
> Do not add `net.ifnames=0` to the definitions. The SelfInstall ISO boots into the installed system once without that kernel argument, so the NIC would change name on the next reboot and lose its static IP.

💼 Why does this matter?
========================

- **Zero touch at the branch.** The node carries everything it needs to call home. The branch manager plugs it in and walks away.
- **Identity bound to hardware.** With `auth: tpm`, a stolen or copied disk cannot join the fleet as another node.
- **Config as code.** The image definitions and network plan live in Git, so every change has an author and a date.

Click **Check my work** to confirm the workspace and the registration config are in place.

📚 More information
===================

- [Elemental documentation](https://elemental.docs.rancher.com/)
- [Edge Image Builder: Elemental integration](https://github.com/suse-edge/edge-image-builder/blob/main/docs/building-images.md)
- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)

---
slug: phoning-home-elemental-boot
id: qhyieyo6t8ry
type: challenge
title: "\U0001F4DE Chapter 4: Phoning Home"
teaser: Two new branch nodes arrive with nothing but an installer. Boot edge1 and edge2
  from their Elemental ISOs, watch them register with HQ through their TPM, and switch
  them to boot from disk.
tabs:
- id: xhdmh4njgvjd
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: 83i0sfc4b9ky
  title: Rancher Prime UI
  type: service
  hostname: kvm-host
  path: /
  port: 30002
  protocol: https
difficulty: intermediate
timelimit: 2400
enhanced_loading: null
---

📞 Chapter 4: Phoning Home
==========================

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

<div id="401" class="story">

Two boxes arrive from the hardware supplier, labelled for a branch on the coast. In real life a courier would drive them out and the branch manager would plug them in. Here, you are both.

*"Show me the part where nobody configures anything,"* Sarah says.

You attach the installers and power the nodes on. Then you watch the Rancher Prime UI and wait for two strangers to introduce themselves.

</div>

## <b class="hovereffect">The ISO workflow</b>

edge1 and edge2 each boot from their own ISO. The ISO holds a self-installer. It writes openSUSE Leap Micro to the virtual disk with the static IP already set, then boots into the installed system, where `elemental-register` calls home.

You do this from the lab host with two tools:

- `rodeo pull-edge-image` copies an image from the `eib` VM and attaches it to a node. For an ISO it adds a CD-ROM drive that boots first and creates a blank disk to install to.
- `virsh` starts, stops and talks to the VMs. Your user is in the `libvirt` group, so no `sudo` is needed.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Attach each node's ISO
2. Start the nodes and confirm the install
3. Watch both nodes register with HQ
4. Eject the ISOs and boot from disk

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

💿 Task 1: Attach each node's ISO
=================================

Each node gets its own ISO, because each has its own IP baked in. In the [button label="Lab Terminal" variant="success"](tab-0), pull both from the `eib` VM:

```bash
# edge1 gets its own ISO (192.168.122.31 baked in)
rodeo pull-edge-image \
  --image /home/eib-workspace/elemental-edge1.iso \
  --nodes edge1 \
  --yes

# edge2 gets its own ISO (192.168.122.32 baked in)
rodeo pull-edge-image \
  --image /home/eib-workspace/elemental-edge2.iso \
  --nodes edge2 \
  --yes
```

Each command copies the ISO into the host's libvirt storage, attaches it as a CD-ROM (boot order 1) and creates a blank disk for the OS (boot order 2).

Confirm the CD-ROM is there:

```bash
virsh domblklist edge1 --details
```

⌨️ Task 2: Start the nodes and confirm the install
=================================================

Start both nodes:

```bash
virsh start edge1
virsh start edge2
```

The installer needs two keypresses per node. The GRUB menu waits until you pick an entry, so install media never wipes a disk by accident. Then the partitioner asks you to confirm before it writes to `/dev/vda`.

Open edge1's console and press Enter at both points. Press Ctrl+] to leave the console:

```bash
virsh console edge1
# Press Enter to select "Install openSUSE Leap Micro"
# Press Enter again to confirm "Destroying ALL data on /dev/vda, continue?"
```

If the console shows nothing, send the keys from the host instead. Wait about 30 seconds after the first one, until the confirmation dialog is on screen:

```bash
virsh send-key edge1 KEY_ENTER     # GRUB: "Install openSUSE Leap Micro"
virsh send-key edge1 KEY_ENTER     # "Destroying ALL data on /dev/vda, continue?" -> Yes
```

Do the same for edge2. After the confirmation, the rest runs on its own.

A progress bar shows the OS being written, then the node boots into the installed system. A few minutes later the console shows a login prompt with the node's IP (`enp1s0: 192.168.122.31` for edge1).

📡 Task 3: Watch both nodes register with HQ
============================================

When `elemental-register` starts, it calls the registration URL you baked into the image. The Elemental Operator checks the TPM and creates a `MachineInventory`:

```bash
ssh root@192.168.122.9 \
  "kubectl get machineinventory -n fleet-default"
```

Run it again every minute or so until two entries appear, one per node. Each entry is a node that proved its TPM identity and is now waiting for instructions.

Open the [button label="Rancher Prime UI" variant="success"](tab-1) and go to **OS Management**. The same records are there, with hardware details and the labels from the MachineRegistration.

Which entry is which node? The inventory name comes from the system UUID, and libvirt shows the same UUID for each VM:

```bash
virsh domuuid edge1
virsh domuuid edge2
```

You use this in Chapter 6 to pick edge1 for a cluster.

⏏️ Task 4: Eject the ISOs and boot from disk
============================================

The ISO is still attached and still first in the boot order. Without this step, the next reboot would land on the installer's GRUB menu again.

Shut both nodes down cleanly and wait until both show `shut off`:

```bash
virsh shutdown edge1
virsh shutdown edge2
watch "virsh list --all | grep edge"
```

Press Ctrl+C to stop watching. Then remove the CD-ROMs, which also puts the disk first in the boot order, and start the nodes again:

```bash
rodeo eject-iso --nodes edge1,edge2 --yes
virsh start edge1
virsh start edge2
```

They now boot from the installed disk and keep their static IPs:

```bash
ping -c3 192.168.122.31
ping -c3 192.168.122.32
```

💼 Why does this matter?
========================

- **Nobody configured these nodes.** They installed, found HQ and identified themselves. A branch manager can do this part.
- **HQ decides what they become.** The nodes are registered but have no role yet. In Chapter 6 you turn one into a Kubernetes cluster without logging in to it.
- **Hardware-bound trust.** The `MachineInventory` is tied to each node's TPM. To revoke one node, delete its `MachineInventory`.

Click **Check** when both nodes are registered and running from disk.

📚 More information
===================

- [Elemental documentation](https://elemental.docs.rancher.com/)
- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)

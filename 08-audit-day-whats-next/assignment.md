---
slug: audit-day-whats-next
id: isgxg76n1n3q
type: challenge
title: "\U0001F305 Chapter 8: Audit Day"
teaser: The auditors arrive and every node tells the same story. Review what you built,
  take a victory lap in the lab, and see where SUSE Edge can take your own sites next.
tabs:
- id: qug1m2hxlfd0
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: evl9y3f11f3v
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

🌅 Chapter 8: Audit Day
=======================

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

<div id="801" class="story">

The auditors sit in the boardroom with a list of questions.

Where does each node's OS come from? Who approved the software on it? What happens when a node is replaced?

You open the Rancher Prime UI and answer from one screen. The images and the app each come from one Git repo. A new node proves its identity with its TPM before HQ gives it a job.

Sarah walks you to the lift afterwards. *"Four nodes done,"* she says. *"Now let's do the rest of the 47 branches."*

</div>

## <b class="hovereffect">🏆 Your Deeds</b>

| Chapter | Problem | What you did |
|:--------|:--------|:-------------|
| 🏦 The Audit Notice | 47 branches, no two alike | Toured the management cluster, the image factory and the Git repos |
| 🤝 The Trust Handshake | New nodes with nobody on site | Set up Elemental registration with TPM identity |
| 🏭 The Image Factory | Two kinds of site | Built two Elemental ISOs and two RKE2/K3s RAW images with EIB |
| 📞 Phoning Home | Onboarding without a technician | Installed edge1 and edge2 and watched them register |
| 📦 Ready on Arrival | Sites with a fixed job | Booted edge3 and edge4 as working clusters |
| 🏗️ The Regional Hub | A cluster nobody installs by hand | Turned edge1 into a K3s cluster with one label |
| 🚚 Every Branch at Once | The same app everywhere | Imported the standalone clusters and deployed with Fleet |

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

🧭 Victory lap: the lab is still yours
======================================

The lab stays up until your timer runs out. Some ideas:

- **Take a final inventory.** In the [button label="Rancher Prime UI" variant="success"](tab-1), look at **Cluster Management**, **OS Management** and **Continuous Delivery** side by side. Three clusters, two registered nodes, one Git repo.
- **Give edge2 a job.** Write your own selector template with a new label, put that label on edge2, and create a second cluster for it. No step-by-step this time: Chapter 6 has everything you need.
- **Look at the whole fleet from the terminal:**

```bash
ssh root@192.168.122.9 "
  kubectl get machineinventory -n fleet-default
  kubectl get clusters.provisioning.cattle.io -n fleet-default
  kubectl get clusters.fleet.cattle.io -n fleet-default
  kubectl get bundle -n fleet-default
"
```

🚀 What's next?
===============

- 📖 Read the [SUSE Edge 3.7 documentation](https://documentation.suse.com/en-us/suse-edge/3.7/) for the full architecture, including the parts this lab skipped.
- 📦 Learn how [Hauler](https://docs.hauler.dev/) builds and carries an air-gap bundle, so sites with no internet still get every image they need.
- 🧪 Rebuild this on your own hardware. The [SUSE Edge workshop guide](https://avaleror.github.io/suse-edge-workshop/) runs the same exercises with [rodeo-cli](https://github.com/avaleror/rodeo-cli) on one KVM host.
- 🏦 Pair it with the SUSE Virtualization Rodeo, where Vertex Trust Bank moves its HQ datacenter off a legacy hypervisor.
- 💬 Talk to your <b class="suse">SUSE</b> representative about what this would look like at your own remote sites: stores, factories, branches or cell towers.

<div id="802" class="story">

It has been a pleasure working with you.

**Happy onboarding!** 🎉

</div>

📚 More information
===================

- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)
- [Elemental documentation](https://elemental.docs.rancher.com/)
- [Edge Image Builder](https://github.com/suse-edge/edge-image-builder)
- [Fleet documentation](https://fleet.rancher.io/)
- [Rancher Prime documentation](https://ranchermanager.docs.rancher.com/)

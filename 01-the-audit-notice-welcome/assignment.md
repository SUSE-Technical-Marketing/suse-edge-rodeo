---
slug: the-audit-notice-welcome
id: cyo7w6zvprsl
type: challenge
title: "\U0001F3E6 Chapter 1: The Audit Notice"
teaser: Vertex Trust Bank runs 47 branches, and every branch has its own config history.
  The regulators arrive next month. Meet the platform that will fix it and tour your
  management cluster and image factory.
notes:
- type: text
  contents: |
    # Welcome to the SUSE Edge Rodeo!

    Please wait while we prepare your lab environment.

    Sarah, the CTO of Vertex Trust Bank, has a letter from the regulators on her desk.
    She wants to see you before lunch.
tabs:
- id: l65its0su2e2
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: qeq75lz823a7
  title: Rancher Prime UI
  type: service
  hostname: kvm-host
  path: /
  port: 30002
  protocol: https
- id: ax2tfii0sf35
  title: Gitea
  type: service
  hostname: kvm-host
  path: /explore/repos
  port: 3000
  protocol: http
difficulty: basic
timelimit: 1800
enhanced_loading: null
---

🏦 Chapter 1: The Audit Notice
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

<div id="101" class="story">

The headquarters datacenter at <b class="bank">Vertex Trust Bank</b> is quiet for the first time in years. The migration is done. Sarah, the CTO, should be happy, but she is holding a letter.

*"The regulators are auditing our branches next month,"* she says. *"47 locations, three to five Linux nodes each, ATMs, teller terminals and regional hubs. Some of those nodes have not been patched in years, and no two branches are set up the same way."*

She slides the letter across the table.

*"I want every edge node on the same OS, installed from the same image, managed from one place. And when a branch gets a new node, nobody on site should have to configure it. Can you do that?"*

</div>

Before you promise anything, look at what you have to work with.

## <b class="hovereffect">What is SUSE Edge?</b>

<b class="edge">SUSE Edge</b> is a set of open source components for running Linux and Kubernetes at many small, remote sites from one central place. In this lab you use SUSE Edge 3.7:

- **Rancher Prime**: the management plane. One UI and one API for every cluster in every branch.
- **Elemental**: node onboarding. A new node boots, proves who it is with its TPM chip, and registers itself with Rancher Prime.
- **Edge Image Builder (EIB)**: builds ready-to-boot OS images with the network config, packages and even Kubernetes already inside.
- **Fleet**: GitOps delivery. Point it at a Git repo and it rolls the same app out to every matching cluster.
- **Hauler**: a portable store for container images and files, so sites with a poor link install from a local copy.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Find out which VMs are running and which are still off
2. Inspect the management cluster
3. Inspect the image factory and its local artifact store
4. Look at the Git repos the lab will build from

</div>

> [!NOTE]
> This lab is for learning. It shows how the pieces fit together, not how to size or secure a production bank. Some choices are there only because everything runs inside one host.

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

> [!NOTE]
> The Rancher Prime UI uses a self-signed certificate. Accept the browser warning when it appears. If a page does not load, wait a minute and refresh the tab.

🖥️ Task 1: Find out what is running
===================================

The whole lab runs inside one KVM host. Your terminal is on that host. Open the [button label="Lab Terminal" variant="success"](tab-0) and list the VMs:

```bash
virsh list --all
```

Two VMs are running and four are shut off:

| VM | IP | Role |
|----|----|------|
| `rancher` | 192.168.122.9 | Management cluster: K3s, Rancher Prime, Elemental Operator, Fleet |
| `eib` | 192.168.122.20 | Image factory: Edge Image Builder, Hauler, Gitea |
| `edge1`, `edge2` | .31, .32 | Branch nodes for the Elemental path (they have a virtual TPM) |
| `edge3`, `edge4` | .33, .34 | Branch nodes for the standalone path |

The four edge nodes have no OS yet. You build their images in Chapter 3 and boot them in Chapters 4 and 5.

🐮 Task 2: Inspect the management cluster
=========================================

The `rancher` VM is the bank's HQ for edge operations. Check its Kubernetes node and the pods that matter for this lab:

```bash
ssh root@192.168.122.9 \
  "kubectl get nodes && kubectl get pods -A | grep -E 'rancher|elemental|cert-manager|fleet'"
```

You should see one K3s node in `Ready` state. You should also see pods for Rancher Prime, cert-manager, the Elemental Operator and Fleet. This one small cluster will manage everything else.

Now open the [button label="Rancher Prime UI" variant="success"](tab-1) and log in with the credentials above. Take a minute to look around:

- **Cluster Management** lists every cluster Rancher Prime manages. Today there is only `local`, the management cluster itself.
- **OS Management** is the Elemental UI. It is empty for now. Your branch nodes will show up here when they register.
- **Continuous Delivery** is Fleet. A Git repo called `vertex-bank-app` is already set up, but no cluster matches it yet.

> [!NOTE]
> In this lab the management cluster runs on K3s to save resources. A production SUSE Edge management cluster usually runs on RKE2.

🏭 Task 3: Inspect the image factory
====================================

The `eib` VM builds the OS images for the branch nodes. Check that the EIB container image is there, that the two Hauler services are up, and what is staged for you:

```bash
ssh root@192.168.122.20 \
  "podman images | grep edge-image-builder && \
   systemctl is-active hauler-registry hauler-fileserver && \
   ls /home/eib-config/"
```

Hauler runs two services:

- an OCI registry on port `5000` for container images
- an HTTP file server on port `8080` for binaries and OS base images

Everything the lab installs comes from one of these two endpoints. Once Hauler is filled, the `eib` VM does not need the internet. Look inside the store:

```bash
ssh root@192.168.122.20 \
  "hauler store info --store /var/lib/hauler 2>/dev/null | head -40"
```

You should see the EIB container image, the Elemental register agent, the `vertex-bank-app` image and the openSUSE Leap Micro base images. They were staged when the lab was built.

📚 Task 4: Look at the Git repos
================================

The `eib` VM also runs Gitea, a small Git server. Open the [button label="Gitea" variant="success"](tab-2) tab. You can browse without logging in.

There are two repos:

- **eib-config**: the EIB image definitions, the network config for each node and the scripts the images run at first boot. You use it in Chapters 2 and 3.
- **vertex-bank-app**: the bank's branch dashboard app. Fleet deploys it to the branches in Chapter 7.

Keeping both in Git means every image and every deployment has a history. That is exactly what the auditors will ask for.

💼 Why does this matter?
========================

- **One control plane for every site.** Rancher Prime, Elemental and Fleet all run on one small management cluster at HQ.
- **Images, not hand-made configs.** Branch nodes will boot from images built in one place. No two nodes drift apart, because nobody logs in to change them.
- **Local artifacts for weak links.** Hauler keeps every image and file a branch needs close to it.

Click **Check** to confirm the management cluster and the image factory are healthy.

📚 More information
===================

- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)
- [Elemental](https://elemental.docs.rancher.com/)
- [Edge Image Builder](https://github.com/suse-edge/edge-image-builder)
- [Hauler](https://docs.hauler.dev/)

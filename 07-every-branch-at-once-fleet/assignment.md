---
slug: every-branch-at-once-fleet
id: 7wxgxv1e3tbc
type: challenge
title: "\U0001F69A Chapter 7: Every Branch at Once"
teaser: Three clusters, two provisioning paths, one app to roll out. Import the standalone
  clusters into Rancher Prime and let Fleet deploy the bank's branch dashboard to every
  matching cluster from Git.
tabs:
- id: vzkscm362gq0
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: clxmhhyd8ifp
  title: Rancher Prime UI
  type: service
  hostname: kvm-host
  path: /
  port: 30002
  protocol: https
- id: 00uwp2ly48ap
  title: Gitea
  type: service
  hostname: kvm-host
  path: /gitea/vertex-bank-app
  port: 3000
  protocol: http
difficulty: intermediate
timelimit: 2400
enhanced_loading: null
---

🚚 Chapter 7: Every Branch at Once
==================================

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

<div id="701" class="story">

The auditors want proof that every branch runs the same approved software. Sarah wants it without a weekend of remote sessions.

*"How long to put the branch dashboard on every site?"* she asks.

*"As long as it takes Fleet to notice a label,"* you say. *"First, HQ needs to see all three clusters."*

</div>

## <b class="hovereffect">Where things stand</b>

You now have:

- **vertex-hub-01** on edge1: K3s, provisioned by Rancher Prime through Elemental
- **edge2**: registered, with no cluster yet
- **edge3**: a standalone RKE2 cluster that Rancher Prime does not know yet
- **edge4**: a standalone K3s cluster that Rancher Prime does not know yet

Fleet already watches for clusters with the right labels. A `GitRepo` called `vertex-bank-app` targets every cluster labelled `demo=true` and `edge-type=x86-cluster`.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Find where Fleet reads the app from
2. Target vertex-hub-01
3. Import edge3 and edge4 into Rancher Prime
4. Target the two branch clusters
5. Verify the app on every cluster

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

📜 Task 1: Find where Fleet reads the app from
==============================================

The `vertex-bank-app` GitRepo points at the Gitea server on the `eib` VM, not at GitHub. No internet is needed. In the [button label="Lab Terminal" variant="success"](tab-0):

```bash
ssh root@192.168.122.9 \
  "kubectl get gitrepo vertex-bank-app -n fleet-default -o jsonpath='{.spec.repo}'; echo"
```

Open the [button label="Gitea" variant="success"](tab-2) tab to see the repo itself: the Kubernetes manifests and the `fleet.yaml` that tells Fleet how to deploy them. Whatever is in Git is what every branch runs.

🎯 Task 2: Target vertex-hub-01
===============================

edge1's `MachineInventory` has the `demo` and `edge-type` labels from Chapter 6. Those labels do not carry over to the cluster object Fleet watches, so label that one too:

```bash
ssh root@192.168.122.9 \
  "kubectl label clusters.fleet.cattle.io vertex-hub-01 -n fleet-default \
   demo=true edge-type=x86-cluster --overwrite"
```

Use the full name `clusters.fleet.cattle.io`. A bare `kubectl label cluster ...` picks a different `Cluster` resource. It succeeds with no error, but Fleet keeps showing 0 targeted clusters.

In the [button label="Rancher Prime UI" variant="success"](tab-1), open **Continuous Delivery > Git Repos > vertex-bank-app**. Within about 15 seconds `vertex-hub-01` shows up as a target.

📥 Task 3: Import edge3 and edge4
=================================

In the Rancher Prime UI, go to **Cluster Management > Import Existing > Generic**. Create two clusters:

<div class="cred">

```txt
vertex-branch-rke2
```

</div>

<div class="cred">

```txt
vertex-branch-k3s
```

</div>

For each one, Rancher Prime shows registration commands. This lab's Rancher Prime uses a self-signed certificate, so copy the **second** command, the `curl --insecure ... | kubectl apply -f -` one. The plain `kubectl apply -f <url>` fails with `x509: certificate signed by unknown authority`.

Replace `<registration-manifest-url>` below with the URL from that command, and run each one on its own node:

```bash
# On edge3, for vertex-branch-rke2 (RKE2 keeps its kubectl under /var/lib/rancher/rke2/bin)
ssh root@192.168.122.33 \
  "curl --insecure -sfL <registration-manifest-url> | \
   /var/lib/rancher/rke2/bin/kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml apply -f -"

# On edge4, for vertex-branch-k3s
ssh root@192.168.122.34 \
  "curl --insecure -sfL <registration-manifest-url> | kubectl apply -f -"
```

Each cluster turns `Active` in **Cluster Management** within a couple of minutes.

🏷️ Task 4: Target the two branch clusters
=========================================

Fleet registers its own copy of each imported cluster shortly after it turns `Active`. Check that both are listed before you label them, because labelling too early fails with `NotFound`:

```bash
ssh root@192.168.122.9 \
  "kubectl get clusters.fleet.cattle.io -n fleet-default"
```

When `vertex-branch-rke2` and `vertex-branch-k3s` are listed, label them:

```bash
ssh root@192.168.122.9 "
  kubectl label clusters.fleet.cattle.io vertex-branch-rke2 -n fleet-default \
    demo=true edge-type=x86-cluster --overwrite
  kubectl label clusters.fleet.cattle.io vertex-branch-k3s -n fleet-default \
    demo=true edge-type=x86-cluster --overwrite
"
```

Fleet deploys the app as soon as the labels match. Watch the per-cluster status in **Continuous Delivery > Git Repos > vertex-bank-app**.

✅ Task 5: Verify the app on every cluster
==========================================

Check the bundle status from the management cluster:

```bash
ssh root@192.168.122.9 \
  "kubectl get bundle -n fleet-default | grep vertex-bank"
```

The app runs on the edge clusters, not on the management cluster. Look at it on vertex-hub-01 through the kubeconfig Rancher Prime keeps for that cluster:

```bash
ssh root@192.168.122.9 "
  kubectl get secret vertex-hub-01-kubeconfig -n fleet-default \
    -o jsonpath='{.data.value}' | base64 -d > /tmp/vertex-hub-01.yaml
  kubectl --kubeconfig /tmp/vertex-hub-01.yaml get pods,svc -n vertex-bank
"
```

The service publishes NodePort `30080`. Ask each node for the page title:

```bash
curl -s http://192.168.122.31:30080/ | grep -o "<title>.*</title>"   # vertex-hub-01 (edge1)
curl -s http://192.168.122.33:30080/ | grep -o "<title>.*</title>"   # vertex-branch-rke2 (edge3)
curl -s http://192.168.122.34:30080/ | grep -o "<title>.*</title>"   # vertex-branch-k3s (edge4)
```

> [!NOTE]
> An older copy of vertex-bank-app may publish a `LoadBalancer` service with `EXTERNAL-IP` stuck at `<pending>`, and Rancher Prime may show the bundle as `NotReady`. Nothing in this lab hands out LoadBalancer IPs. The app still answers on the NodePort.

💼 Why does this matter?
========================

Four nodes, four images, two provisioning paths, one management plane.

- **The image is the configuration.** Every node booted from a purpose-built image. A broken node gets re-imaged, a new branch gets the image.
- **Two paths for two kinds of site.** Elemental suits sites where the node's role is decided after it arrives. EIB standalone suits sites where the role is fixed when you build the image.
- **Trust from the hardware.** With `auth: tpm`, a cloned disk cannot register as another node, and you revoke a node by deleting its `MachineInventory`.
- **Local artifacts.** Hauler kept every image and file on site. In production you sync it at HQ and carry it to the sites that need it.

| | Elemental (edge1, edge2) | EIB standalone (edge3, edge4) |
|---|---|---|
| Node role decided | After registration | At image build time |
| Who runs Kubernetes | Rancher Prime provisions and manages it | The node runs its own, imported if you want |
| Onboarding | Automatic, TPM call home | Manual import |
| Good for | Mixed-role sites, large fleets | Fixed-function nodes, strict air gaps |
| Recovery | Register again, provision again | Write the image again |

Click **Check** when the app answers on all three clusters.

📚 More information
===================

- [Fleet documentation](https://fleet.rancher.io/)
- [Rancher Prime: importing existing clusters](https://ranchermanager.docs.rancher.com/)
- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)

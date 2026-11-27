---
slug: the-regional-hub-provision-cluster
id: 6jcibn9hhb9b
type: challenge
title: "\U0001F3D7️ Chapter 6: The Regional Hub"
teaser: edge1 is registered but has no job yet. Tell HQ it is a regional hub, and
  watch Rancher Prime install K3s on it and build a cluster without anyone logging in
  to the node.
tabs:
- id: devc4z71y9il
  title: Lab Terminal
  type: terminal
  hostname: kvm-host
- id: 5wxph1c05l8b
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

🏗️ Chapter 6: The Regional Hub
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

<div id="601" class="story">

The coastal region needs a hub: a small Kubernetes cluster that runs the branch dashboard for every office nearby. edge1 is the box for the job. It has registered and is waiting for work.

*"So someone SSHs in and installs Kubernetes?"* Sarah asks.

*"Nobody touches it,"* you say. *"I put a label on it at HQ. Rancher Prime does the rest."*

</div>

## <b class="hovereffect">From registered node to cluster</b>

edge1 and edge2 run openSUSE Leap Micro and have called home, but they are not Kubernetes nodes yet. Three objects turn a registered node into a cluster:

- **MachineInventory**: the registered node. You add labels to it.
- **MachineInventorySelectorTemplate**: a label selector. Any inventory that matches can join a machine pool.
- **Cluster**: a Rancher Prime cluster whose machine pool uses that selector.

<div class="missionbox">

## 🎯 Your Quest Objectives

1. Review the registered nodes
2. Create the selector template
3. Label edge1 as a hub
4. Create the cluster
5. Watch the cluster provision

</div>

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

📋 Task 1: Review the registered nodes
======================================

List the registered nodes in the [button label="Lab Terminal" variant="success"](tab-0):

```bash
ssh root@192.168.122.9 \
  "kubectl get machineinventory -n fleet-default --show-labels"
```

In the [button label="Rancher Prime UI" variant="success"](tab-1), open **OS Management**. The same records are there, with their hardware details and labels.

🎯 Task 2: Create the selector template
=======================================

This template matches any `MachineInventory` with the label `site-role: hub`:

```bash
cat << 'EOF' | ssh root@192.168.122.9 "kubectl apply -f -"
apiVersion: elemental.cattle.io/v1beta1
kind: MachineInventorySelectorTemplate
metadata:
  name: vertex-hub-selector
  namespace: fleet-default
spec:
  template:
    spec:
      selector:
        matchLabels:
          site-role: hub
EOF
```

🏷️ Task 3: Label edge1 as a hub
===============================

Neither node matches `site-role: hub` yet. Find edge1's inventory with the UUID libvirt reports for it:

```bash
EDGE1_UUID=$(virsh domuuid edge1)
EDGE1=$(ssh root@192.168.122.9 \
  "kubectl get machineinventory -n fleet-default -o name" \
  | grep -i "$EDGE1_UUID" | cut -d/ -f2)
echo "edge1 MachineInventory: $EDGE1"
```

If the output is empty, list the inventories with the command from Task 1 and pick the one whose name holds edge1's UUID. Then label it:

```bash
ssh root@192.168.122.9 \
  "kubectl label machineinventory $EDGE1 -n fleet-default \
   site-role=hub demo=true edge-type=x86-cluster"
```

Only `site-role=hub` matters for the selector. The other two labels describe the node for later.

☸️ Task 4: Create the cluster
=============================

Create a cluster called `vertex-hub-01`, with one machine pool that takes its node from the selector template:

```bash
cat << 'EOF' | ssh root@192.168.122.9 "kubectl apply -f -"
apiVersion: provisioning.cattle.io/v1
kind: Cluster
metadata:
  name: vertex-hub-01
  namespace: fleet-default
spec:
  kubernetesVersion: v1.36.3+k3s1
  rkeConfig:
    machinePools:
      - name: hub-nodes
        quantity: 1
        etcdRole: true
        controlPlaneRole: true
        workerRole: true
        machineConfigRef:
          kind: MachineInventorySelectorTemplate
          name: vertex-hub-selector
          apiVersion: elemental.cattle.io/v1beta1
EOF
```

⏳ Task 5: Watch the cluster provision
======================================

In the Rancher Prime UI, open **Cluster Management**. `vertex-hub-01` appears as `Provisioning`. Rancher Prime is now installing K3s on edge1 through the Elemental system agent.

From the terminal (press Ctrl+C to stop watching):

```bash
ssh -t root@192.168.122.9 \
  "kubectl get clusters.provisioning.cattle.io -n fleet-default -w"
```

Use the full name `clusters.provisioning.cattle.io`. Rancher Prime and Fleet define several resources called `Cluster`, and a bare `kubectl get cluster` may show a different one.

When `vertex-hub-01` shows `READY` as `true`, the cluster is up. It takes a few minutes.

💼 Why does this matter?
========================

- **Hands-off provisioning.** The node registered itself, you added a label, and the management cluster did the rest. Nobody logged in to edge1.
- **Roles decided late.** The same generic image could have become a hub, a worker or nothing at all. HQ decides after the node is on site.
- **Same model at any scale.** One node or a thousand, the steps do not change: register, label, let the selector match.

Click **Check** when `vertex-hub-01` is ready. If it is still provisioning, the check says so.

📚 More information
===================

- [Elemental documentation](https://elemental.docs.rancher.com/)
- [Rancher Prime: cluster provisioning](https://ranchermanager.docs.rancher.com/)
- [SUSE Edge documentation](https://documentation.suse.com/en-us/suse-edge/3.7/)

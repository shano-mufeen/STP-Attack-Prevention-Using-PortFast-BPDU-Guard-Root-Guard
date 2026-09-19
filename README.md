# 🛡️ STP Attack Prevention Using PortFast, BPDU Guard & Root Guard

## 📌 Project Overview

This project demonstrates how to improve **Spanning Tree Protocol (STP) security** in a Cisco switched network using three important STP protection mechanisms:

* ⚡ **PortFast**
* 🛡️ **BPDU Guard**
* 🔒 **Root Guard**

These features help protect the Layer 2 network from accidental loops, unauthorized switches, and attempts to manipulate the STP root bridge.

The project is implemented using **Cisco Packet Tracer** with multiple interconnected switches and end-user devices.

---

# 📋 Case Study

A company has a switched network where multiple Cisco switches are interconnected using trunk links.

The network contains:

* Multiple Cisco switches
* Inter-switch trunk links
* End-user access ports
* STP-enabled Layer 2 topology

The network administrator wants to protect the topology from potential STP-related attacks.

### ⚠️ Security Risks

An unauthorized switch could be connected to an access port and send **BPDUs** into the network.

This could potentially affect the STP topology and influence root bridge selection.

At the same time, normal end-user ports may take time to transition through STP states before reaching forwarding state.

### 🛡️ Security Solution

The following STP features are implemented:

| Technology     | Purpose                                                                                    |
| -------------- | ------------------------------------------------------------------------------------------ |
| ⚡ PortFast     | Quickly transitions edge/access ports to forwarding                                        |
| 🛡️ BPDU Guard | Disables a PortFast interface if a BPDU is received                                        |
| 🔒 Root Guard  | Prevents an interface from accepting a superior BPDU and becoming a path toward a new root |

---

# 🎯 Project Objectives

* Configure **STP PortFast** on end-device access ports.
* Configure **BPDU Guard** on PortFast interfaces.
* Configure **Root Guard** on selected trunk-facing interfaces.
* Protect the network from unauthorized STP BPDUs.
* Prevent an unauthorized switch from influencing root bridge selection.
* Verify PortFast and BPDU Guard operation.
* Verify Root Guard configuration.
* Test normal PC connectivity.
* Simulate an unauthorized switch connection.

---

# 🏢 Network Topology

```text
                         ┌───────────────┐
                         │    SWITCH 1   │
                         │   Root Bridge │
                         └───────┬───────┘
                                 │
                              Trunk
                                 │
                         ┌───────▼───────┐
                         │    SWITCH 2   │
                         │   STP Switch  │
                         └───────┬───────┘
                                 │
                         Root Guard Link
                                 │
                         ┌───────▼───────┐
                         │    SWITCH 3   │
                         │   Protected   │
                         └───────────────┘

       End Devices
           │
     ┌─────┴─────┐
     │           │
    PC1         PC2
     │           │
 PortFast +   PortFast +
 BPDU Guard   BPDU Guard
```

---

# 🧠 STP Security Concepts

## 🌳 What is STP?

**Spanning Tree Protocol (STP)** prevents Layer 2 switching loops when redundant paths exist between switches.

Without STP, redundant links can create loops that result in problems such as:

* Broadcast storms
* MAC address instability
* Duplicate frames
* Network instability

STP creates a loop-free logical topology by placing some redundant paths into a blocking state.

---

# ⚡ 1. PortFast

**PortFast** is designed for interfaces connected to end devices such as:

* 💻 PCs
* 🖨️ Printers
* 📡 End-user devices

Normally, an STP-enabled interface can take time to transition through STP states before forwarding traffic.

PortFast allows an edge/access port to transition to the forwarding state much faster.

### Without PortFast

```text
PC Connected
     ↓
Listening
     ↓
Learning
     ↓
Forwarding
     ↓
Network Access
```

### With PortFast

```text
PC Connected
     ↓
⚡ Fast Transition
     ↓
Forwarding
     ↓
Network Access
```

### Important

PortFast should normally be used on **edge/access ports connected to end devices**, not on switch-to-switch trunk links.

---

# 🛡️ 2. BPDU Guard

**BPDU Guard** protects PortFast-enabled interfaces from receiving BPDUs.

The basic security assumption is:

> An edge port configured for PortFast should be connected to an end device, not another switch.

If an unauthorized switch is connected and begins sending BPDUs, BPDU Guard can place the interface into an **err-disabled** state.

### Attack Scenario

```text
Unauthorized Switch
        │
        │ Sends BPDU
        ▼
PortFast + BPDU Guard
        │
        ▼
    BPDU Detected
        │
        ▼
  🚫 Interface Disabled
```

This helps prevent an unauthorized switch from participating in the STP topology through a protected edge port.

---

# 🔒 3. Root Guard

STP selects a **Root Bridge** based on the STP bridge ID, which is influenced by bridge priority and MAC address.

A switch sending a superior BPDU could potentially influence the STP topology.

**Root Guard** protects a designated interface from becoming a path toward an unexpected root bridge.

If a superior BPDU is received on a Root Guard-enabled interface, the interface can enter a **root-inconsistent** state until the superior BPDUs stop.

### Root Guard Concept

```text
Existing Root Bridge
        │
        │
        ▼
   Protected Link
        │
    Root Guard
        │
        ▼
 Unauthorized / Unexpected
      STP Root
        │
        🚫 Blocked
```

---

# 🛠️ Step 1 — Build the Topology

Create the multi-switch topology in **Cisco Packet Tracer**.

The topology should include:

* Multiple Cisco switches
* Redundant/inter-switch links
* End-user PCs
* Trunk connections
* Access ports

The objective is to have:

```text
Switch-to-Switch
      ↓
    Trunk
      ↓
      STP

PC-to-Switch
      ↓
   Access Port
      ↓
PortFast + BPDU Guard
```

---

# 🌐 Step 2 — Identify and Configure Trunk Ports

The switch-to-switch interfaces must operate as trunk links.

Example:

```cisco
enable
configure terminal

interface range fa0/1-2
switchport mode trunk
exit
```

Repeat the appropriate configuration on the required switches.

### Why?

Trunk links carry traffic from multiple VLANs between switches and participate in the Layer 2 topology.

---

# ⚡ Step 3 — Identify Access Ports

Access ports are interfaces connected to end-user devices.

For example, if:

```text
Fa0/1 – Fa0/2 → Trunk
Fa0/3 – Fa0/24 → End Devices
Gi0/1 – Gi0/2 → Other Interfaces
```

Then the appropriate end-device interfaces can be configured as access ports.

```cisco
interface range fa0/3-24
switchport mode access
```

---

# ⚡🛡️ Step 4 — Configure PortFast and BPDU Guard

PortFast and BPDU Guard are applied to the end-device access ports.

### Interface-Level Configuration

```cisco
interface range fa0/3-24
switchport mode access
spanning-tree portfast
spanning-tree bpduguard enable
exit
```

This configuration provides two protections:

```text
Access Port
     │
     ├── ⚡ PortFast
     │
     └── 🛡️ BPDU Guard
```

---

# 🌎 Step 5 — Configure PortFast and BPDU Guard Globally

The lab also demonstrates global configuration.

```cisco
spanning-tree portfast default
spanning-tree portfast bpduguard default
```

This enables the default behavior for appropriate PortFast/edge interfaces.

### ⚠️ Important

Global configuration should be used with an understanding of which interfaces are intended to be edge ports. **Do not treat switch-to-switch trunk links as end-device ports.**

---

# 🔒 Step 6 — Configure Root Guard

Root Guard is applied to the interface where the administrator wants to prevent an unexpected superior BPDU from influencing the STP root path.

Example:

```cisco
interface fa0/4
switchport mode trunk
spanning-tree guard root
exit
```

### Root Guard Protection

```text
SW1
Root Bridge
   │
   │
   ▼
SW2
   │
   │ Root Guard
   ▼
SW3

SW3 cannot influence this protected
STP path with a superior BPDU.
```

---

# 🧪 Step 7 — Verify PortFast

Use:

```cisco
show spanning-tree interface fa0/4 portfast
```

You can also inspect the STP summary:

```cisco
show spanning-tree summary
```

Look for PortFast-related information.

---

# 🔍 Step 8 — Verify BPDU Guard

Use:

```cisco
show spanning-tree summary
```

This allows you to verify the configured BPDU Guard behavior.

You can also inspect the interface configuration:

```cisco
show running-config interface fa0/4
```

Look for:

```text
spanning-tree portfast
spanning-tree bpduguard enable
```

---

# 🔒 Step 9 — Verify Root Guard

Check the interface configuration:

```cisco
show running-config interface fa0/4
```

You should see:

```text
spanning-tree guard root
```

You can also inspect STP information:

```cisco
show spanning-tree interface fa0/4 detail
```

---

# 🧪 Security Testing

## Test 1 — Connect a Normal PC

Connect a normal PC to a PortFast-enabled access port.

### Expected Behavior

The interface should transition to forwarding quickly.

```text
PC Connected
     ↓
PortFast
     ↓
⚡ Fast Forwarding
     ↓
Network Access
```

This demonstrates the purpose of PortFast.

---

# 🚨 Test 2 — Connect an Unauthorized Switch

Connect an additional switch to a port protected with BPDU Guard.

The unauthorized switch begins sending BPDUs.

```text
Unauthorized Switch
        │
        │ BPDU
        ▼
Protected Access Port
        │
        ▼
   BPDU Guard
        │
        ▼
🚫 Err-Disabled
```

The port should be placed into an error-disabled state when BPDU Guard detects the unexpected BPDU.

---

# 🧪 Test 3 — Test Root Guard

Connect/configure a switch on the protected Root Guard interface and attempt to introduce a superior BPDU.

The Root Guard-protected interface should not accept the new root information as a valid path.

The interface can enter:

```text
root-inconsistent
```

until the superior BPDU condition is removed.

---

# 🔍 Important Verification Commands

| Purpose                            | Command                                       |
| ---------------------------------- | --------------------------------------------- |
| Show STP information               | `show spanning-tree`                          |
| Show STP summary                   | `show spanning-tree summary`                  |
| Check PortFast                     | `show spanning-tree interface fa0/4 portfast` |
| Detailed STP interface information | `show spanning-tree interface fa0/4 detail`   |
| Check configuration                | `show running-config`                         |
| Check interface status             | `show interfaces status`                      |
| Check trunk links                  | `show interfaces trunk`                       |

---

# 🧰 Troubleshooting

## ❌ Problem 1 — PC Does Not Transition Quickly

Check whether PortFast is enabled:

```cisco
show spanning-tree interface fa0/4 portfast
```

If necessary:

```cisco
interface fa0/4
spanning-tree portfast
```

---

## ❌ Problem 2 — BPDU Guard Is Not Enabled

Check:

```cisco
show running-config interface fa0/4
```

Configure:

```cisco
interface fa0/4
spanning-tree bpduguard enable
```

---

## ❌ Problem 3 — Port Becomes Err-Disabled

Check the interface:

```cisco
show interfaces status
```

If BPDU Guard triggered the condition, remove the unauthorized switch/device first.

The interface can then be manually recovered with:

```cisco
interface fa0/4
shutdown
no shutdown
```

> Only recover the interface after confirming that the unexpected BPDU source has been removed and the port is intended to be an edge port.

---

## ❌ Problem 4 — Root Guard Blocks the Interface

Check STP:

```cisco
show spanning-tree interface fa0/4 detail
```

If the interface is in a root-inconsistent state, investigate which device is sending the superior BPDU.

---

# 🔑 PortFast vs BPDU Guard vs Root Guard

| Feature        | Main Purpose                                 | Typical Location                        |
| -------------- | -------------------------------------------- | --------------------------------------- |
| ⚡ PortFast     | Quickly move edge ports to forwarding        | End-device access ports                 |
| 🛡️ BPDU Guard | Protect PortFast ports from unexpected BPDUs | End-device access ports                 |
| 🔒 Root Guard  | Prevent unexpected root influence            | Selected switch-facing/designated ports |

### Simple Memory Trick

```text
⚡ PortFast
"Make end-device ports fast."

🛡️ BPDU Guard
"Don't let a switch appear here."

🔒 Root Guard
"Don't let this path become an unexpected root path."
```

---

# 🏗️ STP Security Architecture

```text
                       🌳 STP ROOT BRIDGE
                              │
                    ┌─────────┴─────────┐
                    │                   │
                  TRUNK               TRUNK
                    │                   │
                ┌───▼───┐           ┌───▼───┐
                │ SW2   │           │ SW3   │
                └───┬───┘           └───────┘
                    │
              🔒 Root Guard
                    │
                    X
          Unexpected Root Influence


          END-DEVICE ACCESS PORTS
                    │
          ┌─────────┴─────────┐
          │                   │
        PC 1                PC 2
          │                   │
       PortFast            PortFast
          │                   │
     BPDU Guard           BPDU Guard
```

---

# 📊 Security Flow

```text
              STP ATTACK PREVENTION
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
    PortFast       BPDU Guard     Root Guard
        │              │              │
        ▼              ▼              ▼
 Fast Edge        Block BPDUs    Protect Root
  Forwarding       on Edge         Path
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                🛡️ STP HARDENING
```

---

# 🏆 Project Outcome

Successfully implemented an **STP attack prevention and Layer 2 security configuration** using:

* ⚡ PortFast
* 🛡️ BPDU Guard
* 🔒 Root Guard

The lab demonstrated how PortFast improves end-device connectivity speed, how BPDU Guard protects edge ports from unauthorized switches, and how Root Guard helps prevent unexpected STP root influence on protected links.

The configuration was verified using Cisco IOS **STP, interface, and trunk verification commands**.

---

# 🧠 What I Learned

* Spanning Tree Protocol fundamentals
* Root Bridge selection
* STP forwarding and blocking behavior
* BPDU fundamentals
* PortFast configuration
* BPDU Guard configuration
* Root Guard configuration
* Access vs trunk interfaces
* STP security hardening
* Error-disabled interface behavior
* Root-inconsistent STP state
* Cisco IOS STP verification commands

---

# 🛠️ Technologies & Tools

* **Cisco Packet Tracer**
* **Cisco IOS**
* **Spanning Tree Protocol (STP)**
* **PortFast**
* **BPDU Guard**
* **Root Guard**
* **Layer 2 Switching**
* **802.1Q Trunking**

---

# 📌 Project Summary

> **This project demonstrates practical STP security hardening by using PortFast for faster edge-port forwarding, BPDU Guard to protect edge ports from unexpected BPDUs, and Root Guard to prevent unexpected STP root influence.**

---

# 🏷️ Tags

`#Cisco` `#CCNA` `#Networking` `#CiscoPacketTracer` `#STP` `#SpanningTreeProtocol` `#PortFast` `#BPDUGuard` `#RootGuard` `#NetworkSecurity` `#Layer2Security` `#NetworkEngineer` `#ITSupport`

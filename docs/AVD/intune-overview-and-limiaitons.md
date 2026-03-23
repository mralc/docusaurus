---
title: Intune and AVD — An Overview of Capabilities and Limitations
description: A deep dive into how Microsoft Intune interacts with Azure Virtual Desktop, including multi-session hosts and external identities, and what you can do when it falls short.
sidebar_label: Intune Overview & Limitations
tags: [AVD, Intune, Azure Virtual Desktop, Security, Windows 11]
---

# Intune and AVD — An Overview of Capabilities and Limitations

Microsoft Intune is a cloud-based endpoint management solution that lets you enforce security policies, deploy applications, and manage device configurations across your organisation. When combined with **Azure Virtual Desktop (AVD)**, Intune is a natural choice for locking down session hosts — but the reality is more nuanced than it first appears. This post walks through how Intune behaves across different AVD configurations and explores the workarounds available when it falls short.

---

## Intune with Windows 11

On a standard Windows 11 (non-multi-session) session host, Intune behaves largely as expected:

- Intune settings are divided into **Device** settings and **User** settings, both configured via the Settings Catalog.
- Unlike Group Policy, Intune policies can be targeted to **Entra devices** (e.g. an AVD host) or to **Entra users**.
- Device settings require a **device-based Intune license**.
- User settings require a **user-based Intune license** assigned to the user account.
- A user setting targeted at a device will affect all users who sign in — but enforcement only applies to users with a valid Intune user license.
- Policies are assigned via **Entra groups**, which may contain devices or users.

> **Real-world test:** I registered an AVD host with Intune and applied two policies — a device configuration disabling system time changes, and a user configuration blocking access to Control Panel. When a cloud-only user without a user license signed in, the device policy applied (time change was blocked) but the user policy did not. After assigning a user license, the Control Panel restriction applied as expected.

---

## Intune with Windows 11 Multi-Session

Windows 11 Enterprise multi-session allows multiple concurrent users to sign in to the same VM, **Intune support for multi-session VMs is significantly limited**.

According to the [Microsoft documentation](https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/azure-virtual-desktop-multi-session):

> Device-based configuration cannot be assigned to users, and user-based configuration cannot be assigned to devices. Doing so is reported as *Error* or *Not applicable*.

Following a call with Microsoft, it was confirmed that user settings can be configured for multi-session hosts, provided an Intune user license is assigned. However, only a limited subset of settings are supported — for example, blocking access to the C:\ drive is not available. To identify the supported settings, open the Settings Catalog in the Microsoft Intune admin center, select **Settings picker**, then **Add filter**, and apply the following options:

| Field | Value |
|---|---|
| Key | OS edition |
| Operator | == |
| Value | Enterprise multi-session |

After selecting **Apply**, the list is filtered to show only configuration profile categories that support Windows Enterprise multi-session. The scope of each policy is indicated in parentheses: user-scoped policies are labelled **(User)**, and all others are device-scoped.

In addition to the Settings Catalog, Intune also supports a separate set of configuration profile templates. These are distinct from the user settings described above, and only the following templates are fully supported for Windows 11 Enterprise multi-session VMs:
In my testing, device and user restrictions (for example, blocking access to the `C:\` drive) did not apply reliably on multi-session hosts. Policies intermittently failed with *Error* or *Not applicable* — even when configured correctly.

---

## AVD with External Identities

I have been testing external identities for AVD authentication (see the [Microsoft documentation](https://learn.microsoft.com/en-us/azure/virtual-desktop/authentication#external-identity)). As Microsoft states:

> "Intune device configuration policies assigned to an external identity will not be applied to the user on the session host — instead, those policies must be assigned to the device."

**On single-user Windows 11 session hosts**, this behaves as documented: device-targeted settings apply to the host, but user-targeted settings do not apply to external accounts, since a user-based Intune license cannot be assigned to external identities.

**On multi-session hosts**, the results were inconsistent: user-targeted settings did not apply, and several policies reported *Error* or *Not applicable*. The conclusion is clear — user-level controls are unreliable for external identities on multi-session hosts. **Device-targeted policies are the more dependable option.**

---

## How Can We Overcome This?

Given these limitations, there are a few approaches worth considering.

### Third-Party Solutions

I evaluated third-party vendors to address these restrictions. However, multiple vendors confirmed that the constraints are an inherent Windows limitation — their products face the same boundaries and cannot work around them at the OS level.

### Manual Configuration via Registry

Many lockdown settings are ultimately backed by registry keys. To prevent users from reverting them, settings would need to be written to the `HKEY_LOCAL_MACHINE` hive rather than the user hive. Possible deployment methods include:

- **Baked into the VM image** as part of the build process.
- **Applied by a configuration management tool** such as Puppet or DSC.
- **Deployed via an Intune device configuration policy.**

:::caution
Settings written to `HKEY_LOCAL_MACHINE` apply to **all users** on the host, including administrators. This could hinder troubleshooting and needs to be carefully managed.
:::

### Application Allowlisting

Microsoft provides two tools for application allowlisting: **AppLocker** and **App Control for Business**.

#### AppLocker

AppLocker is the traditional allowlisting solution. It works by blocking all executables and DLLs by default, then defining an allowlist based on file paths or file hashes. It is a built-in Windows feature that can be configured locally.

In testing, I configured AppLocker on a multi-session host with external identities using the Local Security Policy, and it **successfully restricted users** to only running approved applications.

However, there are notable drawbacks when managing it through Intune:
- Managing AppLocker through Intune requires manually exporting the XML configuration and importing it into Intune. Every change requires repeating this process, adding operational overhead.
- On a **Windows 11 single-session host**, AppLocker applied reliably — the policy was enforced correctly on the AVD host.
- On a **Windows 11 multi-session host**, AppLocker did **not** apply reliably — the policy failed to apply correctly to the AVD host.

#### App Control for Business

App Control for Business is the modern replacement for AppLocker, deployed via the `applicationControl` CSP (Configuration Service Provider). Key differences from AppLocker:

- It is part of **Microsoft Defender** rather than Intune, making it less affected by the Intune multi-session limitations.
- Policies are applied at the **device level**, affecting all users who sign in.
- It is the **recommended path forward** given the limitations observed with Intune on multi-session hosts.

:::note
Testing is currently in progress — this section will be updated once further results are available.
:::
---

## Summary

| Scenario | Device Policies | User Policies |
|---|---|---|
| Windows 11 (single-session) | ✅ Reliable | ✅ Reliable (with user license) |
| Windows 11 Multi-session | ⚠️ Limited support | ❌ Not supported |
| External Identities (single-session) | ✅ Reliable | ❌ No Intune license support |
| External Identities (multi-session) | ⚠️ Inconsistent | ❌ Unreliable |

For AVD multi-session environments, particularly those using external identities, **device-level controls and application allowlisting** (via App Control for Business) are currently the most viable strategies for enforcing security policies.
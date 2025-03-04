---
title: What's new in Microsoft Defender for IoT for device builders
description: Learn about the latest updates for Defender for IoT device builders.
ms.topic: conceptual
ms.date: 02/20/2022
---

# What's new

[!INCLUDE [Banner for top of topics](../includes/banner.md)]

This article lists new features and feature enhancements in Microsoft Defender for IoT for device builders.

Noted features are in PREVIEW. The [Azure Preview Supplemental Terms](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) include other legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Versioning and support

Listed below are the support, breaking change policies for Defender for IoT, and the versions of Defender for IoT that are currently available.

## February 2022

**Version 4.1.1**:

- **Micro agent for Edge is now in Public Preview**: The micro-agent supports IoT Edge devices, with an easy installation and identity provisioning process that uses an automatically provisioned module identity to authenticate Edge devices without the need to perform any manual authentication.

    For more information, see [Install Defender for IoT micro agent for Edge (Preview)](how-to-install-micro-agent-for-edge.md).

- **New directory structure**: Now aligned with the standard Linux installation directory structure.

    Due to this change, updates to version 4.1.1 require you to reauthenticate the micro agent and save your connection string in the new location. For more information, see [Upgrade the Microsoft Defender for IoT micro agent](upgrade-micro-agent.md).

- **SBoM collector**: The SBoM collector now collects the packages installed on the device periodically. For more information, see [Micro agent event collection (Preview)](concept-event-aggregation.md).

- **CIS benchmarks**:

    - CIS Linux distribution independent benchmarks now supports recommendations based on CIS Distribution Independent Linux Benchmarks, version 2.0.0.

    - CIS benchmark recommendations now support the ability to disable specific CIS Benchmarks checks or groups through twin configurations. For more information, see [Micro agent configurations (Preview)](concept-micro-agent-configuration.md).

- **Micro agent supported devices list expands**: The micro agent now supports Debian 11 AMD64 and ARM32v7 devices, as well as Ubuntu Server 18.04 ARM32 Linux devices & Ubuntu Server 20.04 ARM32 & ARM64 Linux devices.

    For more information, see [Agent portfolio overview and OS support (Preview)](concept-agent-portfolio-overview-os-support.md).

- **DNS hit count**: network collector now includes DNS hit count field that can be visible through Log Analytics, which can help indicate if a DNS request was part of an automatic query.

    For more information, see [Network Connection events (event-based collector)](concept-event-aggregation.md#network-connection-events-event-based-collector).

- **Login Collector**: Now supporting login collector using: SYSLOG collecting SSH login events and PAM collecting SSH, telnet and local login events using the pluggable authentication modules stack. For more information, see [Micro agent event collection (Preview)](concept-event-aggregation.md).


## November 2021

**Version 3.13.1**:

- DNS network activity on managed devices is now supported. Microsoft threat intelligence security graph can now detect suspicious activity based on DNS traffic.

- [Leaf device proxying](../../iot-edge/how-to-connect-downstream-iot-edge-device.md#integrate-microsoft-defender-for-iot-with-iot-edge-gateway): There is now an enhanced integration with IoT Edge. This integration enhances the connectivity between the agent, and the cloud using leaf device proxying.

## October 2021

**Version 3.12.2**:

- More CIS benchmark checks are now supported for Debian 9: These extra checks allow you to make sure your network is compliant with the CIS best practices used to protect against pervasive cyber threats.

- **[Twin configuration](concept-micro-agent-configuration.md)**: The micro agent’s behavior is configured by a set of module twin properties. You can configure the micro agent to best suit your needs.

## September 2021

**Version 3.11**:

- **[Login collector](concept-event-aggregation.md#login-collector-event-based-collector)** - The login collectors gather user logins, logouts, and failed login attempts. Such as SSH & telnet.

- **[System information collector](concept-event-aggregation.md#system-information-trigger-based-collector)** - The system information collector gatherers information related to the device’s operating system and hardware details.

- **[Event aggregation](concept-event-aggregation.md#how-does-event-aggregation-work)** - The Defender for IoT agent aggregates events such as process, login, network events that reduce the number of messages sent and costs, all while maintaining your device's security.  

- **[Twin configuration](concept-micro-agent-configuration.md)** - The micro agent's behavior is configured by a set of module twin properties. (e.g event sending frequency and Aggregation mode). You can configure the micro agent to best suit your needs.

## March 2021

### Device builder - new micro agent (Public preview)

A new device builder module is available. The module, referred to as a micro-agent, allows:

- **Integration with Azure IoT Hub and Defender for IoT** - build stronger endpoint security directly into your IoT devices by integrating it with the monitoring option provided by both the Azure IoT Hub and Defender for IoT.

- **Flexible deployment options with support for standard IoT operating systems** - can be deployed either as a binary package or as modifiable source code, with support for standard IoT operating systems like Linux and Azure RTOS.

- **Minimal resource requirements with no OS kernel dependencies** - small footprint, low CPU consumption, and no OS kernel dependencies.

- **Security posture management** – proactively monitor the security posture of your IoT devices.

- **Continuous, real-time IoT/OT threat detection** - detect threats such as botnets, brute force attempts, crypto miners, and suspicious network activity

The deprecated Defender-IoT-micro-agent documentation will be moved to the *Agent-based solution for device builders>Classic* folder.

This feature set is available with the current public preview cloud release.

## Next steps

[Onboard to Defender for IoT](quickstart-onboard-iot-hub.md)

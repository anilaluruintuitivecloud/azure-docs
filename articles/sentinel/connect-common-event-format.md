---
title: Get CEF-formatted logs from your device or appliance into Azure Sentinel | Microsoft Docs
description: Use the Log Analytics agent, installed on a Linux-based log forwarder, to ingest logs sent in Common Event Format (CEF) over Syslog into your Azure Sentinel workspace.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/26/2021
ms.author: yelevin
ms.custom: ignite-fall-2021
---

# Get CEF-formatted logs from your device or appliance into Azure Sentinel

[!INCLUDE [Banner for top of topics](./includes/banner.md)]

[!INCLUDE [reference-to-feature-availability](includes/reference-to-feature-availability.md)]

Many networking and security devices and appliances send their system logs over the Syslog protocol in a specialized format known as Common Event Format (CEF). This format includes more information than the standard Syslog format, and it presents the information in a parsed key-value arrangement. The Log Analytics Agent accepts CEF logs and formats them especially for use with Azure Sentinel, before forwarding them on to your Azure Sentinel workspace.

This article describes the process of using CEF-formatted logs to connect your data sources. For information about data connectors that use this method, see [Azure Sentinel data connectors reference](data-connectors-reference.md).

There are two main steps to making this connection, that will be explained below in detail:

- Designating a Linux machine or VM as a dedicated log forwarder, installing the Log Analytics agent on it, and configuring the agent to forward the logs to your Azure Sentinel workspace.
The installation and configuration of the agent are handled by a deployment script.

- Configuring your device to send its logs in CEF format to a Syslog server.

> [!NOTE]
> Data is stored in the geographic location of the workspace on which you are running Azure Sentinel.

## Supported architectures

The following diagram describes the setup in the case of a Linux VM in Azure:

 ![CEF in Azure](./media/connect-cef/cef-syslog-azure.png)

Alternatively, you'll use the following setup if you use a VM in another cloud, or an on-premises machine:

 ![CEF on premises](./media/connect-cef/cef-syslog-onprem.png)

## Prerequisites

An Azure Sentinel workspace is required in order to ingest CEF data into Log Analytics.

- You must have read and write permissions on this workspace.

- You must have read permissions to the shared keys for the workspace. [Learn more about workspace keys](../azure-monitor/agents/agent-windows.md).

## Designate a log forwarder and install the Log Analytics agent

This section describes how to designate and configure the Linux machine that will forward the logs from your device to your Azure Sentinel workspace.

Your Linux machine can be a physical or virtual machine in your on-premises environment, an Azure VM, or a VM in another cloud.

Use the link provided on the **Common Event Format (CEF) data connector page** to run a script on the designated machine and perform the following tasks:

- **Installs the Log Analytics agent for Linux** (also known as the OMS agent) and configures it for the following purposes:
    - listening for CEF messages from the built-in Linux Syslog daemon on TCP port 25226
    - sending the messages securely over TLS to your Azure Sentinel workspace, where they are parsed and enriched

- **Configures the built-in Linux Syslog daemon** (rsyslog.d/syslog-ng) for the following purposes:
    - listening for Syslog messages from your security solutions on TCP port 514
    - forwarding only the messages it identifies as CEF to the Log Analytics agent on localhost using TCP port 25226

For more information, see [Deploy a log forwarder to ingest Syslog and CEF logs to Azure Sentinel](connect-log-forwarder.md).

### Security considerations

Make sure to configure the machine's security according to your organization's security policy. For example, you can configure your network to align with your corporate network security policy and change the ports and protocols in the daemon to align with your requirements.

For more information, see [Secure VM in Azure](../virtual-machines/security-policy.md) and [Best practices for Network security](../security/fundamentals/network-best-practices.md).

If your devices are sending Syslog and CEF logs over TLS, such as when your log forwarder is in the cloud, you will need to configure the Syslog daemon (rsyslog or syslog-ng) to communicate in TLS. 

For more information, see:

- [Encrypting Syslog traffic with TLS – rsyslog](https://www.rsyslog.com/doc/v8-stable/tutorials/tls_cert_summary.html)
- [Encrypting log messages with TLS – syslog-ng](https://support.oneidentity.com/technical-documents/syslog-ng-open-source-edition/3.22/administration-guide/60#TOPIC-1209298)

## Configure your device

Locate and follow your device vendor's configuration instructions for sending logs in CEF format to a SIEM or log server. 

If your product appears in the data connectors gallery, you can consult the [Azure Sentinel data connectors reference](data-connectors-reference.md) for assistance, where the configuration instructions should include the settings in the list below.

   - Protocol = TCP
   - Port = 514
   - Format = CEF
   - IP address - make sure to send the CEF messages to the IP address of the virtual machine you dedicated for this purpose.

This solution supports Syslog RFC 3164 or RFC 5424.

> [!TIP]
> Define a different protocol or port number in your device as needed, as long as you also make the same changes in the Syslog daemon on the log forwarder.
>

## Find your data

It may take up to 20 minutes after the connection is made for data to appear in Log Analytics.

To search for CEF events in Log Analytics, query the `CommonSecurityLog` table in the query window.

Some products listed in the data connectors gallery require the use of additional parsers for best results. These parsers are implemented through the use of Kusto functions. For more information, see the section for your product on the [Azure Sentinel data connectors reference](data-connectors-reference.md) page.

To find CEF events for these products, enter the name of the Kusto function as your query subject, instead of "CommonSecurityLog."

You can find helpful sample queries, workbooks, and analytics rule templates made especially for your product on the **Next steps** tab of your product's data connector page in the Azure Sentinel portal.

If you're not seeing any data, see the [CEF troubleshooting](./troubleshooting-cef-syslog.md) page for guidance.

### Changing the source of the TimeGenerated field

By default, the Log Analytics agent populates the *TimeGenerated* field in the schema with the time the agent received the event from the Syslog daemon. As a result, the time at which the event was generated on the source system is not recorded in Azure Sentinel.

You can, however, run the following command, which will download and run the `TimeGenerated.py` script. This script configures the Log Analytics agent to populate the *TimeGenerated* field with the event's original time on its source system, instead of the time it was received by the agent.

```bash
wget -O TimeGenerated.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/TimeGenerated.py && python TimeGenerated.py {ws_id}
```

## Next steps

In this document, you learned how Azure Sentinel collects CEF logs from devices and appliances. To learn more about connecting your product to Azure Sentinel, see the following articles:

- [Deploy a Syslog/CEF forwarder](connect-log-forwarder.md)
- [Azure Sentinel data connectors reference](data-connectors-reference.md)
- [Troubleshoot log forwarder connectivity](troubleshooting-cef-syslog.md#validate-cef-connectivity)

To learn more about what to do with the data you've collected in Azure Sentinel, see the following articles:

- Learn about [CEF and CommonSecurityLog field mapping](cef-name-mapping.md).
- Learn how to [get visibility into your data and potential threats](get-visibility.md).
- Get started [detecting threats with Azure Sentinel](./detect-threats-built-in.md).

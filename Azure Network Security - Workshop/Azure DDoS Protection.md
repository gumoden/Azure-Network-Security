# Module 4 - Azure DDoS Protection

⬅️[Return to the main page](https://github.com/gumoden/Azure-Network-Security/blob/master/Azure%20Network%20Security%20-%20Workshop/README.md)

## Scenarios
- [Verify DDoS Network Protection is associated with a public IP](#verify-ddos-network-protection-is-associated-with-a-public-ip)
- [Use Azure Diagnostic logs and Metrics to analyze Azure DDoS Protection mitigations](#use-azure-diagnostic-logs-and-metrics-to-analyze-azure-ddos-protection-mitigations)
- [Use Microsoft Sentinel to analyze Azure DDoS Protection mitigations](#use-microsoft-sentinel-to-analyze-azure-ddos-protection-mitigations)

## Verify DDoS Network Protection is associated with a public IP

In this scenario, we'll verify that a DDoS Protection plan is associated with our virtual network using Azure Firewall Manager. Once a DDoS protection plan is enabled on a virtual network, all of the Public IP resources within that virtual network are automatically protected by that plan. Additionally, we'll show you how to check if a Public IP is being protected using DDoS IP Protection in the Azure Preview Portal.

1. In the search bar of the Azure Portal, search for **Firewall Manager** and select it. This will bring you to the 'Getting Started' page for Firewall Manager.
2. Once there, select **DDoS Protection Plans** under Security. You should see a protection plan named **ddosPlan-AlpineSkiHouse**, select it.
3. After selecting ddosPlan-AlpineSkiHouse, you should be at the Overview blade for the DDoS Protection Plan. Under Settings, select **Protected resources** so we can identify exactly what resources are protected by this plan.
4. The Protected resources blade will have 8 different tabs at the top, showing the different resource types that are covered by DDoS Network Protection. These are the resource types and expected resource names to be protected:
  - NET: vnet-hub-alpineSkiHouse, vnet-spoke-workload-1, vnet-spoke-workload-2
  !IMAGE[ddos-setup-1.png](instructions281582/ddos-setup-1.png)

  - Firewall: pip-azfw-hub-alpineSkiHouse
  !IMAGE[ddos-setup-2.png](instructions281582/ddos-setup-2.png)

  - Application Gateway: pip-appgw-whmzgkcjeovje-waf
  !IMAGE[ddos-setup-3.png](instructions281582/ddos-setup-3.png)

  - Bastion Host: pip-bastion-whmzgkcjeovje
  !IMAGE[ddos-setup-4.png](instructions281582/ddos-setup-4.png)

  - Load Balancer: None deployed in this lab
  - NIC: None deployed in this lab
  - Virtual Machine Scale Set: None deployed in this lab
  - Virtual Network Gateway: None deployed in this lab

Now let's check on how we can verify if a Public IP is protected using DDoS IP Protection rather than DDoS Network Protection.

1. In the search bar, search for the Application Gateway's Public IP resource, **pip-appgw-whmzgkcjeovje-waf**.
2. In the Overview blade for the Public IP, you'll see a button named **Protect**, select this.
3. Once selected, you'll see a blade appear called **Configure DDoS Protection**. In our environment, we can see that the Protection status shows the Public IP as 'Protected: IP is DDoS protected'. The Protection type shows us that this resource is using the DDoS Network Protection plan from earlier, but if you wanted to use DDoS IP Protection, you can choose IP to utilize the other SKU.
!IMAGE[ddos-setup-5.png](instructions281582/ddos-setup-5.png)
!IMAGE[ddos-setup-6.png](instructions281582/ddos-setup-6.png)

Now that we've verified that our resources are protected with a DDoS Protection Plan, let's move to the next module.

**You've reached the end of this scenario**

⬅️ [Go to the top](#scenarios)

## Use Azure Diagnostic logs and Metrics to analyze Azure DDoS Protection mitigations

In this scenario, we'll first verify that diagnostic settings are enabled on the Public IP resources to ensure that we can see metrics and logs when a resource is under attack. We'll then show you how to determine if a resource is under attack, how to find the current threshold values as well as live traffic values with Metrics. After, we'll demonstrate how to use the Kusto queries to investigate a DDoS attack.
1. In the search bar, search for the Application Gateway's Public IP resource, **pip-appgw-whmzgkcjeovje-waf**.
2. Once selected, navigate to **Diagnostic settings** under 'Monitoring'. We should see a Diagnostic setting named **appgwPip-diag**. Select 'Edit setting' to view more.
3. Inside the Diagnostic setting, under 'Logs', we can see 3 category logs selected.
  - DDoS protection notifications (DDoSProtectionNotifications)
  - Flow logs of DDoS mitigation decisions (DDoSMitigationFlowLogs)
  - Reports of DDoS mitigations (DDoSMitigationReports)
4. Under 'Destination details', we see that the logs are being sent to a Log Analytics workspace named '**CyberSOC**'.
5. Metrics are enabled and visible by default. You do not need to send these to a Log Analytics workspace to view metrics.

Let's quickly touch on what kind of logs are generated for each of the log categories.
1. **DDoSProtectionNotifications**: This log will generate two messages per attack, one with the message 'Start DDOS Mitigation', to let you know when an attack has been identified by the DDoS protection service. The other with the message 'Stop DDOS Mitigation', to let you know when an active DDoS attack has ended. 
2. **DDoSMitigationFlowLogs**: These logs allow you to review the dropped traffic, forwarded traffic and other attack data in near real-time during an active DDoS attack. This will generate a large number of logs as it will show you the action taken per network flow received.
3. **DDoSMitigationReports**: Two types of logs will be generated under this category; Incremental reports, which is generated every 5 minutes when the resource is under an active DDoS attack, and Post mitigation reports, which is generated for the entire duration of the DDoS attack once the attack has ended. Both logs identify information like attack vectors, traffic statistics, protocols involved, top 10 source countries or ASN, and drop reason.

### Metrics
Now we'll explore the metrics to determine the mitigation thresholds and identify if a public IP is under attack.
1. Select Metrics under 'Monitoring' to view Metrics.
2. In the chart, click on the drop-down under Metric and select **Under DDoS attack or not**. You may need to change the time range to Last 30 days to find information, unless you just initiated an attack. The 'Under DDoS attack or not' metric has a value of either 0 or 1, 0 indicating that the resource is not under attack and 1 indicating that the resource is under attack.
3. Select + New chart on the top left and select the Metrics below. You'll have to click 'Add metric' after each selection to add them all to the same chart. These are the current threshold values of this particular resource. All resources can have different threshold values based off their unique traffic patterns.
  - Inbound SYN packets to trigger DDoS mitigation (10k/pps)
  - Inbound TCP packets to trigger DDoS mitigation (50k/pps)
  - Inbound UDP packets to trigger DDoS mitigation (40k/pps)
4. Select + New chart again and select the Metrics below. We'll see that the Inbound packets greatly surpassed the threshold value for this specific resource, forcing DDoS mitigation upon the resource.
  - Inbound SYN packets to trigger DDoS mitigation
  - Inbound packets DDoS
  !IMAGE[ddos-logs-metrics-1.png](instructions281582/ddos-logs-metrics-1.png)
  !IMAGE[ddos-logs-metrics-2.png](instructions281582/ddos-logs-metrics-2.png)

### Logs
Finally, let's explore the logs and get additional details of any DDoS attack that may have been mitigated by Azure DDoS Protection.
1. In the search bar, search for the Log Analytics workspace, **CyberSOC**.
2. Once selected, navigate to **Logs** under 'General' and close the Queries pop up window.
3. Next, click on **Queries** and type 'ddos' in the filter. You should see a query called **DDoSQueries** under 'Security'. Hover over this and select 'Load to editor'. Our 4 queries should automatically populate into the editor window.
4. Click on 'DDoSProtectionNotifications' in the editor window to highlight the entire query and click 'Run'. If you recently triggered an attack, you should you Start and Stop mitigation logs below, if not, you may have to customize the time range to find the most recent attack triggered.
5. Click on 'DDoSMitigationReports' in the editor window to highlight the entire query and click 'Run'. Look for a report type of Incremental to get familiar with how a 5-minute aggregated report looks like. Then look for a report type of Post mitigation to get familiar with how a total time aggregated report looks like.
6. Click on 'DDoSMitigationFlowLogs' in the editor window to highlight the entire query and click 'Run'. This particular query will generate thousands of logs and may show you only the first 30,000. Select any log and familiarize yourself how flows are analyzed, and actions chosen based off of packet details.
!IMAGE[ddos-logs-metrics-3.png](instructions281582/ddos-logs-metrics-3.png)

### Kusto Queries
1. **DDoS Protection Notifications**

```kql
AzureDiagnostics
| where Category == "DDoSProtectionNotifications"
```

2. **DDoS Protection Reports - Post Mitigation**

```kql
AzureDiagnostics
| where Category == "DDoSMitigationReports"
| where ReportType_s == "Post mitigation"
```

3. **DDoS Protection Reports - Incremental**

```kql
AzureDiagnostics
| where Category == "DDoSMitigationReports"
| where ReportType_s == "Incremental"
```

4. **DDoS Mitigation Flow Logs**

```kql
AzureDiagnostics
| where Category == "DDoSMitigationFlowLogs"
```

**You've reached the end of this scenario**

⬅️ [Go to the top](#scenarios)

# Use Microsoft Sentinel to analyze Azure DDoS Protection mitigations

In this scenario, we'll look at how to use Microsoft Sentinel to analyze a DDoS attack against your environment using a Workbook.

1. In the search bar, search for 'Sentinel' and select **Microsoft Sentinel**.
2. Select the Sentinel workspace, **CyberSOC**.
3. Once selected, click on **Workbooks** under 'Threat Management'.
4. By default, we should be in the 'Templates' tab. In the search bar under 'Templates', search for 'ddos' and select the **Azure DDoS Protection Workbook**. Then select 'View saved workbook' in the bottom right.
5. The Azure DDoS Protection Workbook has 3 tabs to help investigate a DDoS attack. These are:
  - **DDoS Summary** - Overview of the protocols, origin data, AS Numbers, and Drop Reasons for the selected DDoS attack.
  - **DDoS Metrics** - Shows Metrics values of the Public IP address that was under attack.
  - **DDoS Investigation** - This page allows you to dive deeper into specific Kusto queries for a specified attack.
6. When you first open the Workbook, no workspace will be selected. Click on the drop-down and select **CyberSOC**. Change the time range to an appropriate time and you can either leave Public IP Addresses set to 'All' or you can select only **pip-appgw-whmzgkcjeovje-waf**.

Now, let's explore all the tabs in this Workbook, and find more information about the attacks mitigated.

**DDoS Summary**

This section provides a summary of the DDoS attacks mitigated by Azure DDoS Protection. Check below for more details:
  - **Traffic Overview**: Here you'll find comprehensive details on the total number of packets and the various categories of dropped packets during the DDoS attacks for the timeline defined.
  - **Last Ten DDoS Attack Reports**: This section provides the details of attack reports, resources affected, attack vectors and packet information.
  - **Location and Protocol details**: This section provides categorized details on the protocols involved in the DDoS attacks, the origins of these attacks, and the protocol violations that occurred during past DDoS incidents.
  - **Raw DDoS Mitigation and Flow Logs**: Furthermore, if we would like to take a look at the Raw DDoS Logs those are also available as part of the workbook so that we do not have to look for them in the log analytics workspace.
!IMAGE[ddos-sentinel-1.png](instructions281582/ddos-sentinel-1.png)
!IMAGE[ddos-sentinel-2.png](instructions281582/ddos-sentinel-2.png)

### DDoS Metrics

The DDoS Metrics tab provides graphical representation of all the important metrics like Packet count, Syn packet thresholds to trigger DDoS mitigation, inbound DDoS TCP/UDP packets, and Under DDoS attack or not as shown below. Most of the metrics here are based on number of Packets Per Second (PPS) and Packets/Byte Counts.
!IMAGE[ddos-sentinel-3.png](instructions281582/ddos-sentinel-3.png)

### DDoS Investigation

The Investigation Tab in the workbook offers specific details on the number of packets that were dropped or allowed during past DDoS attacks, including the ports involved. Additionally, this tab provides information on the top attacking IPs and the timeline of the mitigation activities, as illustrated below.
!IMAGE[ddos-sentinel-4.png](instructions281582/ddos-sentinel-4.png)

**You've reached the end of this scenario**

⬅️ [Go to the top](#scenarios)

## Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions provided by the bot. You will only need to do this once across all repos using our CLA.

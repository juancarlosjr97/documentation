---
title: Cloud Network Monitoring
description: Explore metrics for point to point communication on your infrastructure.
aliases:
  - /monitors/network_flow_monitors/
  - /graphing/infrastructure/network_performance_monitor/
  - /network_performance_monitoring/
  - /network_monitoring/performance/
further_reading:
- link: '/network_monitoring/cloud_network_monitoring/guide/detecting_application_availability/'
  tag: 'Guide'
  text: 'Detecting Application Availability using Network Insights'
- link: "https://www.datadoghq.com/blog/npm-windows-support/"
  tag: "Blog"
  text: "Monitor Windows hosts with Cloud Network Monitoring"
- link: "https://www.datadoghq.com/blog/cloud-service-autodetection-datadog/"
  tag: "Blog"
  text: "Monitor cloud endpoint health with cloud service autodetection"
- link: "https://www.datadoghq.com/blog/npm-best-practices/"
  tag: "Blog"
  text: "Best practices for getting started with Datadog CNM"
- link: "https://www.datadoghq.com/blog/monitor-consul-with-datadog-npm/"
  tag: "Blog"
  text: "Datadog CNM now supports Consul networking"
- link: "https://www.datadoghq.com/blog/npm-story-centric-ux/"
  tag: "Blog"
  text: "Quickstart network investigations with CNM's story-centric UX"
- link: "https://www.datadoghq.com/blog/monitor-connection-churn-datadog/"
  tag: "Blog"
  text: "Best practices for monitoring and remediating connection churn"
algolia:
  tags: ['Cloud Network Monitoring', 'Network Performance Monitoring', 'CNM', 'NPM']
---

## Overview

{{< vimeo url="https://player.vimeo.com/progressive_redirect/playback/670228207/rendition/1080p/file.mp4?loc=external&signature=42d4a7322017fffa6d5cc2e49ddbb7cfc4c6bbbbf207d13a5c9830630bda4ece" poster="/images/poster/npm.png" >}}

Datadog Cloud Network Monitoring (CNM) gives you visibility into your network traffic between services, containers, availability zones, and any other tag in Datadog. Connection data at the IP, port, and PID levels is aggregated into application-layer dependencies between meaningful client and server endpoints, which can be analyzed and visualized through a customizable [network page][1] and [network map][2]. Use flow data along with key network traffic and DNS server metrics to:

* Pinpoint unexpected or latent service dependencies
* Optimize costly cross-regional or multi-cloud communication
* Identify outages of cloud provider regions and third-party tools
* Troubleshoot client-side and server-side DNS server issues

CNM makes it simple to monitor complex networks with built in support for Linux and [Windows OS][3] as well as containerized environments that are orchestrated and [instrumented with Istio service mesh][4].

Additionally, [Network path][5], a feature of CNM, is available in Preview, which allows you to see hop-by-hop traffic in your network.

{{< whatsnext desc="This section includes the following topics:">}}
    {{< nextlink href="network_monitoring/cloud_network_monitoring/setup" >}}<u>Setup</u>: Configure the Agent to collect network data.{{< /nextlink >}}
    {{< nextlink href="network_monitoring/cloud_network_monitoring/network_analytics" >}}<u>Network Analytics</u>: Graph your network data between each client and server available.{{< /nextlink >}}
    {{< nextlink href="network_monitoring/cloud_network_monitoring/network_map" >}}<u>Network Map</u>: Map your network data between your tags.{{< /nextlink >}}
    {{< nextlink href="monitors/types/cloud_network_monitoring/" >}}<u>Recommended Monitors</u>: Configure recommended CNM monitors.{{< /nextlink >}}
{{< /whatsnext >}}

## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: https://app.datadoghq.com/network
[2]: https://app.datadoghq.com/network/map
[3]: https://www.datadoghq.com/blog/npm-windows-support/
[4]: https://www.datadoghq.com/blog/monitor-istio-with-npm/
[5]: /network_monitoring/network_path/

---
title: Getting Started with Cloud Security Management
aliases:
- /getting_started/cloud_security_management
further_reading:
- link: "/security/cloud_security_management/"
  tag: "Documentation"
  text: "Cloud Security Management"
- link: "/infrastructure/resource_catalog/schema/"
  tag: "Documentation"
  text: "Cloud Resources Schema Reference"
- link: "https://www.datadoghq.com/blog/automate-end-to-end-processes-with-datadog-workflows/"
  tag: "Blog"
  text: "Automate end-to-end processes with Datadog Workflows"
- link: "https://www.datadoghq.com/blog/csm-at-datadog/"
  tag: "Blog"
  text: "How we use Datadog CSM to improve security posture in our cloud infrastructure"
- link: "https://www.datadoghq.com/blog/detecting-leaked-credentials/"
  tag: "Blog"
  text: "How we detect and notify users about leaked Datadog credentials"
- link: "https://www.datadoghq.com/blog/security-posture-csm/"
  tag: "Blog"
  text: "Report on changes to your security posture with Cloud Security Management"
- link: "https://www.datadoghq.com/blog/agentless-scanning/"
  tag: "Blog"
  text: "Detect vulnerabilities in minutes with Agentless Scanning for Cloud Security Management"
- link: "https://dtdg.co/fe"
  tag: "Foundation Enablement"
  text: "Join an interactive session to elevate your security and threat detection"
- link: "https://securitylabs.datadoghq.com/"
  tag: "Security Labs"
  text: "Security research, reports, tips, and videos from Datadog"
---

## Overview

[Datadog Cloud Security Management][1] (CSM) delivers deep visibility, continuous configuration audits, identity risk assessments, vulnerability detection, and real-time threat detection across your entire cloud infrastructure—all in a unified platform for seamless collaboration and faster remediation.

With CSM, Security and DevOps teams can act on the shared context of observability and security data to quickly prioritize and remediate issues. This guide walks you through best practices for getting your team up and running with CSM.

## Phase 1: Deployment

1. Using [Agentless][34] and/or the [Datadog Agent (version 7.46 or above)][4], [enable CSM for your cloud resources and infrastructure][5]:
    - **[Threats][3]**: Kubernetes, Docker, and host-based installations.
    - **[Misconfigurations][2]**: AWS, Azure, GCP, Kubernetes, and Docker instructions.
    - **[Identity Risks][28]**: Enable AWS resource collection and Cloudtrail logs forwarding.
    - **[Vulnerabilities][6]**: Container image scanning and host scanning instructions for AWS, Azure, Kubernetes, ECS EC2 instances, and host-based installations.
1. Check out the [CSM homepage][13] to get an overview of your organization's risks and threats.
1. Review [500+ out-of-the-box Threats and Misconfigurations detection rules][14].
1. Explore [security signals][15] and review [CSM Misconfigurations findings][16].
1. Review and remediate identity risks on the [Identity Risks][29] page.
1. Review container vulnerabilities on the [Container Images][25] page, and a consolidated list of vulnerabilities on the [Infrastructure Vulnerability][30] page.
1. Set up [notification rules][17] and receive alerts using Slack, Jira, email, and more.

## Phase 2: Customization

1. Set up [CSM Threats suppression rules][18] to reduce noise.
2. Create custom detection rules for [CSM Misconfigurations][19] and [CSM Threats][20].

## Phase 3: Reports and dashboards

1. Assess your organization's posture by reviewing [compliance reports][21].
2. Use out-of-the-box dashboards or [create your own][22] for faster investigations, reporting, and monitoring.
3. Subscribe to the weekly [security digest][31] reports to begin investigation and remediation of the most important new security issues discovered in the last seven days. 

## Disable CSM

For information on disabling CSM, see the following:

- [Disable CSM Vulnerabilities][32]
- [Disable CSM Threats][33]

## Further reading

{{< partial name="whats-next/whats-next.html" >}}

[1]: /security/cloud_security_management/
[2]: /security/cloud_security_management/misconfigurations/
[3]: /security/threats/
[4]: https://app.datadoghq.com/account/settings/agent/latest
[5]: /security/cloud_security_management/setup
[6]: /security/cloud_security_management/vulnerabilities/
[13]: https://app.datadoghq.com/security/csm
[14]: /security/default_rules/#cat-cloud-security-management
[15]: /security/cloud_security_management/misconfigurations/signals_explorer/
[16]: /security/cloud_security_management/misconfigurations/findings/
[17]: https://app.datadoghq.com/security/configuration/notification-rules
[18]: /security/cloud_security_management/guide/tuning-rules/
[19]: /security/cloud_security_management/misconfigurations/custom_rules
[20]: /security/threats/agent_expressions
[21]: /security/cloud_security_management/misconfigurations/frameworks_and_benchmarks
[22]: /dashboards/#overview
[25]: https://app.datadoghq.com/containers/images
[26]: /integrations/amazon_web_services/?tab=roledelegation#cloud-security-posture-management
[27]: /integrations/amazon_cloudtrail/#send-logs-to-datadog
[28]: /security/cloud_security_management/identity_risks/
[29]: https://app.datadoghq.com/security/identities
[30]: https://app.datadoghq.com/security/infra-vulnerability
[31]: https://app.datadoghq.com/security/configuration/reports
[32]: /security/cloud_security_management/troubleshooting/vulnerabilities/#disable-csm-vulnerabilities
[33]: /security/cloud_security_management/troubleshooting/threats/#disable-csm-threats
[34]: /security/cloud_security_management/setup/cloud_integrations
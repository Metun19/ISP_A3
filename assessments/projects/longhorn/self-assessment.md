

# Self-assessment
This is the Longhorn's Self-assessment documentation that we wrote after we began thinking about the security of the project, determining gaps in its security, and preparing security documentation for Langhorn's users. This project is currently in the CNCF Incubation phase preparing for Graduation.

# Self-assessment outline

## Table of contents

* [Metadata](#metadata)
  * [Security links](#security-links)
* [Overview](#overview)
  * [Actors](#actors)
  * [Actions](#actions)
  * [Background](#background)
  * [Goals](#goals)
  * [Non-goals](#non-goals)
* [Self-assessment use](#self-assessment-use)
* [Security functions and features](#security-functions-and-features)
* [Project compliance](#project-compliance)
* [Secure development practices](#secure-development-practices)
* [Security issue resolution](#security-issue-resolution)
* [Appendix](#appendix)

## Metadata

A table at the top for quick reference information, later used for indexing.

|   |  |
| -- | -- |
| Software | [A link to the Longhorn’s repository.](https://github.com/longhorn/longhorn) |
| Security Provider | No |
| Languages | Python, Shell, Mustache |
| SBOM | https://longhorn.io/docs/1.5.3/deploy/install/ |
| | |

### Security links

Provide the list of links to existing security documentation for the project. You may
use the table below as an example:
| Doc | url |
| -- | -- |
| Security file | https://longhorn.io/docs/1.5.3/advanced-resources/security/ |
| Default and optional configs (Volume Encryption) | https://longhorn.io/docs/1.5.3/advanced-resources/security/volume-encryption/ |
| MTLS Support | https://longhorn.io/docs/1.5.3/advanced-resources/security/mtls-support/ |

## Overview
Longhorn is a cloud-native, distributed block storage system for Kubernetes. It provides highly available and resilient storage, supporting operations like snapshotting, backup, and disaster recovery.


### Background
Longhorn is a cloud-native distributed block storage system designed to run on Kubernetes. It is particularly engineered to provide persistent storage for stateful applications running in Kubernetes clusters. In cloud-native architectures, persistence, resilience, and scalability are critical challenges due to the ephemeral nature of containerised workloads. Longhorn addresses these challenges by turning distributed block storage into a micro-service, managed by Kubernetes, making it highly available and resilient.


<p align="center"><img width="600" alt="image" src="https://github.com/Makesh-Srinivasan/ISP_A3/assets/66047630/8e9e94c2-b507-474e-8bd1-7773a62344e7"></p>


* `Pods and Volumes`: Longhorn integrates with Kubernetes through volumes presented to pods. These volumes are dynamically provisioned by Longhorn as per the demands of the pods, which could be running various workloads such as databases, CMS, or any stateful application.
* `Longhorn Engine`: Each volume is managed by a dedicated Longhorn Engine, which is responsible for replicating and synchronising data across multiple nodes in the cluster. This dedicated engine approach ensures that the failure of one volume does not impact the function of others.
* `Replicas`: The data for each volume is replicated across multiple replicas spread over different nodes, as shown in the image. This design enhances data protection and availability. If one node fails, the data is still accessible from other replicas.
* `Nodes and Disks`: Longhorn is designed to work with the underlying infrastructure such as in SSDs. As depicted, Longhorn can manage multiple disks across nodes, optimising the utilisation of available storage resources within the cluster.
* `Data Flow`: The image illustrates how read and write operations are conducted across the system. Data written to a Kubernetes volume is managed by the Longhorn Engine and then replicated to the replicas. These replicas handle the actual storage of data on the physical disks. This multi-layered approach allows for a flexible and resilient system that can withstand various failure scenarios.

Longhorn's placement within the cloud-native ecosystem is characterised by its focus on simplicity, reliability, and ease of use. It provides a user-friendly interface, automated volume and snapshot management, backup and restore capabilities, and disaster recovery solutions, all of which are crucial for managing stateful applications in a Kubernetes environment.


### Actors
* `Longhorn Manager`: Manages API calls, volume creation, and overall orchestration within Kubernetes. It communicates with the Kubernetes API server and orchestrates the Longhorn Engine and replicas.
* `Longhorn Engine`: A storage controller responsible for synchronously replicating volumes across multiple nodes. It runs on the same node as the pod using the Longhorn volume, thereby isolating the replication process to specific nodes and limiting lateral movement in case of a compromise.
* `Users/Administrators`: Interact with Longhorn through Kubernetes or the Longhorn UI. This actor is distinct in terms of access control and permissions, which are essential for maintaining the security and integrity of the system.


### Actions
* `Volume Provisioning and Attachment`:
    * Security Checks: Authenticate and authorise Kubernetes volume requests through the Kubernetes RBAC system before provisioning.
    * Data Handling: Provisioning of a volume is done with consideration to data placement policies and storage resource availability on the nodes.
    * Actor Interactions: The Longhorn Manager communicates with Kubernetes to create a Persistent Volume (PV) and Persistent Volume Claim (PVC) and attaches the volume to the appropriate Pod through the Longhorn Engine.
* `Data Replication`:
    * Security Checks: Replication actions are triggered by the Longhorn Engine after ensuring the volume is in a healthy state and authorised for replication.
    * Data Handling: Data is written to the primary volume and then replicated across multiple replicas to ensure redundancy. The integrity of data during replication is maintained by checksum verification.
    * Actor Interactions: The Longhorn Engine orchestrates the replication process across different replicas residing on SSDs in various nodes, ensuring data consistency and handling any replication failures.
* `Snapshot and Backup Operations`:
    * Security Checks: Verify that the request for snapshots or backups comes from an authenticated and authorised source.
    * Data Handling: Snapshots are taken at a consistent state of the volume, and backups are securely transmitted to and from integrated external backup services like NFS or AWS S3, with encryption where applicable.
    * Actor Interactions: Longhorn interacts with the external backup services to store and manage backups, providing an additional layer of data protection and recovery options.
* `Volume Restoration and Recovery`:
    * Security Checks: Validate the authenticity and integrity of backup data before initiating a volume restoration.
    * Data Handling: Restores volume data from snapshots or backups accurately, ensuring no data corruption occurs during the process.
    * Actor Interactions: The Longhorn system manages the recovery process, coordinating between the backup services and the Kubernetes cluster to restore data to the respective Pods and volumes efficiently.


### Goals
* Data Security and Integrity: Ensuring that data stored and managed by Longhorn is protected from unauthorised access and corruption. This includes securing data at rest (via encryption if configured) and in transit across the network.
* Resilience and Availability: Longhorn is designed to maintain high availability and data durability across Kubernetes clusters. This involves replicating data across multiple nodes and ensuring that the failure of a single component does not lead to data loss.
* Secure Data Operations: Providing secure mechanisms for creating, managing, and restoring volumes, with robust authentication and authorisation checks to ensure that only authorised users and systems can perform these operations.
* Disaster Recovery: Offering secure and reliable disaster recovery solutions, enabling quick and consistent data recovery in case of catastrophic events, thereby safeguarding against data loss.
* Compliance with Kubernetes Security Standards: Ensuring that Longhorn's architecture and operations adhere to Kubernetes security best practices, including integration with Kubernetes' Role-Based Access Control (RBAC) for managing access to its resources.


### Non-goals
* Primary Database Services: Longhorn is not intended to serve as a primary database or a data processing system. It is a block storage system for Kubernetes and does not handle database management functions.
* Data Analytics and Processing: Longhorn is not designed for direct involvement in data analytics or processing tasks. Its role is confined to the storage and retrieval of data for such operations, not processing or analysing the data itself.
* Limitless Data Storage: While Longhorn facilitates data storage, it is not designed to support unlimited data storage or handle scenarios where the volume of data could potentially overwhelm the storage capacity or incur excessive costs.
* Network Infrastructure Management: Longhorn's scope does not include the management or configuration of underlying network infrastructure. It focuses on the storage aspect within the given network environment.
* Comprehensive Data Security: While Longhorn implements measures to secure data, it does not offer a complete data security solution. Users are responsible for implementing additional security measures according to their specific requirements, such as network security or end-to-end encryption of sensitive data.


## Self-assessment use

This section serves to inform Longhorn users and contributors about its security practices, as well as to assist CNCF TAG-Security in their joint assessment for incubation phase projects. Firstly, we explore the threats using the STRIDE model. Then, we explore using the lightweight threat modelling. Finally, we see an example attack tree. These steps were helpful in creating the subsequent sections in our assessment.

<details>

<summary>STRIDE Threat Model for Longhorn</summary>

### 1. Introduction
This document provides a STRIDE-based threat model analysis for Longhorn, a cloud-native distributed block storage system designed for Kubernetes. The purpose is to identify potential security threats and suggest measures to mitigate them.

### 2. System Overview
Longhorn offers cloud-native, distributed block storage capabilities, integrating seamlessly with Kubernetes. It manages volumes, snapshots, backups, and ensures data resilience and availability.

#### 2.1 Components
- **Longhorn Manager**
- **Longhorn Engine**
- **Replicas**
- **Nodes and Disks**

#### 2.2 Actors
- **Longhorn Manager**
- **Longhorn Engine**
- **Users/Administrators**

#### 2.3 Actions
- **Volume Provisioning and Attachment**
- **Data Replication**
- **Snapshot and Backup Operations**
- **Volume Restoration and Recovery**

### 3. STRIDE Analysis

#### 3.1 Spoofing
**Threats**:
- Unauthorised access to the Longhorn management interface or API.
- Impersonation of Longhorn components or services.

**Mitigations**:
- Implement strong authentication mechanisms for API and management interface access.
- Use mutual TLS (mTLS) for internal communications.

#### 3.2 Tampering
**Threats**:
- Unauthorised modifications to data in transit or at rest.
- Tampering with Longhorn configuration or codebase.

**Mitigations**:
- Enable data encryption at rest and in transit.
- Ensure data integrity through checksums and replication verification.
- Employ strict access controls and code signing for codebase and configurations.

#### 3.3 Repudiation
**Threats**:
- Denial of actions performed by users or internal processes.
- Lack of auditing trails for critical operations.

**Mitigations**:
- Implement comprehensive logging and auditing mechanisms.
- Ensure that all critical actions are traceable to specific users or entities.

#### 3.4 Information Disclosure
**Threats**:
- Unauthorised access to sensitive data stored in Longhorn volumes.
- Exposure of internal configuration or metadata.

**Mitigations**:
- Enforce strict access controls to data volumes.
- Use encryption to protect sensitive data.
- Restrict access to internal metadata and configuration details.

#### 3.5 Denial of Service (DoS)
**Threats**:
- Overloading the Longhorn system, leading to unavailability.
- Exploiting vulnerabilities to crash the system or degrade performance.

**Mitigations**:
- Implement rate limiting and access controls.
- Design for high availability and resilience.
- Regularly update and patch to address known vulnerabilities.

#### 3.6 Elevation of Privilege
**Threats**:
- Exploitation of vulnerabilities to gain higher privileges.
- Unauthorised access leading to control over Longhorn operations.

**Mitigations**:
- Adhere to the principle of least privilege in access controls.
- Regular security assessments and penetration testing.
- Monitor and promptly update software components to address vulnerabilities.

### 4. Conclusion
This STRIDE threat model for Longhorn identifies key areas of potential security risks and provides a foundation for implementing effective security measures. Regular updates, vigilant monitoring, and adherence to security best practices are essential to maintain the security and integrity of the Longhorn system.

</details>


<details>
 
<summary>Longhorn Lightweight Threat Model</summary>

### Overview

Project: Longhorn (https://github.com/longhorn/longhorn)
Intended usage: Cloud-native distributed block storage system for Kubernetes.
Project data classification: Sensitive
Highest risk impact: Cluster breach, pod breach
Owner(s) and/or maintainer(s):
- Sheng Yang, <sheng@yasker.org>, @yasker
- Shuo Wu, <shuo.wu@suse.com>, @shuo-wu
- David Ko, <dko@suse.com>, @innobead
- Derek Su, <derek.su@suse.com>, @derekbit
- Phan Le, <phan.le@suse.com>, @PhanLe1010


### Threat Modelling Notes

Longhorn operates as a persistent volume provider for Kubernetes, managing the storage lifecycle, replication, and backup of data across a distributed environment. The system's architecture, as visualised in the provided image, depicts data flow between Kubernetes pods, the Longhorn Engine, and storage replicas across different nodes, using secure communication protocols and access controls.

### Data Dictionary

- **Volume Data**
  - Classification/Sensitivity: High
  - Comments: Contains user and application data, potentially including secrets.
- **Replica Data**
  - Classification/Sensitivity: High
  - Comments: Replicated data across nodes for high availability.
- **Snapshot and Backup Data**
  - Classification/Sensitivity: High
  - Comments: Stored both locally and externally (e.g., S3/NFS), contains historical data states.

### Control Families

#### Deployment Architecture
- **Control**: Managed through Kubernetes with defined storage classes and volume claims.
- **Data**: Persistent volume data.
- **Threats**: Misconfiguration leading to unauthorised access.

#### Networking
- **Control**: Internal Kubernetes networking with CNI, optional TLS encryption for external communication.
- **Data**: Control plane and data plane traffic.
- **Threats**: Data interception, man-in-the-middle attacks.

#### Multi-tenancy Isolation
- **Control**: Kubernetes namespaces and RBAC.
- **Data**: Volume provisioning and management operations.
- **Threats**: Cross-tenant access or attacks.

#### Secrets Management
- **Control**: Integration with Kubernetes Secrets for sensitive information management.
- **Data**: Encryption keys, credentials.
- **Threats**: Exposure of sensitive data.

#### Storage
- **Control**: Replicas across diverse storage media (SSDs, HDDs).
- **Data**: User and system data.
- **Threats**: Data loss or corruption.

#### Authentication and Authorisation
- **Control**: Kubernetes RBAC and Service Accounts.
- **Data**: API requests and responses.
- **Threats**: Unauthorised actions within the storage system.

#### Audit and logging
- **Control**: Kubernetes-native logging, Longhorn UI logs.
- **Data**: Operational logs.
- **Threats**: Lack of visibility or forensics in case of an incident.

#### Security Tests
- **Control**: Automated security scanning in CI/CD pipelines.
- **Data**: Code and dependencies.
- **Threats**: Introduction of vulnerabilities.

### Threat Scenarios

#### An External Attacker without access
- **Threats**: Probing for vulnerabilities in exposed interfaces, performing DDoS attacks.
- **Controls**: Firewall rules, rate limiting, and robust authentication mechanisms.

#### An External Attacker with valid access
- **Threats**: Abuse of valid credentials to perform malicious actions.
- **Controls**: Strict RBAC policies, anomaly detection for unusual activities.

#### An Internal Attacker with access to the hosting environment
- **Threats**: Insider threats with access to the cluster might misuse their privileges.
- **Controls**: Audit logging, regular access reviews, and principle of least privilege.

#### A Malicious Internal User
- **Threats**: Introducing backdoors or vulnerabilities.
- **Controls**: Code review processes, restricted merge privileges, and signed commits.

### Potential Controls Summary

| Threat                         | Description                                  | Controls                                                               | References  |
|--------------------------------|----------------------------------------------|------------------------------------------------------------------------|-------------|
| Deployment Architecture        | Misconfigurations leading to data breaches   | Use secure defaults in storage classes, enforce pod security policies   | [Kubernetes Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/) |
| Networking                     | Data interception or manipulation           | Implement network policies, use encrypted communication (TLS)           | [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/) |
| Cryptography                   | Unauthorized data access and tampering       | Enable volume encryption, TLS for data in transit                       | [Encrypting Data at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) |
| Multi-tenancy Isolation        | Cross-tenant access or attacks               | Namespace isolation, fine-grained RBAC                                 | [Kubernetes Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) |
| Secrets Management             | Exposure of sensitive data                   | Use Kubernetes Secrets with appropriate access controls                | [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) |
| Storage                        | Data loss or corruption                      | Replication across nodes, regular snapshots, and backups               | [Longhorn Documentation](https://longhorn.io/docs/) |
| Authentication                 | Unauthorized system access                   | Strong authentication mechanisms, integration with Kubernetes auth      | [Kubernetes Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) |
| Authorization (Access Control) | Unauthorized actions within the storage system| Strict RBAC policies, least privilege access                           | [Kubernetes Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/) |
| Audit and Logging              | Lack of incident visibility or forensics     | Enable detailed logging, integrate with monitoring tools               | [Kubernetes Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/) |


### Recommendations

- Regularly update and patch Longhorn and Kubernetes components.
- Ensure that all communication is encrypted, both in transit and at rest.
- Conduct periodic access reviews and privilege audits.


### Conclusion

- Critical findings should be promptly disclosed to the community.
- Consideration for additional threat modelling tools, such as attack trees or matrices, to further analyse complex threats.
</details>

The attact tree for a generic high-level threat scenario is depicted:

<p align="center"><img alt="image" src="https://github.com/Makesh-Srinivasan/ISP_A3/assets/66047630/4f88f24c-14e9-40c1-92e5-ea3761806556"></p>

Each branch of this tree represents a step or method an attacker might use to progress their attack. The end goal is unauthorized access to and exfiltration of sensitive data stored within Longhorn. This attack tree can be used to identify potential vulnerabilities, assess risks, and develop strategies to mitigate or prevent such attacks.

<hr>

## Security functions and features

Critical Security Components:
* Volume Encryption: Ensures that data at rest is encrypted, providing confidentiality and protection against unauthorised access. This is a non-configurable element critical for data security, especially when handling sensitive information.
* Data Replication Integrity: Synchronous replication of data across multiple replicas in different nodes ensures high availability and integrity of data. This is vital for maintaining the state of the data in case of node failure.
* Automated Snapshot Management: Regular, automatic snapshots protect against data loss by preserving the state of data at specific points in time. These snapshots form the basis for recovery in the event of corruption or data loss incidents.

Security-Relevant Components:
* Configurable Replication and Backup Settings: While the replication process itself is critical, the ability to configure the number of replicas and backup intervals allows administrators to tailor the redundancy and backup frequency to the risk profile of the data being stored.
* Kubernetes' Role-Based Access Control (RBAC) Integration: By leveraging Kubernetes RBAC, Longhorn ensures that only authorised users and processes can manage volumes, snapshots, and backups, thus enforcing the principle of least privilege.
* Health Checks and Monitoring: Longhorn's continuous health checks and monitoring of volumes and replicas are crucial for early detection of potential issues, allowing for proactive remediation before they escalate into security incidents.
* Backup Data Validation: Before restoring data from backups, Longhorn validates the backup integrity, ensuring that the data has not been tampered with or corrupted, which is essential for the reliability of the restoration process.


## Project compliance
* Compliance: Longhorn adheres to best practices for Kubernetes storage solutions, ensuring it aligns with the security and operational standards expected within the cloud-native ecosystem. However, specific compliance certifications or adherence to standards like PCI-DSS, COBIT, ISO, or GDPR have not been documented.


## Secure development practices

Development Pipeline
* Version Control and Commit Signing: Longhorn uses Git for version control. Contributors are encouraged to sign commits to verify the identity of contributors.
* Code Reviews: Each pull request requires a thorough review by at least two maintainers to ensure quality and security.
* Continuous Integration and Deployment (CI/CD): Longhorn utilises automated CI/CD pipelines to build and test code, ensuring that tests pass before merging.
* Container Image Security: Container images are built using trusted base images, and measures are taken to make images immutable and signed to prevent tampering.
* Automated Vulnerability Checks: The project includes automated security scanning tools in the CI pipeline that regularly scan the codebase and dependencies for known vulnerabilities.
* Static Code Analysis: Longhorn employs static code analysis tools to detect potential security issues before they are merged into the main codebase.
  
Communication Channels
* Internal: The Longhorn development team uses Slack channels for real-time communication and GitHub for asynchronous communication, issue tracking, and feature planning.
* Inbound: Users and prospective users can file issues or feature requests via GitHub Issues. Additionally, they can seek support and discuss with the community on the Longhorn Forum.
* Outbound: Updates, announcements, and security advisories are communicated through the official Longhorn website and GitHub repository. They may also use mailing lists such as longhorn-announce@ for significant updates or security announcements.

Ecosystem
* Longhorn is designed to be an integral part of the cloud-native ecosystem, providing persistent storage solutions that are fully integrated with Kubernetes. It supports dynamic provisioning of storage, seamless scaling, and recovery features, which are critical for Kubernetes deployments.
* It complements other CNCF projects by adhering to the principles of containerisation, orchestration, micro-services, and immutable infrastructure.
* Longhorn fills the need for a reliable distributed block storage system in Kubernetes, making stateful applications and services more resilient and easier to manage in cloud-native environments.


## Security issue resolution

Responsible Disclosures Process
* Longhorn adopts a responsible disclosure policy, where external and internal parties are encouraged to report suspected security vulnerabilities through a predefined process.
* Strategy: Upon receiving a report, the team acknowledges receipt, conducts a confidential investigation, and works on a fix in a private repository to prevent premature exposure of the vulnerability.

Vulnerability Response Process
* Responsible Team: The Longhorn security team is tasked with responding to security reports. This team includes maintainers who have a deep understanding of the codebase and security practices.
* Reporting Process: Reporters are asked to include detailed information about the issue, steps to reproduce, and any other relevant data.
* Response Actions: The team assesses the report for validity, determines the severity, develops a fix, and then coordinates a release timeline for the patch. They also communicate with the reporter throughout the process, providing updates and seeking additional information if necessary.

Incident Response
* Triage: Security incidents are prioritised based on severity, impact, and complexity. The team rapidly assesses the scope to understand the breadth of the impact.
* Confirmation: The team confirms the incident and identifies affected systems and data.
* Notification: Stakeholders, including users and potentially affected parties, are informed about the incident as appropriate. Communication is carefully managed to ensure transparency while avoiding unnecessary alarm.
* Patching: Development of a security patch is expedited, and upon completion, the patch is made available. Users are notified of the update and provided with instructions for applying the patch.
* Post-Incident Analysis: After resolving the incident, the team conducts a post-mortem to understand the root cause, improve future response efforts, and enhance security measures to prevent similar incidents.


## Appendix
Known Issues Over Time
* Longhorn tracks all issues publicly on their [GitHub issues page](https://github.com/longhorn/longhorn/issues). Any security vulnerabilities discovered in the past are documented there along with their resolutions.
* The project maintains a [security advisory](https://github.com/longhorn/longhorn/security/advisories) page where they publish details about the security vulnerabilities found.
* Statistics on vulnerabilities found and fixed are not explicitly listed unless provided on the project’s repository or documentation. If available, data about the effectiveness of code reviews and automated testing in catching issues would be included.

CII Best Practices
* Longhorn is actively working towards meeting the CII Best Practices criteria.
* Longhorn’s commitment to security can be observed in their adherence to these practices, such as using automated testing, code review, and maintaining a public version control repository.

Case Studies
* Case studies detailing real-world usage of Longhorn can provide valuable insights into its effectiveness and reliability.
* Companies from different fields such as technology, finance, business services, or more that utilize the Kubernetes cluster tend to use it for secure, provision, and backup storage and more.
* Further information can be found here: [Companies using Longhorn](https://discovery.hgdata.com/product/rancher-longhorn).

Related Projects / Vendors
* Longhorn is often compared to other CNCF projects or cloud-native storage solutions such as Rook, OpenEBS, and Portworx.
* Especially Rook is often the most compared with Longhorn since both are Kubernetes-native. Rook is an open-source cloud-native storage utility for Kubernetes that aims to automate some of the tasks of a storage administrator, such as programmatic storage, migration, disaster recovery, monitoring, and resource management. Longhorn is valued to be easier to work on from scratch, but Rook is preferred when dealing with larger clusters in PRD.
* Prospective users are interested in differences in performance, scalability, ease of use, and specific features like snapshotting and backup/restore capabilities.


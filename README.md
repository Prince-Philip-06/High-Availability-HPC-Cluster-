# High Availability HPC Cluster

A production-grade High Availability High Performance Computing (HPC) cluster designed for **fault tolerance, automated failover, secure remote access, and real-time observability**.

This project demonstrates how to engineer a resilient HPC environment using DRBD replication, Pacemaker/Corosync clustering, Slurm scheduling, centralized authentication, and Ansible automation.

---

## Architecture Overview

![Image](https://d2908q01vomqb2.cloudfront.net/e6c3dd630428fd54834172b8fd2735fed9416da4/2021/06/03/ha-hpc-fig1.png)

![Image](https://wiki.anunna.wur.nl/images/d/d5/Cluster_scheme.png)

![Image](https://slurm.schedmd.com/arch.gif)

![Image](https://api.app.rwth-aachen.de/v3/sabio/Manual/File/documents%2F0962a984657bb2e901657bc9e2810004%2F1jifm42ljvvhw%2Fc9fba738b70746e9b1d086be14901bc0%2Fslurmdiagramv2.png)

The system is built around an multi-layer architecture that separates access, control, and compute:

```mermaid
flowchart TB
    User[External Users] --> VPN[WireGuard VPN]
    VPN --> Login[Login Node Gateway]

    Login --> Active[Active Master]
    Login --> Passive[Passive Master]

    Active <-->|DRBD Sync| Passive

    Active --> W1[Worker Node 1]
    Active --> W2[Worker Node 2]

    Passive -. Failover .-> W1
    Passive -. Failover .-> W2

    Login --> NFS[(Shared NFS Storage)]
    Active --> NFS
    Passive --> NFS
    W1 --> NFS
    W2 --> NFS
```

---

## System Architecture from Project Implementation

![Image](https://www.digitalengineering247.com/images/article/HPC-Workflow.jpg)

![Image](https://www.researchgate.net/publication/387403530/figure/fig1/AS%3A11431281299906437%401735147585553/High-Performance-Computing-HPC-workflow-architecture.png)

![Image](https://assets.serverwatch.com/uploads/2023/10/SW_FailoverClustering_23-1024x768.png)

The architecture consists of:

### Login Node (Gateway)

The login node is the secure entry point to the cluster and provides:

* LDAP authentication and centralized identity
* DNS and DHCP network services
* NTP time synchronization
* NFS shared storage
* WireGuard VPN remote access
* Firewall enforcement
* User on-demand web portal

Users interact only through this node.

---

### Master Nodes (High Availability Core)

Two master nodes operate in active/passive mode.

```mermaid
sequenceDiagram
    participant Active as Active Master
    participant Passive as Passive Master
    participant Coro as Corosync/Pacemaker

    Active->>Coro: Heartbeat signal
    Note over Active: Normal operation

    Active--xCoro: Failure detected
    Coro->>Passive: Promote to Active
    Passive->>Passive: Acquire Virtual IP
    Passive->>Cluster: Resume scheduling
```

Active Master:

* Runs Slurm controller
* Manages job queues
* Hosts monitoring stack

Passive Master:

* Mirrors active state via DRBD
* Automatically takes over on failure

No job data is lost during failover.

---

### Worker Nodes

Worker nodes execute distributed workloads.

```mermaid
flowchart LR
    Slurm[Slurm Scheduler] --> Worker1
    Slurm --> Worker2

    Worker1 -->|Job Output| Storage[(Shared Storage)]
    Worker2 -->|Job Output| Storage
```

They:

* Run Slurm execution daemons
* Access shared NFS storage
* Execute user jobs
* Receive automatic rescheduling if a node fails

---

## Technology Stack and Communication

```mermaid
graph TD
    LDAP --> Login
    Login --> Slurm
    Slurm --> Workers
    DRBD <-->|Sync| Masters
    Prometheus --> Grafana
    Masters --> Prometheus
    Workers --> Prometheus
```

### Core Technologies

**Slurm Workload Manager**

Handles job scheduling and resource allocation.

**Pacemaker + Corosync**

Provides cluster orchestration and automated failover.

**DRBD Replication**

Ensures synchronous storage replication between masters.

**NFS Shared Storage**

Delivers unified filesystem access across nodes.

**LDAP Authentication**

Centralized user identity and access control.

**WireGuard VPN**

Encrypted secure remote access.

**Prometheus + Grafana**

Real-time monitoring and visualization.

**HPL Benchmarking**

Validates distributed compute performance.

---

## System Workflow

```mermaid
flowchart LR
    User --> Auth[LDAP Authentication]
    Auth --> Submit[Job Submission]
    Submit --> Schedule[Slurm Scheduling]
    Schedule --> Execute[Worker Execution]
    Execute --> Store[Shared Storage]
    Store --> Monitor[Monitoring Stack]
```

1. User connects via VPN and web portal
2. LDAP authenticates identity
3. Jobs are stored on shared storage
4. Slurm schedules execution
5. Workers process tasks
6. Monitoring tracks performance
7. Results are stored centrally

Failover activates automatically on node failure.

---

## Monitoring and Performance Validation

![Image](https://grafana.com/mw/_next/image/?q=75\&url=https%3A%2F%2Fs3.amazonaws.com%2Fa-us.storyblok.com%2Ff%2F1022730%2F73f39454fa%2Fself-hosted-grafana-dashboard.png\&w=3840)

![Image](https://prometheus.io/assets/docs/grafana_qps_graph.png)

![Image](https://s3.amazonaws.com/a-us.storyblok.com/f/1022730/fa4a784cd5/nersc_grafana-dashboard_blog.png)

The monitoring stack provides:

* Real-time node health visualization
* Resource utilization dashboards
* Slurm job analytics
* Failover detection
* Performance trends

HPL benchmarking validates cluster performance under full load.

---

## Automated Deployment with Ansible

The cluster is fully deployable using automation.

### Setup Steps

Clone the repository:

```
git clone https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-.git
cd High-Availability-HPC-Cluster
```

Configure inventory:

[https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/ansible/inventory](https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/ansible/inventory)

Run deployment playbooks:

```
ansible-playbook ansible/playbooks/deploy-cluster.yml
```

Playbooks:

[https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/ansible/playbooks](https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/ansible/playbooks)

Cluster configurations:

[https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/configs](https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/configs)

---

## Project Documentation

Project report:

[https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/blob/main/docs/High-Availability-HPC-Cluster-Report.pdf](https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/blob/main/docs/High-Availability-HPC-Cluster-Report.pdf)

Project presentation:

[https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/blob/main/docs/High-Availability-HPC-Cluster-Presentation.pptx](https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/blob/main/docs/High-Availability-HPC-Cluster-Presentation.pptx)

Automation scripts:

[https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/ansible](https://github.com/Prince-Philip-06/High-Availability-HPC-Cluster-/tree/main/ansible)

---

## Failover Behavior

When the active master node fails:

1. Corosync detects heartbeat loss
2. Pacemaker migrates services
3. Virtual IP switches nodes
4. Passive master becomes active
5. Slurm resumes scheduling

Users experience uninterrupted execution.

---

## Security Model

The cluster uses layered security:

* VPN-only external access
* Firewall restrictions
* Private internal networking
* Centralized LDAP authentication
* Encrypted communication
* Role-based permissions

---

## Use Cases

This system supports:

* Scientific simulations
* Research computing
* Academic HPC labs
* Fault-tolerant infrastructure training
* Distributed analytics

---

## Future Improvements

* Cloud bursting integration
* Advanced checkpointing
* Automated alert notifications
* Container orchestration
* Scalable node expansion

---

## Conclusion

This project demonstrates a complete high availability HPC architecture combining redundancy, automation, performance, and security. The system delivers continuous operation, transparent failover, and scalable distributed computation.

Anyone following this documentation can reproduce the cluster and understand how modern HPC infrastructure is engineered.

---

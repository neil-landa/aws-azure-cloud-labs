---
title: Securing Networks with Micro-Segmentation using NACLs and Security Groups
id: b3e29925
category: networking
difficulty: 300
subject: aws
services: VPC, EC2, CloudWatch, IAM
estimated-time: 180 minutes
recipe-version: 1.2
requested-by: mzazon
last-updated: 2025-07-12
last-reviewed: 2025-07-23
passed-qa: null
tags: vpc, security, groups, nacls, cloudwatch, networking, micro-segmentation
recipe-generator-version: 1.3
---

# Securing Networks with Micro-Segmentation using NACLs and Security Groups

## Problem

Enterprise organizations must implement granular network security controls to prevent lateral movement of threats and ensure compliance with regulatory frameworks like PCI DSS, HIPAA, and SOC 2. Traditional network security approaches often rely on perimeter-based defenses that allow broad access within the network, creating security vulnerabilities when an attacker gains initial access. Modern enterprise applications require fine-grained network segmentation that isolates different tiers, environments, and data classification levels while maintaining operational efficiency and application performance.

## Solution

This solution implements a comprehensive micro-segmentation strategy using Network Access Control Lists (NACLs) for subnet-level enforcement and advanced security group configurations for instance-level controls. The architecture creates multiple security zones with layered defenses, automated traffic monitoring, and zero-trust networking principles that ensure every network connection is explicitly authorized and continuously validated.

## Architecture Diagram

```mermaid
graph TB
    subgraph "Production VPC (10.0.0.0/16)"
        subgraph "DMZ Zone (10.0.1.0/24)"
            DMZ_NACL[DMZ NACL<br/>Internet-facing Rules]
            ALB[Application Load Balancer]
            DMZ_SG[DMZ Security Group<br/>Port 80/443 from Internet]
        end

        subgraph "Web Tier Zone (10.0.2.0/24)"
            WEB_NACL[Web Tier NACL<br/>ALB-only Access]
            WEB[Web Servers]
            WEB_SG[Web Security Group<br/>Port 80/443 from ALB]
        end

        subgraph "App Tier Zone (10.0.3.0/24)"
            APP_NACL[App Tier NACL<br/>Web Tier Access]
            APP[Application Servers]
            APP_SG[App Security Group<br/>Port 8080 from Web]
        end

        subgraph "Data Tier Zone (10.0.4.0/24)"
            DB_NACL[Database NACL<br/>App Tier Only]
            RDS[RDS Database]
            DB_SG[Database Security Group<br/>Port 3306 from App]
        end

        subgraph "Management Zone (10.0.5.0/24)"
            MGMT_NACL[Management NACL<br/>Admin Access Only]
            BASTION[Bastion Host]
            MGMT_SG[Management Security Group<br/>SSH from VPN]
        end

        subgraph "Monitoring Zone (10.0.6.0/24)"
            MON_NACL[Monitoring NACL<br/>All Zones Access]
            CLOUDWATCH[CloudWatch Agent]
            VPC_FL[VPC Flow Logs]
            MON_SG[Monitoring Security Group<br/>Metrics Collection]
        end
    end

    subgraph "Internet Gateway"
        IGW[Internet Gateway]
    end

    subgraph "VPN Gateway"
        VGW[VPN Gateway]
    end

    IGW --> DMZ_NACL
    DMZ_NACL --> WEB_NACL
    WEB_NACL --> APP_NACL
    APP_NACL --> DB_NACL
    VGW --> MGMT_NACL
    MON_NACL --> CLOUDWATCH

    ALB --> WEB
    WEB --> APP
    APP --> RDS

    style DMZ_NACL fill:#FF6B6B
    style WEB_NACL fill:#4ECDC4
    style APP_NACL fill:#45B7D1
    style DB_NACL fill:#96CEB4
    style MGMT_NACL fill:#FFEAA7
    style MON_NACL fill:#DDA0DD
```

## Prerequisites

1. AWS account with appropriate permissions for VPC, EC2, RDS, CloudWatch, and IAM operations
2. AWS CLI v2 installed and configured (or AWS CloudShell)
3. Understanding of network security principles, CIDR blocks, and multi-tier architectures
4. Familiarity with security group and NACL rule precedence and evaluation order
5. Estimated cost: $200-300/month for EC2 instances, RDS, and CloudWatch monitoring

> **Note**: This recipe demonstrates advanced security concepts. Test thoroughly in a non-production environment before implementing in production workloads.

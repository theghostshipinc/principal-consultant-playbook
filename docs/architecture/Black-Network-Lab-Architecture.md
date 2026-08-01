# Black Network Lab Architecture

Version: 1.0

---

# Vision

The Black Network Lab is a persistent enterprise engineering environment designed for:

- Enterprise networking
- Security
- Load balancing
- Automation
- Cloud networking
- Multi-vendor interoperability
- Customer solution validation

This environment serves as both a learning platform and a professional portfolio.

---

# Compute Platforms

## LabCentral

Role:

Primary virtualization workstation

Operating System:

Windows 11

Primary Technologies

- VMware Workstation
- GNS3
- Cisco ACI Simulator
- F5 BIG-IP
- Palo Alto
- Fortinet
- Cisco SD-WAN
- CCIE Labs

---

## LabCentral2

Role:

Automation and cloud-native infrastructure

Operating System

Ubuntu Server

Primary Technologies

- Docker
- Containerlab
- Arista EVPN
- Git
- Python
- Ansible
- Terraform

---

# Lab Tiers

## Tier 1

Persistent Infrastructure

- Cisco ACI Simulator
- Arista Containerlab
- Automation Services
- Git Repositories

---

## Tier 2

Long-Term Engineering Labs

- F5 Principal Consultant Lab
- Palo Alto Lab
- Fortinet Lab
- Cisco SD-WAN Lab
- CCIE Lab

---

## Tier 3

Temporary Labs

- Customer reproductions
- Interview labs
- Proof of Concepts

---

# Source of Truth

GitHub repositories are considered the source of truth.

Running virtual machines are not.

Configuration should always be reproducible.

---

# Documentation Standards

Every lab must include:

- README
- Architecture diagram
- IP addressing
- Validation steps
- Troubleshooting guide
- Git history
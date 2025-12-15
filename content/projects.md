---
title: "Projects"
date: 2025-10-07T13:38:45+02:00
draft: false
---

# Volunteering: De Sering

[De Sering](https://desering.org/) is a community center, kitchen, and public
space dedicated to bringing people together through food, culture, and shared
purpose. My involvement started small at the beginning of 2024 with cooking
large amounts of food throughout most of the year, and then turned to building
up a group of volunteers who now come together regularly to repair broken
kitchen equipment and maintain and improve the public space in its entirety.

As this group turned more stable and autonomous, I shifted focus towards the
digital infrastructure of our fast-growing community - better websites,
communication channels, knowledge sharing and ways to get involved were in dire
need, but we did not have the financial means to support it. This led me to
build out a small, automated platform to host open-source tools that enable and
support our community as best as possible - all in the open and for everyone to
see and contribute.

We are now operating various different applications and websites, host a weekly
event where community members can get involved and contribute, and we even
started our own open-source software project which is seeing significant
involvement and garnering interest from other volunteer-based organisations in
the area. Our biggest challenge at the moment is scaling our processes to the
unexpected amount of volunteers wanting to get involved and contribute because
they see value in our purpose and vision - a really nice problem to have :).

**Used technologies**

 * Google Workspace for Nonprofits
 * Hetzner Cloud (virtual servers, storage buckets, managed via Terraform)
 * Cloudflare (DNS and DDoS protection, managed via Terraform)
 * Terraform, Ansible (Infrastructure as Code)
 * GitHub Actions (CI/CD, provisioning and configuring infrastructure, testing and deploying applications)
 * GitHub Pages for stateless web apps

**Hosted applications**

 * [Coolify](https://github.com/coollabsio/coolify) (container runtime/orchestrator)
 * Parts of our Websites (https://desering.org/, https://testtafel.nl/)
 * A wiki ([Wiki.js](https://github.com/requarks/wiki))
 * A self-built application that organises volunteer contributions
   * https://github.com/desering/volunteer-scheduler/
   * Open Source project with active participation and contributions from the community
 * A communication platform ([Mattermost](https://github.com/mattermost/mattermost))
 * A password manager ([Vaultwarden](https://github.com/dani-garcia/vaultwarden))
 * Single Sign-On ([Authentik](https://github.com/goauthentik/authentik))
 * Monitoring and alerting with Prometheus, Grafana, Netdata
 * And more...

# Homelab / Cloudlab

This is my playground for anything I'm curious about - self-hosted systems,
enterprise-grade hardware, testing things for work or supporting the community
I volunteer at. Currently, this includes:

 * My own fiber internet connection (Mikrotik RB5009UG+S+IN, Ubiquiti UniFi AP)
 * A self-built "server" (32gb RAM, 12core CPU, 4x 3TB HDDs)
 * Multiple Raspberry PIs for various smaller projects
 * Ubuntu MaaS / Talos Linux to provision immutable k8s clusters directly on hardware
 * Ansible configuration management
 * A 3D Printer ([Creality Ender-6](https://www.creality.com/products/ender-6-3d-printer))
 * This personal website/blog on GitHub Pages
 * Automation with GitHub Actions and Terraform
 * Some minor things in Cloudflare and AWS

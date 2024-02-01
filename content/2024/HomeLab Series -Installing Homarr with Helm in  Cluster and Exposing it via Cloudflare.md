---
notetype: Blog
title: HomeLab Series - Install Homarr and expose it
date: 2024-01-09
tags:
  - output/blog
  - Tech/Cloud
up: "[[Tech MOC]]"
---
  In this guide, I will walk you through the process of using Helm to install Homarr as a navigation page in your cluster, and then how to expose it to the public internet through Cloudflare.

  
  Using this method, Cloudflare automatically adds and manages the certificates, eliminating concerns about certificate renewals. This approach is more convenient compared to using Let's Encrypt.

  Cloudflare also offers multiple authentication methods, enhancing security beyond what traditional firewall and local network configurations can provide. This multi-faceted approach to authentication makes it more robust and secure.


![image](https://github.com/bintis/xirin/assets/57840704/94c5f18f-1a3b-43d1-8c41-fbf481157425)



*previous* : [[HomeLab Series -  Setting up K3s Cluster with MetalLB]]

*Next*: [[HomeLab Series -Installing Roon in  Cluster and Setup Back]]

### 1. **Installing Homarr via Helm**

#### a. Add the Deler Helm Repository

First, add the Deler Helm repository, provided by Deler Aziz. Detailed information can be found in the [Homarr README](https://github.com/deler-aziz/helm-charts/blob/main/charts/homarr/README.md).


```bash
helm repo add deler https://helm.deler.dev
```

#### b. Update Information on Available Charts

Refresh your local list of available charts from the repositories:

```bash
helm repo update
```

#### c. Install the Chart

Install Homarr, setting the service type to LoadBalancer to obtain a local IP address:

```bash
helm install my-release deler/homarr --set service.type=LoadBalancer
```

### 2. **Retrieving the Homarr LoadBalancer IP Address**

Use the `kubectl` command to fetch the external IP address assigned to Homarr:


```bash
kubectl get svc my-release-homarr
```

### 3. **Configuring the IP Address through Cloudflare Tunnel**

#### a. Sign Up for Cloudflare and Add Your Domain

If you're new to Cloudflare, sign up at [Cloudflare](https://www.cloudflare.com/) and add your domain.

#### b. Install Cloudflare Tunnel on Your Server

Download and install the `cloudflared` daemon from the Cloudflare Tunnel documentation.

#### c. Authenticate `cloudflared`

Authenticate `cloudflared` with your Cloudflare account:


```bash
cloudflared tunnel login
```

#### d. Create a Tunnel

Create a new tunnel with a command like:

```bash
cloudflared tunnel create my-tunnel
```

#### e. Configure the Tunnel

Set up the tunnel configuration in `~/.cloudflared/config.yml`, specifying your tunnel ID and the local service address:


```bash
tunnel: <TUNNEL-ID> credentials-file: /root/.cloudflared/<TUNNEL-ID>.json ingress: - hostname: homarr.example.com service: http://localhost:PORT - service: http_status:404
```

#### f. Start the Tunnel

Begin routing traffic to your Homarr service:

```bash
cloudflared tunnel run my-tunnel
```

#### g. Update DNS Records in Cloudflare

In Cloudflare, add a CNAME record pointing your desired subdomain to your tunnel.

#### h. Verify the Setup

Access Homarr through your configured domain, such as `homarr.example.com`.

This guide provides a comprehensive overview of installing Homarr in a cluster using Helm and securely exposing it through Cloudflare Tunnel.


***

<a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/"><img alt="クリエイティブ・コモンズ・ライセンス" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-nd/4.0/88x31.png" /></a><br />この 作品 は <a rel="license" href="http://creativecommons.org/licenses/by-nc-nd/4.0/">クリエイティブ・コモンズ 表示 - 非営利 - 改変禁止 4.0 国際 ライセンス</a>の下に提供されています。

@Bintis 著作权，不许抄。

+++
author = "Steve"
title = "Route Server"
date = "2024-11-28"
description = "Lab to undestand how route server work"
series = ["Networking"]
tags = [
    "Networking",
]
+++

How route server work?, lets see how that things work <!--more-->

This Blog is for documented my lab about route server using [IXP Manager](https://www.ixpmanager.org/)

In production you can see [hare](https://squad.iix.net.id/login)

So much tool like IXP Manager, to manage Internet Exchange Point, the other things is [Alice Looking Glass](https://github.com/alice-lg/alice-lg), 
but i use IXP Manager because this tool so much feature to manage Internet Exchange Point. 

First we must undestand [BGP bird](https://bird.network.cz/), because in ixp manager using bgp bird version 2 and then bird is can be customized. 

Here is my lab topologi, 

![Topologi](https://raw.githubusercontent.com/Thevi1/Thevi.github.io/main/images/Route-server/TOPO.PNG)

## Table of Contents
1. [What is Route Server?](#what-is-route-server)
2. [How Crucial Route Server on Data Center](#how-crucial-route-server-on-data-center)

---

## What is Route Server
 
Route servers facilitate the sharing of routing information between multiple autonomous systems (ASes) without each network having to directly establish multiple peer-to-peer BGP (Border Gateway Protocol) sessions. This is especially useful in an IXP environment, where multiple networks (ISPs, data centers, etc.) connect to a common exchange.

## How Crucial Route Server on Data Center

Let undestand how crucial route server is. 

### Peering Simplification

- #### Simplifies Peering Relationships
- A route server allows multiple Autonomous Systems (ASes) to exchange routing information without requiring direct, bilateral peering agreements between each pair. Instead of every network operator needing to manually configure routes for every other network, the route server automates and centralizes the process.
- #### Reduces Complexity
- large-scale data centers or IXPs, where many networks peer with each other, a route server simplifies peering management and reduces the number of direct connections required, saving time and effort.


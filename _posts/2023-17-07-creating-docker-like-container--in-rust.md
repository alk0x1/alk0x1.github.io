---
layout: post
title:  "Simplifying Traits in Rust"
date:   2023-07-17 14:35:50 -0300
categories: ["Rust", "Computer Science"]

---

1. What are docker containers?
  - Which problem solve?
  - Architecture
  ![alt text for screen readers](/_posts/architecture.svg "Text to show on mouseover")
  - How are implemented?
1. What docker containers isn't?
2. Creating a container using Rust.
3. Creating a VM using Rust. (Other article)


------------
As Virtual Machines, containers are designed to provide isolated enviroment to run an application, but the main difference is that Virtual Machines create a new kernel possibly with a new architecture, while containers reuse the same kernel.

### Limiting Processes
Cgroups
namespaces

### Why containers isn't a virtualization model?
Virtual Machines emulate a whole machine, Containers just put the process into a namespace but use the same machine for that, they basically just "hide" everything from the process since network, clocks from system, users and groups etc. In that way the process will really think that he is in another machine. In practice "Container" is a name given to a process running without the same accesses that the other processes in the OS have.

### Isolating 
Resources:
- Memory
- CPU
- Network
- 
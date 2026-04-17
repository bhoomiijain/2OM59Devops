# VMs vs Containers

VMs run multiple OSes using hypervisor. Each VM = full OS + memory + CPU.

Good: strong isolation, run Linux and Windows together, disaster recovery
Bad: heavy (full OS), slow boot (minutes), expensive resources, overhead

Containers share host OS kernel instead of running full OS. Shares kernel, only takes what app needs, seconds to start, minimal resources, hundreds on same machine.

Key: VMs = full OS each, containers = shared kernel

Comparison:
- OS: VMs guest OS each, containers share host kernel
- Boot: VMs minutes, containers seconds
- Size: VMs GBs, containers MBs
- Isolation: VMs strong, containers process-level
- Portability: VMs low, containers high
- Resource: VMs high, containers low
- Use: VMs different OS, containers microservices

Why containers won: Apps evolved monolithic → components → microservices. VMs worked for split apps but microservices need fast, lightweight, portable. Containers perfect fit. Same container runs on dev laptop and production.


# cgroups (Control Groups)

## What = cgroups?

Linux thing that limits resources for containers

**Namespaces** = isolation | **cgroups** = limits

Without cgroups → one container eats all CPU/RAM → everything crashes

## Why important?

Like apartment electricity:
- Each flat (container) has a limit
- No one tenant can use ALL power
- Fair distribution

## What cgroups control:

**CPU** - maximum CPU usage  
**Memory** - max RAM (kills container if over)  
**Disk I/O** - read/write speed limits  
**Network** - traffic control(?? check this)

## Without cgroups = BAD
- One container hogs all CPU
- Host freezes
- Other containers crash

## With cgroups = GOOD
- Each container limited
- System stable
- Even under heavy load

---
*Teacher talked about this but too fast. Need to practice with docker itself to understand better*
|---------|-----------|---------|
| Purpose | Isolation — hides resources | Limits — caps usage |
| What it does | Each container sees its own view | Controls how much resource each can use |
| Example | Container has PID 1 (isolated) | Container gets max 512MB RAM |

---


# cgroups

cgroups = Linux feature that limits resources for containers.

Namespaces = isolation (hides resources)
cgroups = limits (caps usage)

Without cgroups: one container eats all CPU/RAM and everything crashes.

Why important? Like apartment electricity - each flat (container) has a limit. No tenant can use ALL power. Fair distribution.

What cgroups control:
- CPU: maximum CPU usage
- Memory: max RAM (kills container if over)
- Disk I/O: read/write speed limits
- Network: traffic control

Without cgroups = BAD
- One container hogs all CPU
- Host freezes
- Other containers crash

With cgroups = GOOD
- Each container limited
- System stays stable
- Handles heavy load


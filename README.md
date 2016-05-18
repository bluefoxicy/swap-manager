# swap-manager
Configurable swap space manager using simple rules

# Configuration

Configuration file is in /etc/swap-manager/swap-manager.conf

Configuration is blocked off in sections based on type of allocation.

Configuration allows multiple `include` statements using shell globs:

```
[main]
include = conf.d/*.conf
```
## General

A few settings are general and trigger for any type of managed swap.

```
[swapfile]
type=file
min_free_swap=20M
max_free_swap=64M
min_size=256M
max_size=1G
max_total=2G
max_devices=4
coalesce=1
division_size=128M
swap_priority=5000
priority=100
```

* **type** - The type of storage (e.g. swap file, zram device)
* **min\_free_swap** - Below this value, swap-manager will add more of
  this type of swap.  Accepts a 1024-based magnitude (K,M,G)
* **max\_free_swap** - Above this value, swap-manager will attempt to
  remove swap.  Removal will fail if it can't be done without falling
  below  _min\_free\_swap\_.
* **min\_size** - The minimum-size device to create.
* **max\_size** - The maximum-size device to create.
* **max\_total** - The maximum total amount of swap space to allocate
  this way.  If unset, is unlimited.
* **max\_devices** - Prohibits creation of more than a number of devices
  under this policy.
* **coalesce** - If set to 1, will handle reaching _max\_devices_ but
  not reaching _max\_total_ by creating a larger device and removing
  smaller devices.
* **division\_size** - If set, will allow creation of new, smaller
  devices and then removal of the larger device to respond to exceeding
  _max\_free\_swap_.
* **swap\_priority** - Kernel swap priority.
* **priority** - The priority of this rule.  swap-manager evaluates the
  highest priority rules first; if it takes action, it re-evaluates the
  rule chain from the beginning.

Coalescing and division can cause system disruption by moving data from
one swap device to another, either through disk writes (swap files) or
compression (zram)

## zram

The zram configuration looks as such:

```
[zram]
type=zram
max_ram_usage=25%
;max_ram_usage=8G
algorithm=lz4
;algorithm=lzo
```

These configuration values are thus:

* **type** - Always equal to `zram`.
* **max\_ram\_usage** - Either a percentage of RAM or an absolute value.
  Must be less than 100% of RAM.
* **algorithm** - lz4 (faster) or lzo (better compression).

With zram, _max\_ram\_usage_ obviates the need for _max\_total_:  if you
don't configure a _max\_total_, zram will provide as much swap as it can
using a fixed portion of your RAM.

## file

Creates swap files.

```
[swapfile]
type=file
path=/var/lib/swap
```

* **type** - file
* **path** - The path at which to place swap files.  Must be a
  directory.


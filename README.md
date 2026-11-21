# tssh

**NAME**

tssh - ssh over Tailscale, without root

**SYNOPSIS**

    tssh [ssh-options] [user@]host [command]

**DESCRIPTION**

tssh starts a private, userspace tailscaled from its own directory,
runs one ssh(1) through it, and stops it when the session ends.
Nothing is installed; no interface, route, resolver or service on the
machine is changed. host is a tailnet name or a 100.x address.

On first use the static Tailscale binaries are fetched into `bin/` and
a login URL is printed. Later runs reuse the saved identity.

The node accepts no inbound connections (shields up), so nothing on
this machine becomes reachable from the tailnet. Only the ssh started
by tssh uses the tunnel; `run/` and `state/` are private to the user.

Concurrent sessions share one daemon; the last to exit stops it.

The user's ssh configuration and keys are used as usual. Host keys
seen through tssh are kept apart, in `known_hosts`.

**ENVIRONMENT**

`TS_AUTHKEY`
: auth key; log in without a browser.

`TSSH_EPHEMERAL=1`
: keep no identity on disk. Requires an ephemeral `TS_AUTHKEY`;
  the node logs out on exit.

`TSSH_HOSTNAME`
: name shown in the tailnet. Default `tssh-`*hostname*.

**FILES**

    bin/          tailscale, tailscaled
    state/        node identity, logs
    run/          socket, pid, locks, daemon log
    known_hosts   host keys

**EXAMPLES**

    tssh pi
    tssh -p 2222 root@100.101.102.103 uptime
    TS_AUTHKEY=tskey-... TSSH_EPHEMERAL=1 tssh build@ci

To make scp(1) or rsync(1) go the same way, use the ProxyCommand that
tssh passes, which `tssh -v` will show.

**REQUIRES**

POSIX sh, ssh, flock(1). curl and tar for the first fetch only. Linux.

**SEE ALSO**

ssh(1), tailscale(1), tailscaled(8)

**BUGS**

Copying the directory copies the node; two copies running at once
contend for one identity.

Userspace networking is slower than a kernel interface.

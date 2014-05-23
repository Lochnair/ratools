ratools
=======

This is ratools, a fast, dynamic, multi-threading framework for creating,
modifying and sending IPv6 Router Advertisements (RA).


Architecture
------------

The basic idea is to have two programs. On the one hand a
powerful CLI, called ratools/ractl, for manipulating the
configuration. And on the other hand an efficient, non-bloated
stable daemon process, called ratools/rad. As depicted below,
both programs communicate with each other via an UNIX socket.


                              +------------------+
                              |                  +-- eth0
                              |  ratools/rad     +-- eth1
                              |                  +--  :
                              |           Daemon +-- ethN
                              +---+--------------+
    +------------------+          |
    |                  |          |
    |  ratools/ractl   |          |
    |                  +----------+ AF_UNIX
    |              CLI |
    +------------------+


The internal structure of ratools/rad looks like this:



       + KERNEL                        + INTERFACES
       |                               |
    +--+-------------+              +--+-------------+
    | Netlink Socket |          +-->|   RAW Socket   +---+
    +--+-------------+          |   +----------------+   |
       |                        |                        |
       v                        |                        v
    +----------------+       +--+-------------+       +----------------+
    | Configuration  +---+   |  Packet Delay  |<------+  RS Listener   |
    |   Database     |   |   |    Threads     |       |     Thread     |
    +----------------+   |   +----------------+       +--+-------------+
       ^                 |      ^                        |
       |                 |      |                        |
    +--+-------------+   |   +--+-------------+          |
    |  UNIX Socket   |   +-->|   RA Worker    |<---------+
    +--+-------------+       |    Threads     |
       |                     +----------------+
       + CLI



Building
--------

Although ratools is complex from the inside, the code looks very POSIX-ish from
the outside. Thus, bulding is relatively easy. Clone the repository and type
`make`. It is as simple as that!

    $ git clone git@github.com:danrl/ratools.git
    Cloning into 'ratools'...
    [...]
    $ cd ratools/src/
    $ make
    [...]


Quick start
-----------

First start the daemon (ratools/rad)

    # rad --loglevel info
    ratools/rad 0.3.5 (May 2014)
    Log: Level set to `info'.
    Info: Netlink: Thread started.
    Info: Listener: Thread started.


Now use the CLI (ratools/ractl) to create a new RA

    $ ractl ra@dummy0 create
    $ ractl ra@dummy0 enable

How about advertising the link MTU?

    $ mtu@dummy0 create
    $ mtu@dummy0 enable

It is also nice to let clients know the link-local address.

    $ sll@dummy0 create
    $ sll@dummy0 enable

Of course we like to advertise at least one prefix!

    $ pi0@dummy0 create
    $ pi0@dummy0 set prefix 2001:db8::/64
    $ pi0@dummy0 enable

Nice routers come with recursive DNS server addresses as well:

    $ rdnss0@dummy0 create
    $ rdnss0@dummy0 add server 2001:db8::53
    $ rdnss0@dummy0 enable

Have at look at your ICMPv6 masterpiece using the CLI

    $ ractl show
    Router Advertisement `ra@dummy0':
      State:                  Enabled
      Created:                2014-05-22 00:16:32
      Updated:                2014-05-22 00:18:19
      Version:                11/20           (Compilation scheduled)
      Interface ID:           4               (dummy0)
      Interface State:        1               (Up)
      Interface MTU:          1500
      Hardware Address:       4e:3a:65:ec:78:4f
      Link-local Address:     fe80::4c3a:65ff:feec:784f
      Forwarding:             1               (Enabled)
      Maximum Interval:       600             (0d 0h 10m 0s)
      Minimum Interval:       198             (0d 0h 3m 18s)
      Solicited/Unsolicited:  3/0
      Unicast/Multicast:      0/3
      Total RAs:              3               (56 Bytes)
      Next RA scheduled:      2014-05-22 00:23:27
      Current Hop Limit:      64
      Managed Flag:           0               (No Managed Address Configuration)
      Other Managed Flag:     0               (No Other Managed Configuration)
      Home Agent Flag:        0               (No Mobile IPv6 Home Agent)
      Router Preference:      00              (Medium)
      NDP Proxy Flag:         0               (No NDP Proxy)
      Lifetime:               1800            (0h 30m 0s)
      Reachable Time:         0               (0h 0m 0s 0ms)
      Retransmission Timer:   0               (0h 0m 0s 0ms)
      Link-MTU Option `mtu@dummy0':
        State:                Enabled
        Auto-detection:       On
        Link-MTU:             1500
      Source Link-layer Address Option `sll@dummy0':
        State:                Enabled
        Auto-detection:       On
        Hardware Address:     4e:3a:65:ec:78:4f
      Prefix Information Option `pi0@dummy0':
        State:                Enabled
        On-link Flag:         1               (On-link Prefix)
        Autonomous Flag:      1               (Autonomous Address Configuration)
        Router Address Flag:  0               (No Mobile IPv6 Router Address)
        Valid Time:           2592000         (30d 0h 0m 0s)
        Preferred Time:       604800          (7d 0h 0m 0s)
        Prefix:               2001:db8::/64
          Warning: Documentation prefix!
      Recursive DNS Server Option `rdnss0@dummy0':
        State:                Enabled
        Lifeime:              900             (0d 0h 15m 0s)
        Servers:
          Address:            2001:db8::53



Please make sure to use the latest version before reporting bugs. Thanks!

    $ ractl version


Resources
---------

* Mailing List http://www.freelists.org/list/ratools

* Mailing List Archive http://www.freelists.org/archive/ratools/

* Repository https://github.com/danrl/ratools

* Documentation https://www.nonattached.net/ratools/index.html


License
-------

    Copyright 2013-2014 Dan Luedtke <mail@danrl.de>

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

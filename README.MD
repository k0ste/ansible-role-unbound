ansible-unbound
==================

Role for deploy [Unbound](//www.unbound.net/) - validating, recursive, and caching DNS resolver.

Requirements for usage
-----------------------

* Ansible 2.2;
* GNU/Linux;
* [python-netaddr](//docs.ansible.com/ansible/playbooks_filters_ipaddr.html);

Role support
-------------

* unbound server;
* unbound local zones;
* unbound forward zones;
* unbound stub zones;
* unbound views;
* unbound control;
* root.hints download/update;
* unbound-control-setup;


Extra
-----------

Behavior of handlers rule by variables:

* **unbound_enable**, *default is false* - enable service or not;
* **unbound_restart**, *default is false* - restart service after configuration
validation or not;

Other variables:

**unbound_restart_method**, *default is `reload`* - what to do when
`unbound_restart` set to `true` - `reload` or `restart`;

**unbound_use_ansible_managed**, *default is false* - set `{{ ansible_managed }}`
stamp or not;

**unbound_add_deployed_timestamp**, *default is false* - put real time of deploy
to generated configuration;

**unbound_root_hints_src**, *default is
`https://www.internic.net/domain/named.cache`* - the source of root hints. May
be http/https/ftp/file;

**unbound_root_hints_dest**, *default is `/etc/unbound/named.cache`*, destination
to download `unbound_root_hints_src`. Should be match with `root_hints` of
`unbound_server`;

**unbound_download_root_hints**, *default is true*:
 * `false` - do not download anythig;
 * `true` - download only when `unbound_root_hints_dest` does not exists (do not
update existing file);
 * `always` - download always;

**unbound_control_setup_dest**, *default is `/etc/unbound`*, destination for
`unbound-control-setup` certificates generation. Should be match with `key
files` of `unbound_control`;

**unbound_control_setup**, *default is true*:
 * `false` - do not run `unbound-control-setup`;
 * `true` - run `unbound-control-setup` only when no '\*.pem' files on
`unbound_control_setup_dest` is preset (do not update existing files);
 * `always` - always run `unbound_control_setup_dest`;

Example configuration
-------------------------

```yaml
---
unbound_enable: 'true'
unbound_restart: 'true'
unbound_restart_method: 'reload' # restart or reload
unbound_install_package: 'true'
unbound_use_ansible_managed: 'true'
unbound_add_deployed_timestamp: 'false'
unbound_root_hints_src: 'https://www.internic.net/domain/named.cache'
unbound_root_hints_dest: '/etc/unbound/named.cache'
unbound_download_root_hints: 'true' # true, false, always
unbound_control_setup: 'true' # true, false, always
unbound_control_setup_dest: '/etc/unbound'

unbound_server:
# If given, after binding the port the user privileges are dropped.
- username: 'unbound'
# Enable or disable systemd socket activation.
  use_systemd: 'yes'
# Enable or disable whether the unbound forks into the background as a daemon.
# Set the value to no when unbound runs as systemd service.
  do_daemonize: 'no'
# If given a chroot is done to the given directory.
# The default is "/etc/unbound". If you give "" no chroot is performed.
  chroot: '/etc/unbound'
# Sets the working directory for the program. Default is "/etc/unbound".
  directory: '/etc/unbound'
# The process id is written to the file. Default is "/run/unbound.pid".
  pidfile: '/run/unbound.pid'
# Read the root hints from this file. Default is nothing, using builtin hints
# for the IN class. The file has the format of zone files, with root nameserver
# names and addresses only. The default may become outdated, when servers
# change, therefore it is good practice to use a root-hints file.
  root_hints: '/etc/unbound/named.cache'
# If enabled id.server and hostname.bind queries are refused.
  hide_identity: 'no'
# If enabled version.server and version.bind queries are refused.
  hide_version: 'no'
# Set the identity to report. If set default ("") then the hostname of
# the server is returned.
  identity: ''
# Set the version to report. If set default ("") then the package version
# is returned.
  version: ''
# Level 0 means no verbosity, only errors.
# Level 1 gives operational information.
# Level 2 gives detailed operational information.
# Level 3 gives query level information, output per query.
# Level 4 gives algorithm level information.
# Level 5 logs client identification for cache misses.
  verbosity: '1'
# The number of seconds between printing statistics to the log for every thread.
# Disable with value 0 or  "". The histogram statistics are only printed if
# replies were sent during the statistics interval, requestlist statistics are
# printed for every interval (but can be 0). This is because the median
# calculation requires data to be present.
  statistics_interval: '3600'
# If enabled, statistics are cumulative since starting unbound, without clearing
# the statistics counters after logging the statistics.
  statistics_cumulative: 'no'
# If enabled, extended statistics are printed from unbound-control(8).
# Default is off, because keeping track of more statistics takes time.
  extended_statistics: 'no'
# The number of threads to create to serve clients. Use 1 for no threading.
  num_threads: '8'
# If "" is given, logging goes to stderr, or nowhere once daemonized.
  logfile: ''
# Sets unbound to send log messages to the syslogd, using syslog(3). The log
# facility LOG_DAEMON is used, with identity "unbound". The logfile setting is
# overridden when use-syslog is turned on.
  use_syslog: 'no'
# If "" is given, then the name of the executable, usually "unbound" is used to
# report to the log.
  log_identity: ''
# Sets logfile lines to use a timestamp in UTC ascii. Default is no, which
# prints the seconds since 1970 in brackets. No effect if using syslog, in that
# case syslog formats the timestamp printed into the log files.
  log_time_ascii: 'yes'
# Prints one line per query to the log, with the log timestamp and IP address,
# name, type and class. Note that it takes time to print these lines which makes
# the server (significantly) slower.
  log_queries: 'no'
# Prints one line per reply to the log, with the log timestamp and IP address,
# name, type, class, return code, time to resolve, from cache and response size.
# Note that it takes time to print these lines which makes the server
# (significantly) slower.
  log_replies: 'no'
# Detect source interface on UDP queries and copy them to replies. This feature
# is experimental, and needs support in your OS for particular socket options.
  interface_automatic: 'yes'
# The port number, default 53, on which the server responds to queries.
  port: '53'
# Number of ports to open. This number of file descriptors can be opened per
# thread. Must be at least 1. Larger numbers need extra resources from the
# operating system. For performance a very large value is best, use libevent to
# make this possible.
  outgoing_range: '4096'
# Permit unbound to open this port or range of ports for use to send queries.
# A larger number of permitted outgoing ports increases resilience against
# spoofing attempts. Make sure these ports are not needed by other daemons. By
# default only ports above 1024 that have not been assigned by IANA are used.
# Give a port number or a range of the form "low-high", without spaces.
  outgoing_port_permit: ''
# Do not permit unbound to open this port or range of ports for use to send
# queries. Use this to make sure unbound does not grab a port that another
# daemon needs. The port is avoided on all outgoing interfaces, both IP4 and IP6.
# By default only ports above 1024 that have not been assigned by IANA are used.
# Give a port number or a range of the form "low-high", without spaces.
  outgoing_port_avoid: '3200-3208'
# Number of outgoing TCP buffers to allocate per thread. Default is 10. If set
# to 0, or if do-tcp is "no", no TCP queries to authoritative servers are done.
# For larger installations increasing this value is a good idea.
  outgoing_num_tcp: '10'
# Number of incoming TCP buffers to allocate per thread. Default is 10. If set
# to 0, or if do-tcp is "no", no TCP queries from clients are accepted. For
# larger installations increasing this value is a good idea.
  incoming_num_tcp: '10'
# If not 0, then set the SO_RCVBUF socket option to get more buffer space on UDP
# port 53 incoming queries. So that short  spikes on busy servers do not drop
# packets (see counter in netstat -su). Default is 0 (use system value).
# Otherwise, the number of bytes to ask for, try "4m" on a busy server.
  so_rcvbuf: '4m'
# If not 0, then set the SO_SNDBUF socket option to get more buffer space on UDP
# port 53 outgoing queries. This for very busy servers handles spikes in answer
# traffic, otherwise 'send: resource temporarily unavailable' can get logged,
# the buffer overrun is also visible by netstat -su. Default is 0 (use system
# value). Specify the number of bytes to ask for, try "4m" on a very busy server.
  so_sndbuf: '4m'
# If yes, then open dedicated listening sockets for incoming queries for each
# thread and try to set the SO_REUSEPORT socket option on each socket. May
# distribute incoming queries to threads more evenly.
  so_reuseport: 'yes'
# If yes, then use IP_TRANSPARENT socket option on sockets where unbound is
# listening for incoming traffic. Default no. Allows you to bind to non-local
# interfaces. For example for non-existant IP addresses that are going to exist
# later on, with host failover configuration. This is a lot like
# interface-automatic, but that one services all interfaces and with this
# option you can select which (future) interfaces unbound provides service on.
  ip_transparent: 'no'
# If yes, then use IP_FREEBIND socket option on sockets where unbound is
# listening to incoming traffic. Default no. Allows you to bind to IP addresses
# that are nonlocal or do not exist, like when the network interface or IP
# address is down.
  ip_freebind: 'no'
# Number of bytes size to advertise as the EDNS reassembly buffer size. This is
# the value put into datagrams over UDP towards peers. The actual buffer size is
# determined by msg-buffer-size (both for TCP and UDP). Do not set higher than
# that value. Default is 4096 which is RFC recommended. If you have
# fragmentation reassembly problems, usually seen as timeouts, then a value of
# 1480 can fix it. Setting to 512 bypasses even the most stringent path MTU
# problems, but is seen as extreme, since the amount of TCP fallback generated
# is excessive (probably also for this resolver, consider tuning the outgoing
# tcp number).
  edns_buffer_size: '4096'
# Maximum UDP response size (not applied to TCP response). 65536 disables the
# udp response size maximum, and uses the choice from the client, always.
# Suggested values are 512 to 4096. Default is 4096.
  max_udp_size: '4096'
# Number of bytes size of the message buffers. Default is 65552 bytes, enough
# for 64 Kb packets, the maximum DNS message size. No message larger than this
# can be sent or received. Can be reduced to use less memory, but some requests
# for DNS data, such as for huge resource records, will result in a SERVFAIL
# reply to the client.
  msg_buffer_size: '65552'
# Number of bytes size of the message cache. Default is 4 megabytes. A plain
# number is in bytes, append 'k', 'm' or 'g'.
  msg_cache_size: '64m'
# Number of slabs in the message cache. Slabs reduce lock contention by threads.
# Must be set to a power of 2. Setting (close) to the number of cpus is a
# reasonable guess.
  msg_cache_slabs: '2'
# The number of queries that every thread will service simultaneously. If more
# queries arrive that need servicing, and no queries can be jostled out, then
# the queries are dropped. This forces the client to resend after a timeout,
# allowing the erver time to work on the existing queries.
  num_queries_per_thread: '1024'
# Timeout used when the server is very busy. Set to a value that usually results
# in one roundtrip to the authority servers. If too many queries arrive, then
# 50% of the queries are allowed to run to completion, and the other 50% are
# replaced with the new incoming query if they have already spent more than
# their allowed time. This protects against denial of service by slow queries or
# high query rates. Default 200 milliseconds. The effect is that the qps for
# long-lasting queries is about:
# (numqueriesperthread / 2) / (average time for such long queries) qps.
# The qps for short queries can be about:
# (numqueriesperthread / 2) / (jostletimeout inwhole seconds) qps per thread,
# about (1024/2)*5 = 2560 qps by default.
  jostle_timeout: '200'
# Extra delay for timeouted UDP ports before they are closed, in msec. Default
# is 0, and that disables it. This prevents very delayed answer packets from the
# upstream (recursive) servers from bouncing against closed ports and setting
# off all sort of close-port counters, with eg. 1500 msec. When timeouts happen
# you need extra sockets, it checks the ID and remote IP of packets, and
# unwanted packets are added to the unwanted packet counter.
  delay_close: '3000'
# Number of bytes size of the RRset cache. Default is 4 megabytes. A plain
# number is in bytes, append 'k', 'm' or 'g'.
  rrset_cache_size: '64m'
# Number of slabs in the RRset cache. Slabs reduce lock contention by threads.
# Must be set to a power of 2.
  rrset_cache_slabs: '2'
# Time to live minimum for RRsets and messages in the cache.
# If the minimum kicks in, the data is cached for longer than the domain owner
# intended, and thus less queries are made to look up the data. Zero makes sure
# the data in the cache is as the domain owner intended, higher values,
# especially more than an hour or so, can lead to trouble as the data in the
# cache does not match up with the actual data any more.
  cache_min_ttl: '120'
# Time to live maximum for RRsets and messages in the cache. Default is 86400
# seconds (1 day). If the maximum kicks in, responses to clients still get
# decrementing TTLs based on the original (larger) values. When the internal TTL
# expires, the cache item has expired. Can be set lower to force the resolver to
# query for data often, and not trust (very large) TTL values.
  cache_max_ttl: '86400'
# Time to live maximum for negative responses, these have a SOA in the authority
# section that is limited in time. Default is 3600.
  cache_max_negative_ttl: '3600'
# Time to live for entries in the host cache. The host cache contains roundtrip
# timing, lameness and EDNS support information.
  infra_host_ttl: '900'
# Number of slabs in the infrastructure cache. Slabs reduce lock contention by
# threads. Must be set to a power of 2.
  infra_cache_slabs: '2'
# Lower limit for dynamic retransmit timeout calculation in infrastructure cache.
# Default is 50 milliseconds. Increase this value if using forwarders needing
# more time to do recursive name resolution.
  infra_cache_min_rtt: '50'
# Number of hosts for which information is cached.
  infra_cache_numhosts: '10000'
# Define the tags that can be used with local-zone and access-control.
# Tags are whitespace separated.
  define_tag:
  - 'tag1'
  - 'tag2'
# Enable or disable whether ip4 queries are answered or issued.
  do_ip4: 'yes'
# Enable or disable whether ip6 queries are answered or issued. If disabled,
# queries are not answered on IPv6, and queries are not sent on IPv6 to the
# internet nameservers. With this option you can disable the ipv6 transport for
# sending DNS traffic, it does not impact the contents of the DNS traffic, which
# may have ip4 and ip6 addresses in it.
  do_ip6: 'no'
# If enabled, prefer IPv6 transport for sending DNS queries.
  prefer_ip6: 'no'
# Enable or disable whether UDP queries are answered or issued.
  do_udp: 'yes'
# Enable or disable whether TCP queries are answered or issued.
  do_tcp: 'yes'
# Maximum segment size (MSS) of TCP socket on which the server responds to
# queries. Value lower than common MSS on Ethernet will address path MTU problem.
# Note that not all platform supports socket option to set MSS (TCP_MAXSEG).
# Default (zero) is system default MSS determined by interface MTU and
# negotiation between server and client.
  tcp_mss: '0'
# Maximum segment size (MSS) of TCP socket for outgoing queries (from Unbound
# to other servers). Value lower than common MSS on Ethernet will address path
# MTU problem. Note that not all platform supports socket option to set
# MSS (TCP_MAXSEG). Default (zero) is system default MSS determined by interface
# MTU and negotiation between Unbound and other servers.
  outgoing_tcp_mss: '0'
# Enable or disable whether the upstream queries use TCP only for transport.
  tcp_upstream: 'no'
# If enabled (default is disabled), then for private address space, the reverse
# lookups are no longer filtered. This allows unbound when running as dns
# service on a host where it provides service for that host, to put out all of
# the queries for the 'lan' upstream. When enabled, only localhost, 127.0.0.1
# reverse and ::1 reverse zones are configured with default local zones.
# Disable the option when unbound is running as a (DHCP-) DNS network resolver
# for a group of machines, where such lookups should be filtered (RFC
# compliance), this also stops potential data leakage about the local network
# to the upstream DNS servers.
  unblock_lan_zones: 'no'

unbound_interfaces:
# Interface to use to connect to the network. This interface is listened to for
# queries from clients, and answers to clients are given from it. Can be given
# multiple times to work on several interfaces. If none are given the default is
# to listen to localhost. The interfaces are not changed on a reload but only on
# restart. A port number can be specified, if not specified the default port
# (from port) is used.
- listen_interfaces:
  - address: '0.0.0.0'
    port: '53'
# Interface to use to connect to the network. This interface is used to send
# queries to authoritative servers and receive their replies. Can be given
# multiple times to work on several interfaces. If none are given the default
# (all) is used. You can specify the same interfaces in interface: and
# outgoing-interface: lines, the interfaces are then used for both purposes.
# Outgoing queries are sent via a random outgoing interface to counter spoofing.
- outgoing_interfaces:
  - '0.0.0.0'

unbound_control:
# The netblock is given as an IP4 or IP6 address with /size appended for a
# classless network block. The most specific netblock match is used, if none
# match deny is used.
#
# deny - stops queries from hosts from that netblock.
#
# refuse - stops queries too, but sends a DNS rcode REFUSED error message back.
#
# allow - gives access to clients from that netblock. It gives only access for
# recursion clients (which is what almost all clients need). Nonrecursive
# queries are refused. Action does allow nonrecursive queries to access the
# local-data that is configured. The reason is that this does not involve the unbound server
# recursive lookup algorithm, and static data is served in the reply. This
# supports normal operations where nonrecursive queries are made for the
# authoritative data. For nonrecursive queries any replies from the dynamic cache
# are refused.
#
# allow_snoop gives nonrecursive access too. This give both recursive and non
# recursive access.
# The name allow_snoop refers to cache snooping, a technique to use nonrecursive
# queries to examine the cache contents (for malicious acts). However,
# nonrecursive queries can also be a valuable debugging tool (when you want to
# examine the cache contents). In that case use allow_snoop for your
# administration host.
#
# By default only localhost is allowed, the rest is refused. The default is
# refused, because that is protocol-friendly. The DNS protocol is not designed
# to handle dropped packets due to policy, and dropping may result in (possibly
# excessive) retried queries. The deny_non_local and refuse_non_local settings
# are for hosts that are only allowed to query for the authoritative local-data,
# they are not allowed full recursion but only the static data. With
# deny_non_local, messages that are disallowed are dropped, with refuse_non_local
# they receive error code REFUSED.

- access_control:
  - address: '192.168.0.0/16'
    action: 'allow'
  - address: '198.19.0.0/16'
    action: 'allow'
  - address: '172.16.0.0/12'
    action: 'allow'
  - address: '10.0.0.0/8'
    action: 'allow'
# Assign tags to access-control elements. Clients using this access control
# element use localzones that are tagged with one of these tags. If
# access-control-tag is configured for a netblock that does not have an
# access-control, an access-control element with action allow is configured for
# this netblock.
- access_control_tag:
  - address: '192.0.2.0/24'
    tags:
    - 'tag1'
    - 'tag2'
# Set action for particular tag for given access control element. If you have
# multiple tag values, the tag used to lookup the action is the first tag match
# between access-control-tag and local-zone-tag where "first" comes from the
# order of the define-tag values.
- access_control_tag_action:
  - address: '192.0.2.0/24'
    tag: 'tag2'
    action: 'refuse'
# Set redirect data for particular tag for given access control element.
- access_control_tag_data:
  - address: '192.0.2.0/24'
    tag: 'tag2'
    data: 'A 127.0.0.1'
# Set view for given access control element.
- access_control_view:
  - address: '192.0.2.0/24'
    view: 'view1'
  - address: '192.0.3.0/24'
    view: 'view2'

unbound_local_zones:
# Configure a local zone. The type determines the answer to give if there is no
# match from local-data.
# After that the default settings are listed. Use local-data: to enter data into
# the local zone. Answers for local zones are authoritative DNS answers. By
# default the zones are class IN. If you need more complicated authoritative
# data, with referrals, wildcards, CNAME/DNAME support, or DNSSEC authoritative
# service, setup a stub-zone.
#
# Action types:
#
# deny - Do not send an answer, drop the query. If there is a match from local
# data, the query is answered.
#
# refuse - Send an error message reply, with rcode REFUSED. If there is a match
# from local data, the query is answered.
#
# static - If there is a match from local data, the query is answered.
# Otherwise, the query is answered with nodata or nxdomain. For a negative
# answer a SOA is included in the answer if present as local-data for the zone
# apex domain.
#
# transparent - If there is a match from local data, the query is answered.
# Otherwise if the query has a different name, the query is resolved normally.
# If the query is for a name given in localdata but no such type of data is
# given in localdata, then a noerror nodata answer is returned. If no local-zone
# is given local-data causes a transparent zone to be created by default.
#
# typetransparent - If there is a match from local data, the query is answered.
# If the query is for a different name, or for the same name but for a
# different type, the query is resolved normally. So, similar to transparent but
# types that are not listed in local data are resolved normally, so if an A
# record is in the local data that does not cause a nodata reply for AAAA queries.
#
# redirect - The query is answered from the local data for the zone name. There
# may be no local data beneath the zone name. This answers queries for the zone,
# and all subdomains of the zone with the local data for the zone. It can be
# used to redirect a domain to return a different address record to the end user,
# with local-zone: "example.com." redirect
# local-data: "example.com. A 127.0.0.1"
# queries for www.example.com and www.foo.example.com are redirected, so that
# users with web browsers cannot access sites with suffix example.com.
#
# inform - The query is answered normally, same as transparent. The client IP
# address (@portnumber) is printed to the logfile. The log message is:
# timestamp, unbound-pid, info: zonename inform IP@port queryname type class.
# This option can be used for normal resolution, but machines looking up
# infected names are logged, eg. to run antivirus on them.
#
# inform_deny - The query is dropped, like 'deny', and logged, like 'inform'.
# Ie. find infected machines without answering the queries.
#
# always_transparent - Like transparent, but ignores local data and resolves
# normally.
#
# always_refuse - Like refuse, but ignores local data and refuses the query.
#
# always_nxdomain - Like static, but ignores local data and returns nxdomain
# for the query.
#
# nodefault - Used to turn off default contents for AS112 zones. The other types
# also turn off default contents for the zone. The 'nodefault' option has no
# other effect than turning off default contents for the given zone. Use
# nodefault if you use exactly that zone, if you want to use a subzone,
# use transparent.

- name: 'example.net'
  action: 'redirect'
# Configure local data, which is served in reply to queries for it. The query
# has to match exactly unless you configure the local-zone as redirect. If not
# matched exactly, the local-zone type determines further processing. If
# local-data is configured that is not a subdomain of a local-zone, a
# transparent local-zone is configured.
  data:
  - resource: 'example.net'
    type: 'A'
    answer: '192.168.5.1'
  - resource: 'example.net'
    type: 'MX 10' # MX record should have weight!
    answer: '192.168.5.2'
# Configure local data shorthand for a PTR record with the reversed IPv4 or IPv6
# address and the host name.
  data_ptr:
  - resource: '192.168.5.1'
    ttl: '7200'
    answer: 'www.example.net'
  - resource: '192.168.5.2'
    answer: 'refused.example.net'
- name: 'example.org'
  action: 'redirect'
  data:
  - resource: 'example.org'
    type: 'A'
    answer: '192.168.3.1'
  - resource: 'example.org'
    type: 'TXT'
    answer: 'supa_scoopa'
# Assign tags to localzones. Tagged localzones will only be applied when the
# used access-control element has a matching tag. Tags are whitespace separated.
  tags:
  - 'tag1'
  - 'tag2'

unbound_forward_zones:
# For the forward zone this list of nameservers is used to forward the queries
# to. The servers listed as forward-host and forward-addr have to handle further
# recursion for the query. Thus, those servers are not authority servers, but
# are (just like unbound is) recursive servers too; unbound does not perform
# recursion itself for the forward zone, it lets the remote server do it.
# Class IN is assumed.
- name: 'example.local'
# IP address of server to forward to. Can be IP 4 or IP 6.
  forward_address:
  - address: '192.168.1.1'
    port: '53'
  - address: '192.168.2.1'
# If enabled, a query is attempted without the forward clause if it fails. The
# data could not be retrieved and would have caused SERVFAIL because the servers
# are unreachable, instead it is tried without this clause.
  forward_first: 'no'
# Enabled or disable whether the queries to this forwarder use SSL for transport.
  forward_ssl_upstream: 'no'
- name: 'google.com'
# Name of server to forward to. Is itself resolved before it is used.
  forward_host:
  - host: 'ns1.google.com'
    port: '53'
  - host: 'ns2.google.com'
- name: 'opentech.ru'
  forward_address:
  - address: '193.150.124.1'

unbound_stub_zones:
# There may be multiple stub-zone: clauses. Each with a name: and zero or more
# hostnames or IP addresses. For the stub zone this list of nameservers is used.
# Class IN is assumed. The servers should be authority servers, not recursors.
# unbound performs the recursive processing itself for stub zones.
# The stub zone can be used to configure authoritative data to be used by the
# resolver that cannot be accessed using the public internet servers. This is
# useful for company-local data or private zones. Setup an authoritative server
# on a different host (or different port). The unbound resolver can then access
# the data, without referring to the public internet for it.
# This setup allows DNSSEC signed zones to be served by that authoritative
# server, in which case a trusted key entry with the public key can be put in
# config, so that unbound can validate the data and set the AD bit on replies
# for the private zone (authoritative servers do not set the AD bit). This setup
# makes unbound capable of answering queries for the private zone, and can even
# set the AD bit ('authentic'), but the AA ('authoritative') bit is not set on
# these replies.
# Consider adding server: statements for domain-insecure: and for local-zone:
# name nodefault for the zone if it is a locally served zone. The insecure
# clause stops DNSSEC from invalidating the zone. The local zone nodefault
# (or transparent) clause makes the (reverse-) zone bypass unbound's filtering
# of RFC1918 zones.

# Name of the stub zone.
- name: 'example.local'
  stub_address:
  - address: '192.168.1.1'
    port: '53'
  - address: '192.168.2.1'
# If enabled it performs NS set priming, which is similar to root hints, where
# it starts using the list of nameservers currently published by the zone. Thus,
# if the hint list is slightly outdated, the resolver picks up a correct list
# online.
  stub_prime: 'no'
# If enabled, a query is attempted without the stub clause if it fails. The data
# could not be retrieved and would have caused SERVFAIL because the servers are
# unreachable, instead it is tried without this clause.
  stub_first: 'no'
# Enabled or disable whether the queries to this stub use SSL for transport.
  stub_ssl_upstream: 'no'
- name: 'google.com'
# Name of stub zone nameserver. Is itself resolved before it is used.
  stub_host:
  - host: 'ns1.google.com'
    port: '53'
  - host: 'ns2.google.com'
- name: 'opentech.ru'
  stub_address:
  - address: '193.150.124.1'

unbound_views:
# Name of the view. Must be unique. This name is used in access-control-view
# elements.
- name: 'view1'
  zones:
# View specific local-zone elements. Has the same types and behaviour as the
# global local-zone elements.
  - name: 'example.com'
    action: 'redirect'
# View specific local-data elements. Has the same behaviour as the global
# local-data elements.
    data:
    - resource: 'example.com'
      type: 'A'
      answer: '192.168.5.1'
    - resource: 'example.com'
      type: 'MX 10'
      answer: '192.168.5.2'
# View specific local-data-ptr elements. Has the same behaviour as the global
# local-data-ptr elements.
    data_ptr:
    - resource: '192.168.5.1'
      ttl: '7200'
      answer: 'www.example.com'
    - resource: '192.168.5.2'
      answer: 'refused.example.com'
# If enabled, it attempts to use the global local-zone and local-data if there
# is no match in the view specific options.
    view_first: 'no'
  - name: 'refused.example.com'
    action: 'refuse'
- name: 'view2'
  zones:
  - name: 'example.org'
    action: 'redirect'
    data:
    - resource: 'example.org'
      type: 'A'
      answer: '192.168.3.1'
    - resource: 'example.org'
      type: 'MX 20'
      answer: '192.168.3.2'
    data_ptr:
    - resource: '192.168.3.1'
      ttl: '7200'
      answer: 'www.example.org'
    - resource: '192.168.3.2'
      answer: 'refused.example.org'
    view_first: 'no'
  - name: 'refused.example.org'
    action: 'refuse'

# If this is enabled, the unbound-control(8) utility can be used to send
# commands to the running unbound server. The server uses these clauses to setup
# SSLv3 / TLSv1 security for the connection. The unbound-control(8) utility also
# reads the remote-control section for options. To setup the correct self-signed
# certificates use the unbound-control-setup(8) utility.
unbound_remote_control:
# The option is used to enable remote control. If turned off, the server does
# not listen for control commands.
- enabled: 'yes'
# Give IPv4 or IPv6 addresses or local socket path to listen on for control
# commands. By default localhost (127.0.0.1 and ::1) is listened to.
# Use 0.0.0.0 and ::0 to listen to all interfaces. If you change this and
# permissions have been dropped, you must restart the server for the change to
# take effect.
  control_interfaces:
  - '127.0.0.1'
# The port number to listen on for IPv4 or IPv6 control interfaces, default is
# 8953. If you change this and permissions have been dropped, you must restart
# the server for the change to take effect.
  control_port: '8953'
# Whether to require certificate authentication of control connections. This
# should not be changed unless there are other mechanisms in place to prevent
# untrusted users from accessing the remote control interface.
  control_use_cert: 'yes'
# Path to the server private key, by default unbound_server.key. This file is
# generated by the unbound-control-setup utility. This file is used by the
# unbound server, but not by unbound-control.
  server_key_file: '/etc/unbound/unbound_server.key'
# Path to the server self signed certificate, by default unbound_server.pem.
# This file is generated by the unbound-control-setup utility. This file is used
# by the unbound server, and also by unbound-control.
  server_cert_file: '/etc/unbound/unbound_server.pem'
# Path to the control client private key, by default unbound_control.key. This
# file is generated by the unbound-control-setup utility. This file is used by
# unbound-control.
  control_key_file: '/etc/unbound/unbound_control.key'
# Path to the control client certificate, by default unbound_control.pem. This
# certificate has to be signed with the server certificate. This file is
# generated by the unbound-control-setup utility. This file is used by
# unbound-control.
  control_cert_file: '/etc/unbound/unbound_control.pem'
```

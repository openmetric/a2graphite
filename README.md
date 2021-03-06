Any Metric to Graphite convertor.

This tool is used to receive metrics sent via any other common protocols, and convert to graphite protocol, and send to backend carbon server.
Currently, only ceilometer receiver is implemented (receive message sent by ceilometer udp publisher).

## Config file

```yaml
log:
  # default log file is "/dev/stdout"
  filename: "./a2graphite.log"
  # default level is  "info", available levels: debug, info, notice, warning, error, critical
  level: "info"

# settings for graphite backend
graphite:
  # graphite url, valid forms:
  # - tcp://127.0.0.1:2003
  # - udp://127.0.0.1:2004
  # - pickle://example.com:2005  
  # - 127.0.0.1:2003    # protocol defaults to "tcp" if omitted
  # - tcp://example.com # port defaults to "2003" if omitted
  # - example.com       # if both protocol and ports are omitted, use tcp://${url}:2003
  url: "tcp://127.0.0.1:2003"
  # all metrics' names are prefixed with "${prefix}" before sent out
  # if prefix contains "{host}", "{host}" would be replace by hostname
  prefix: ""
  # delay time before reconnect on connection failure
  reconnect_delay: 100ms
  # receiver generated metrics are buffered, if network is fast enough to send
  # all metrics, this buffer does not need to be too large
  buffer_size: 100

# internal stats
stats:
  # whether enable internal statistics
  enabled: true
  # interval of internal statistics
  interval: 60s
  # the following settings are same as graphite section, you can specify a different
  # prefix or even different graphite server for internal stats
  url: tcp://127.0.0.1:2003
  reconnect_delay: 100ms
  prefix: "a2graphite.{host}.stats."
  buffer_size: 100

# whether to enable profiler
profiler:
  enabled: false
  listen_addr: "127.0.0.1:6060"

# config for ceilometer receiver
ceilometer:
  enabled: true
  # you can listen on more than one ports or addresses
  listen_addrs:
    - ":4952"
  # received messages are pushed to a buffer, and consumed by multiple worker routines.
  # if workers are fast enough to consume messages, this buffer does not need to be too large.
  buffer_size: 100
  # number of decoding and converting workers, increase this if buffer is full and you have free CPUs.
  workers: 4
  # rules for convert ceilometer message to graphite metrics.
  # no default rules are provided, you must write your own. you can take example-config.yml for reference.
  # ${rules} is a map, map key is ceilometer message's $CounterName, and map value is graphite metric's name.
  # these substitutions are supported: {InstanceID}, {MountPoint}, {DiskName}, {VnicName}
  rules:
    "disk.allocation": "disk.overall.size.allocation"
    "disk.free": "disk.mount-{MountPoint}.size.free"
    "disk.device.capacity": "disk.device-{DiskName}.size.capacity"
    "network.incoming.bytes.rate": "network.{VnicName}.bytes.incoming"
  prefix: "ceilometer.{InstanceID}."
```

## Install and run

```sh
go get github.com/openmetric/a2graphite
a2graphite -config /path/to/config.yml
```

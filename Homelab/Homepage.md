- greeting:
    text_size: xl
    text: Greeting Text
- openmeteo:
    label: Kyiv # optional
    latitude: 50.449684
    longitude: 30.525026
    timezone: Europe/Kiev # optional
    units: metric # or imperial
    cache: 5 # Time in minutes to cache API responses, to stay within limits
    format: # optional, Intl.NumberFormat options
      maximumFractionDigits: 1
- logo:
    icon: https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/I_Love_New_York.svg/1101px-I_Love_New_York.svg.png # optional

- glances:
    url: http://host.or.ip:port
    username: user # optional if auth enabled in Glances
    password: pass # optional if auth enabled in Glances
    version: 4 # required only if running glances v4 or higher, defaults to 3
    cpu: true # optional, enabled by default, disable by setting to false
    mem: true # optional, enabled by default, disable by setting to false
    cputemp: true # disabled by default
    cpuSensorLabel: Package id # optional additional cputemp sensor label prefix
    unit: imperial # optional for temp, default is metric
    uptime: true # disabled by default
    disk: / # disabled by default, use mount point of disk(s) in glances. Can also be a list (see below)
    diskUnits: bytes # optional, bytes (default) or bbytes. Only applies to disk
    expanded: true # show the expanded view
    label: MyMachine # optional
- datetime:
    text_size: xl
    format:
      timeStyle: short
widget:
  type: uptimekuma
  url: http://uptimekuma.host.or.ip:port
  slug: statuspageslug

widget:
  type: tailscale
  deviceid: deviceid
  key: tailscalekey
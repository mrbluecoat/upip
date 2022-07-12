# upip
A lite version of Firehol's [update-ipsets](https://github.com/firehol/firehol/blob/master/sbin/update-ipsets) tool

[Firehol](https://github.com/firehol/firehol) and [blocklist-ipsets](https://github.com/firehol/blocklist-ipsets) are really amazing but can be difficult to set up for first time users. `upip` is designed to make that process easier. Here's a quickstart:

```
# Debian/Ubuntu:
sudo apt-get install firehol jq
# RedHat:
#sudo yum install firehol jq

cat > /etc/firehol/whitelist4.netset <<EOF
224.0.0.0/4
240.0.0.0/4
10.0.0.0/8
172.16.0.0/12
169.254.0.0/16
192.168.0.0/16
127.0.0.0/24
EOF

cat > /etc/firehol/whitelist6.netset <<EOF
::1/128
fc00::/7
EOF

# interface to bind to (usually your WAN ethernet but in some cases your LAN ethernet
INT="eth0"

# configure Firehol
cp /etc/firehol/firehol.conf /etc/firehol/firehol.conf.original
cat > /etc/firehol/firehol.conf <<EOF
version 6

# whitelists
ipset4 create whitelist4 hash:net
ipset4 addfile whitelist4 /etc/firehol/whitelist4.netset
ipset6 create whitelist6 hash:net
ipset6 addfile whitelist6 /etc/firehol/whitelist6.netset

# blacklists go here

interface ${INT} myint
    policy accept
EOF

# enable Firehol
sed -i "s/START_FIREHOL=.*/START_FIREHOL=YES/" /etc/default/firehol
sed -i "s/WAIT_FOR_IFACE=.*/WAIT_FOR_IFACE=\"${INT}\"/" /etc/default/firehol

# start Firehol
systemctl restart firehol
systemctl status firehol

# install upip
sudo curl -sLo /usr/local/bin/upip https://raw.githubusercontent.com/mrbluecoat/upip/main/upip
sudo chmod +x /usr/local/bin/upip

# initialize upip (one-time setup)
sudo upip init
```

**IMPORTANT** `# blacklists go here` must exist in your *firehol.conf* file below the whitelist section and above the `interface` section

Now that you have a basic Firehol installation and upip installed, activating blacklists is as easy as 

`sudo upip add firehol_level1` or `sudo upip add dm_tor et_tor yoyo_adservers`

You can view installed lists via `sudo upip list`

You can remove lists via `sudo upip remove dm_tor et_tor`

You can add `upip update` to cron to regularly check for remote updates and install them. It will only download and install if the remote list has been updated.

`upip` supports any list from [iplists.firehol.org](https://iplists.firehol.org/) that has a public *local copy* link. Note: private lists require [extra setup](https://github.com/firehol/firehol/wiki/dnsbl-ipset.sh).

You can even add a custom list not hosted at [iplists.firehol.org](https://iplists.firehol.org/). For example: 

`upip add doh4.ipset https://raw.githubusercontent.com/dibdot/DoH-IP-blocklists/master/doh-ipv4.txt`

While `upip` doesn't have the advanced geo blocking of [update-ipsets](https://github.com/firehol/firehol/blob/master/sbin/update-ipsets), it does support IPv6: 

`upip add doh6.ipset https://raw.githubusercontent.com/dibdot/DoH-IP-blocklists/master/doh-ipv6.txt`

```
Usage: upip [OPTION]

Options:
  help                  this document
  init                  initialize local data from https://iplists.firehol.org/
  list                  show all installed and available lists
  add [NAME(s)]         add an official list to the local Firehol instance
  add [FILENAME] [URL]  add a custom public list to the local Firehol instance
  remove [NAME(s)]      remove a list from the local Firehol instance
  update                refresh local lists with latest version from external sources
```

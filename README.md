# Nftables geoip script

Python script that generates .nft files with mappings betwween IP addresses
and its geolocation, so you can include them inside your rules.

# Requirements

Package used that are not present in the Python Standard Library

- `requests`

# Usage example

To generate ipv4 and ipv6 mappings, download geoip data from db-ip.com
saving the output in the current folder

```
   ./nft_geoip.py --file-location location.csv --download
```

Giving the following output

```
  drwxr-xr-x 2 foobar foobar 4,0K ene  4 19:38 .
  drwxr-xr-x 5 foobar foobar 4,0K ene  4 19:38 ..
  -rw-r--r-- 1 foobar foobar  22M ene  4 19:38 dbip.csv
  -rw-r--r-- 1 foobar foobar  956 ene  4 19:38 geoip-def-africa.nft
  -rw-r--r-- 1 foobar foobar 8,3K ene  4 19:38 geoip-def-all.nft
  -rw-r--r-- 1 foobar foobar  902 ene  4 19:38 geoip-def-americas.nft
  -rw-r--r-- 1 foobar foobar   15 ene  4 19:38 geoip-def-antarctica.nft
  -rw-r--r-- 1 foobar foobar  808 ene  4 19:38 geoip-def-asia.nft
  -rw-r--r-- 1 foobar foobar  810 ene  4 19:38 geoip-def-europe.nft
  -rw-r--r-- 1 foobar foobar  461 ene  4 19:38 geoip-def-oceania.nft
  -rw-r--r-- 1 foobar foobar 8,8M ene  4 19:38 geoip-ipv4.nft
  -rw-r--r-- 1 foobar foobar  16M ene  4 19:38 geoip-ipv6.nft
```

* geoip-def-all defines variables for each country (2-char iso name)
  in location.csv assigning its iso numeric value (eg. Canada, `define $CA = 124`)
* geoip-ipv4.nft defines geoip4 map (@geoip4)
* geoip-ipv6.nft defines geoip6 map (@geoip6)

## Marking packets to its corresponding country

```
  meta mark set ip saddr map @geoip4
```

## Example: Counting incoming Spanish traffic

```
  table filter {
    include "./geoip-def-all.nft"
    include "./geoip-ipv4.nft"
    include "./geoip-ipv6.nft"

    chain input {
                  type filter hook input priority filter; policy accept;
                  meta mark set ip saddr map @geoip4
                  meta mark $ES counter
          }
  }
```

# Caveats

__It is not possible to use the country definitions inside an interactive
nft shell.__

So, although maps are defined, country definitions do not persist.
You will need to define those geoip rules inside your ruleset definition
files. Like in the example above.

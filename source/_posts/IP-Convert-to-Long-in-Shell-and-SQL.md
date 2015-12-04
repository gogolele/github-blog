title: IP Convert to Long in Shell and SQL
date: 2015-10-01 13:35:52
categories: dev
tags: [awk]
---
## Use AWK
for this file, change ip long to ip address.
```txt
34603008,34611711,"Akamai Technologies"
34611968,34613503,"Akamai Technologies"
34613760,34619647,"Akamai Technologies"
34619904,34641919,"Akamai Technologies"
34642176,34648063,"Akamai Technologies"
34648320,34656255,"Akamai Technologies"
34657280,34732031,"Akamai Technologies"
34733056,34824191,"Akamai Technologies"
34828288,34848767,"Akamai Technologies"
34852864,34876927,"Akamai Technologies"
```

use this shell
```shell
cat GeoIPOrg.csv | grep -i akamai | awk -F "," -v OFS="," 'function long2ip(a){ return int(a / 16777216) "." int(a % 16777216 / 65536) "." int(a % 65536 / 256) "." int(a % 256) }{ print $1,$2,long2ip($1), long2ip($2),$3}'
```

## Use SQL
IP Address to need to convert to Long. use this SQL
```SQL
select distinct ip,(cast(PARSENAME(ip,4) as bigint) * 16777216) + (cast(PARSENAME(ip,3) as bigint) * 65536) + (cast(PARSENAME(ip,2) as bigint) * 256) + cast(PARSENAME(ip,1) as bigint)  as iplong.
```

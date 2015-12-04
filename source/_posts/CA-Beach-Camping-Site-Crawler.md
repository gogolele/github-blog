title: CA Beach Camping Site Crawler
date: 2015-09-22 15:36:22
categories: [life, dev]
tags: [shell, crawler]
---

Everytime we want to order a camping site, we need to
1. go to http://www.reserveamerica.com/
2. input the destination
3. select a date, and length of stay
4. check if there are avaiable sites.


Now i wrote a script to check the availability of the beach sites in Southern California.

the 10 camping sites list, you know they are most popular:
```shell
$> touch sites.data
$> cat sites.data
120047 leo-carrillo-sb
120083 san-elijo-sb
120030 el-capitan-sb
120072 point-mugu-sp
120025 doheny-sb
123400 crystal-cove-sp-moro-campground
120015 carpinteria-sb
120075 refugio-sb
120082 san-clemente-sb
120059 morro-strand-sb
```

the shell script to support check the availability of a specific site on a specific date
```shell
#!/bin/bash
campingsitename=$1
campingsiteid=$2
datestr=$3
curl -v --silent -X POST -H "Content-Type: application/x-www-form-urlencoded" -H "User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/45.0.2454.93 Safari/537.36" -H "Cookie: __gads=ID=fc8ebd44da692630:T=1441151421:S=ALNI_MYrxbgiHCgJZfqWAGrU0wGybjgv3A; JSESSIONID=79F8EE5E92C62133E187CD7B16ED03E9.web05-ny; _ga=GA1.2.1844483365.1441151397; s_sess=%20s_cc%3Dtrue%3B%20SC_LINKS%3D%3B%20s_sq%3D%3B; _gat=1; utag_main=_st:1442959116539$ses_id:1442956639131%3Bexp-session; NSC_QSPE-VXQ-IUUQ=ffffffff09293c2a45525d5f4f58455e445a4a4225b9" -H "Referer:http://www.reserveamerica.com/camping/$campingsitename/r/campgroundDetails.do?contractCode=CA&parkId=$campingsiteid" -d "contractCode=CA&parkId=$campingsiteid&siteTypeFilter=ALL&availStatus=&submitSiteForm=true&search=site&campingDate=$datestr&lengthOfStay=1&campingDateFlex=&currentMaximumWindow=12&contractDefaultMaxWindow=MS%3A24%2CLT%3A18%2CGA%3A24%2CSC%3A13&stateDefaultMaxWindow=MS%3A24%2CGA%3A24%2CSC%3A13&defaultMaximumWindow=12&loop=&siteCode=&lookingFor=&camping_2001_3013=&camping_2001_218=&camping_2002_3013=&camping_2002_218=&camping_2003_3012=&camping_3100_3012=&camping_10001_3012=&camping_10001_218=&camping_3101_3012=&camping_3101_218=&camping_9002_3012=&camping_9002_3013=&camping_9002_218=&camping_9001_3012=&camping_9001_218=&camping_3001_3013=&camping_2004_3013=&camping_2004_3012=&camping_3102_3012=" http://www.reserveamerica.com/camping/$campingsitename/r/campgroundDetails.do?contractCode=CA&parkId=$campingsiteid
```

the shell script to do the availability check for Saturday in future 2 weeks
```shell
#!/bin/bash
datestr=$(date -d "next saturday" | awk -v OFS="+"  '{print $1,$2,$3,$6}')
nextsat=$(date -d "next saturday")
nextsat2=$(date -d "$nextsat+7 days")
datestr2=$(echo $nextsat2 | awk -v OFS="+" '{print $1,$2,$3,$6}')
 cat sites.data | while read line
 do
    siteid=$(echo $line | awk '{print $1}')
    sitename=$(echo $line | awk '{print $2}')
    ivt=$( sh curl-camping.sh $sitename $siteid $datestr  2>&1 | grep  '>STANDARD (' | sed "s/.*(\([0-9]*\)).*/\1/g" | uniq)
    sleep 2
    ivt=$( sh curl-camping.sh $sitename $siteid $datestr  2>&1 | grep  '>STANDARD (' | sed "s/.*(\([0-9]*\)).*/\1/g" | uniq)
    echo "site="$sitename",date="$datestr",inventory="$ivt
    sleep 10
 done


 cat sites.data | while read line
 do
    siteid=$(echo $line | awk '{print $1}')
    sitename=$(echo $line | awk '{print $2}')
    ivt=$( sh curl-camping.sh $sitename $siteid $datestr2  2>&1 | grep  '>STANDARD (' | sed "s/.*(\([0-9]*\)).*/\1/g" | uniq)
    sleep 2
    ivt=$( sh curl-camping.sh $sitename $siteid $datestr2  2>&1 | grep  '>STANDARD (' | sed "s/.*(\([0-9]*\)).*/\1/g" | uniq)
    echo "site="$sitename",date="$datestr2",inventory="$ivt
    sleep 10
 done
```

the result
```shell
site=leo-carrillo-sb,date=Sat+Sep+26+2015,inventory=0
site=san-elijo-sb,date=Sat+Sep+26+2015,inventory=0
site=el-capitan-sb,date=Sat+Sep+26+2015,inventory=1
site=point-mugu-sp,date=Sat+Sep+26+2015,inventory=0
site=doheny-sb,date=Sat+Sep+26+2015,inventory=0
site=crystal-cove-sp-moro-campground,date=Sat+Sep+26+2015,inventory=0
site=carpinteria-sb,date=Sat+Sep+26+2015,inventory=0
site=refugio-sb,date=Sat+Sep+26+2015,inventory=0
site=san-clemente-sb,date=Sat+Sep+26+2015,inventory=0
site=morro-strand-sb,date=Sat+Sep+26+2015,inventory=0
site=leo-carrillo-sb,date=Sat+Oct+3+2015,inventory=0
site=san-elijo-sb,date=Sat+Oct+3+2015,inventory=0
site=el-capitan-sb,date=Sat+Oct+3+2015,inventory=0
site=point-mugu-sp,date=Sat+Oct+3+2015,inventory=0
site=doheny-sb,date=Sat+Oct+3+2015,inventory=0
site=crystal-cove-sp-moro-campground,date=Sat+Oct+3+2015,inventory=0
site=carpinteria-sb,date=Sat+Oct+3+2015,inventory=0
site=refugio-sb,date=Sat+Oct+3+2015,inventory=0
site=san-clemente-sb,date=Sat+Oct+3+2015,inventory=0
site=morro-strand-sb,date=Sat+Oct+3+2015,inventory=0
```

in conclusion, the scripts list
```shell
.
├── camping-hunter.sh
├── curl-camping.sh
└── sites.data
```

to run the job

```shell
./camping-hunter.sh
```

interesting. :)

-EOF-

TAO WU@CA, USA

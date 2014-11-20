FAA NASR
========

This repository includes some tools to fetch, convert and compare the FAA's NASR data files

https://nfdc.faa.gov/xwiki/bin/view/NFDC/56+Day+NASR+Subscription

Requirements
------------

-	Ruby 1.9.3+
-	Node.js 0.10+

Setup
-----

```
bundle install
npm install -g shapefile
npm install -g topojson
```

To download the latest NASR data set, and convert NASR `.shp` Shapefiles into JSON:

```
rake
```

This command will only take the time to download the 80+ Mb NASR subscription data if you haven't already downloaded it into the current folder.

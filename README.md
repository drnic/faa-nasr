FAA NASR
========

This repository includes some tools to fetch, convert and compare the FAA's NASR data files

https://nfdc.faa.gov/xwiki/bin/view/NFDC/56+Day+NASR+Subscription

Outputs:

-	https://gist.github.com/drnic/76c8545a7797cb3dabe1 - Airspace changes since September 18, 2014 (comparing last two NASR subscriptions)

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

Tasks
-----

If you have the latest subscription and the previous subscription, you can determine if any of the Airspace Classes have changed (Bravo, Charlie and Delta):

```
$ rake shapefile:diff
Class B: no changes

Class C: no changes

Class D, additional:

* KS, WICHITA, WICHITA, MCCONNELL AFB CLASS D

Class D, updated:

* AK, ANCHORAGE, ANCHORAGE, BRYANT AAF CLASS D
* AK, BETHEL, BETHEL CLASS D
* AK, BETHEL, BETHEL CLASS D
* AK, JUNEAU, JUNEAU CLASS D
...
```

To share the diff via GitHub Gist:

```
brew install gist
gist --login
rake shapefile:diff | gist -f airspace-changes.md -f "Airspace changes since CHANGEDATE (comparing last two NASR subscriptions from https://nfdc.faa.gov/xwiki/bin/view/NFDC/56+Day+NASR+Subscription)"
```

Then update the URL at top of README.

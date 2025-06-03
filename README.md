# OpenMapChest map build guide

**The mkgmap style file in this repo is not up-to-date with the style file used to make maps currently available on OpenMapChest. You may see some differences in your maps compared to the pre-compiled versions.**

Preface: I kindly ask that you don't use the OpenMapChest name in any of the maps you create with this guide. The OpenMapChest name is used exclusively for the maps distributed on https://www.openmapchest.org.

This guide should work for all of Geofabrik's region extracts. The only caveat is that
gmapsupp.img files for SD cards are limited to 4Gb, and as such, it's not possible to
make gmappsup.img maps from the larger regions. "gmap" maps for BaseCamp don't have this
limitation and should work for the larger regions. To make this guide easier to follow,
the instructions are listed for united-states-northeast.

## Data needed for the build

### Supplemental data
* http://osm.thkukuk.de/data/sea-latest.zip
* http://osm.thkukuk.de/data/bounds-latest.zip
* https://download.geonames.org/export/dump/cities1000.zip

### Map data
* https://download.geofabrik.de/north-america/us-northeast-latest.osm.pbf

**NOTE** Other Geofabrik regions work as well.

## Prepare TIGER data for better address search - US only, optional

This step is optional but does produce better results for the address search in US maps.

### TIGER: Software
* https://github.com/linuxrocks123/tiger_versus_python
* https://wiki.openstreetmap.org/wiki/Osmosis
* https://gitlab.com/osm-c-tools/osmctools

### TIGER: Data
* https://download.geofabrik.de/north-america/us-northeast.poly
* https://www.nominatim.org/data/tiger2024-nominatim-preprocessed.csv.tar.gz

### TIGER: Procedure
* Untar tiger2024-nominatim-preprocessed.csv.tar.gz:
```
tar zxf tiger2024-nominatim-preprocessed.csv.tar.gz
```

* Convert the nominatim TIGER CSV to an OSC file:
```
cd tiger
cat *.csv | ../tiger_versus_python/tiger_versus_python.py > usa-tiger-addresses-2024.osc
```

* Merge address OSC file into the OSM data:
```
osmosis --read-xml-change file=<path to>/usa-tiger-addresses-2024.osc --read-pbf-fast \
  file=united-states-northeast-latest.pbf workers=3 --apply-change --write-pbf \
  omitmetadata=true file=<path to>/united-states-northeast-latest-addresses-merged.osm.pbf
```

* Trim off the address data outside of the area:
```
osmconvert <path to>/united-states-northeast-latest-addresses-merged.osm.pbf \
  -B=<path to>/united-states-northeast.poly --complete-ways --complete-multipolygons \
  --complete-boundaries -o=united-states-northeast-latest.pbf
```

## Building the Garmin map

### Software

https://www.mkgmap.org.uk/download/splitter.html

https://www.mkgmap.org.uk/download/mkgmap.html

### Procedure

* splitter command to split the map into smaller pieces:
```
java -jar <path to>/splitter.jar --geonames-file=<path to>/cities1000.zip \
  --output=o5m --mapid=<8 digits> --wanted-admin-level=8 --status-freq=120 \
  --keep-complete --precomp-sea=<path to>/sea-latest.zip --search-limit=400000 \
  --polygon-file=<path to>/united-states-northeast.poly \
  united-states-northeast.osm.pbf
```

* mkgmap command to build gmapsupp.img for SD Cards:
```
java -jar <path to>/mkgmap.jar --latin1 --gmapsupp --index \
  --location-autofill=bounds,is_in,nearest --route --housenumbers --x-name-service-roads=5 \
  --report-roundabout-issues=all --fix-roundabout-direction --add-pois-to-areas \
  --pois-to-areas-placement="entrance=main;entrance=yes;building=entrance" --link-pois-to-ways \
  --process-destination --process-exits --order-by-decreasing-area --family-id=<5 digits> \
  --product-id=1 --family-name="United States" \
  --series-name="United States <YYYY.MM.DD> US Northeast" \
  --product-version=<4 digits> --overview-mapname=usneast --overview-mapnumber=<family id 5 digits>000 \
  --bounds=<path to>/bounds-latest.zip --precomp-sea=<path to>/sea-latest.zip \
  --make-opposite-cycleways --road-name-config=<path to>/roadNameConfig.txt \
  --improve-overview --merge-lines --allow-reverse-merge --polygon-size-limits=24:12,18:10,16:8 \
  --ignore-fixme-values --drive-on=right --add-boundary-nodes-at-admin-boundaries=4 \
  --style-file=<path to>/omc-mkgmap-style -c template.args \
  --description="United States <YYYY.MM.DD> US Northeast" \
  <path to>/omc-typ.txt
```

* mkgmap command to build gmap file for BaseCamp:
```
java -jar <path to>/mkgmap.jar --latin1 --tdbfile --gmapi --index \
  --location-autofill=bounds,is_in,nearest --route --housenumbers --x-name-service-roads=5 \
  --report-roundabout-issues=all --fix-roundabout-direction --add-pois-to-areas \
  --pois-to-areas-placement="entrance=main;entrance=yes;building=entrance" --link-pois-to-ways \
  --process-destination --process-exits --order-by-decreasing-area --family-id=<5 digits> \
  --product-id=1 --family-name="United States" \
  --series-name="United States <YYYY.MM.DD> US Northeast" \
  --product-version=<4 digits> --overview-mapname=usneast --overview-mapnumber=<family id 5 digits>000 \
  --bounds=<path to>/bounds-latest.zip --precomp-sea=<path to>/sea-latest.zip \
  --make-opposite-cycleways --road-name-config=<path to>/roadNameConfig.txt \
  --improve-overview --merge-lines --allow-reverse-merge --polygon-size-limits=24:12,18:10,16:8 \
  --ignore-fixme-values --drive-on=right --add-boundary-nodes-at-admin-boundaries=4 \
  --style-file=<path to>/omc-mkgmap-style -c template.args \
  --description="United States <YYYY.MM.DD> US Northeast" \
  <path to>/omc-typ.txt
```

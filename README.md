# OpenMapChest map build guide

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

## Prepare TIGER data for better address search in US maps

This step is optional but does produce better results for the address search in US maps.

### TIGER: Software
* https://github.com/linuxrocks123/tiger_versus_python
* https://wiki.openstreetmap.org/wiki/Osmosis
* https://gitlab.com/osm-c-tools/osmctools

### TIGER: Data
* https://download.geofabrik.de/north-america/us-northeast.poly
* https://www.nominatim.org/data/tiger2019-nominatim-preprocessed.tar.gz

**NOTE** The 2020 TIGER data does not work for building the maps and I haven't had time
to debug the problem.

### TIGER: Procedure
* Untar tiger2019-nominatim-preprocessed.tar.gz:
```
tar zxf tiger2019-nominatim-preprocessed.tar.gz
```

* Convert the SQL to an OSC file:
```
cd tiger
cat *.sql | ../tiger_versus_python/tiger_versus_python.py > usa-tiger-addresses-2019.osc
```

* Merge address OSC file into the OSM data:
```
osmosis --read-xml-change file=<path to>/usa-tiger-addresses-2019.osc --read-pbf-fast \
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
  --output=o5m --mapid=27244000 --wanted-admin-level=8 --status-freq=0 --max-areas=50 \
  united-states-northeast-latest.pbf
```

* mkgmap command to build gmapsupp.img for SD Cards:
```
java -jar <path to>/mkgmap.jar --latin1 --gmapsupp --hide-gmapsupp-on-pc --index \
  --route --series-name="OpenMapChest United States Northeast <YYYY.MM.DD>" \
  --family-name="OpenMapChest United States Northeast" \
  --location-autofill=bounds,is_in,nearest --remove-ovm-work-files \
  --bounds=<path to>/bounds-latest.zip --precomp-sea=<path to>/sea-latest.zip \
  --add-pois-to-areas --process-destination --order-by-decreasing-area \
  --process-exits --pois-to-areas-placement="entrance=main;entrance=yes;building=entrance" \
  --check-roundabout-flares --remove-short-arcs --family-id=<5 digits> \
  --product-id=<1 or 2 digits> --make-opposite-cycleways \
  --road-name-config=<path to>/omc-roadNameConfig.txt --reduce-point-density=4 \
  --reduce-point-density-polygon=8 --merge-lines --polygon-size-limits=24:12,18:10,16:8 \
  --drive-on=right --check-roundabouts --housenumbers \
  --style-file=<path to>/omc-mkgmap-style -c template.args \
  --description="OpenMapChest United States Northeast <YYYY.MM.DD>" \
  <path to>/omc-typ.txt
```

* mkgmap command to build gmap file for BaseCamp:
```
java -jar <path to>/mkgmap.jar --latin1 --gmapi --index \
  --route --series-name="OpenMapChest United States Northeast <YYYY.MM.DD>" \
  --family-name="OpenMapChest United States Northeast" \
  --location-autofill=bounds,is_in,nearest --remove-ovm-work-files \
  --bounds=<path to>/bounds-latest.zip --precomp-sea=<path to>/sea-latest.zip \
  --add-pois-to-areas --process-destination --order-by-decreasing-area \
  --process-exits --pois-to-areas-placement="entrance=main;entrance=yes;building=entrance" \
  --check-roundabout-flares --remove-short-arcs --family-id=<5 digits> \
  --product-id=<1 or 2 digits> --make-opposite-cycleways \
  --road-name-config=<path to>/omc-roadNameConfig.txt --reduce-point-density=4 \
  --reduce-point-density-polygon=8 --merge-lines --polygon-size-limits=24:12,18:10,16:8 \
  --drive-on=right --check-roundabouts --housenumbers \
  --style-file=<path to>/omc-mkgmap-style -c template.args \
  --description="OpenMapChest United States Northeast <YYYY.MM.DD>" \
  <path to>/omc-typ.txt
```

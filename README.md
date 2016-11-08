# Configuring an OpenStreetMap server (Ubuntu 16.04)

The process described here provides a means of creating a standalone tile server that is capable of creating
Raster and Vector tiles from OpenStreetMap data.

It is inspired and based upon the information contained at:

* [Manually building a tile server from switch2osm](https://switch2osm.org/serving-tiles/manually-building-a-tile-server-14-04/)
* [OSM 2 Vector tiles](http://osm2vectortiles.org/)
* [Imposm3 Differ](https://github.com/lyrk/imposm3-differ)

This process utilises [imposm3](https://github.com/omniscale/imposm3) to import data into a postgis database.
The import is significantly faster than using [osm2pgsql](https://github.com/openstreetmap/osm2pgsql).

## Installing prerequisites

```
sudo apt-get install libboost-all-dev subversion git-core tar unzip wget bzip2 build-essential autoconf libtool libxml2-dev libgeos-dev libgeos++-dev 
libpq-dev libbz2-dev libproj-dev munin-node munin libprotobuf-c0-dev protobuf-c-compiler libfreetype6-dev libpng12-dev libtiff5-dev libicu-dev 
libgdal-dev libcairo-dev libcairomm-1.0-dev apache2 apache2-dev libagg-dev liblua5.2-dev ttf-unifont lua5.1 liblua5.1-dev libgeotiff-epsg node-carto sqlite3
gdal-bin osmctools
```

## Install postgresql / postgis

Get the relevant packages from the Ubuntu package manager
```
sudo apt-get install postgresql postgresql-contrib postgis postgresql-9.5-postgis-2.2 pgadmin3
```
Create a database and create a user called osm
```
sudo -u postgres -i
createuser osm
createdb -E UTF8 -O osm osm
exit
```
Create a Ubuntu username called osm and set a password
```
sudo useradd -m osm
sudo passwd osm
```
Setup PostGIS and hstore on the PostgreSQL database and set the osm user password to osm.
```
sudo -u postgres psql
\c osm
CREATE EXTENSION postgis;
CREATE EXTENSION hstore;
ALTER TABLE geometry_columns OWNER TO osm;
ALTER TABLE spatial_ref_sys OWNER TO osm;
ALTER USER osm WITH PASSWORD 'osm';
\q
```

## Install Go and Imposm3
Install Go using
```
sudo apt-get install golang-go
```
Obtain the latest Imposm3 [binary](https://imposm.org/static/rel/).
```
mkdir ~/bin
```
Extract the archive to the `~/bin` and add it to your path.
```
export PATH=$PATH:$HOME/bin
```

## Clone OSMServer repo
```
cd ~/src
git clone https://github.com/daveb1034/OSMServer.git
cd OSMUtils/src
```

## Download OpenStreetMap data

This process has been designed to work on a whole planet load and applies the daily change files to the data on completion.

Download the latest [plant file](http://planet.openstreetmap.org/pbf/planet-latest.osm.pbf) from [planet.openstreetmap.org](http://planet.openstreetmap.org/).

```
cd ~/src/OSMServer/Data/
wget http://planet.openstreetmap.org/pbf/planet-latest.osm.pbf
```

## Import the data into PostgreSQL

The import process is managed using a number of configuration files and a python script.
```
cd ~/src/OSMServer/src/updater
./import_pbf.py -i -C
```
The -i switch calls imposm3 to import the data using the default configuration which will store the cache in ~/src/OSMServer/Data and utilise the mapping file in the smae directory as import_pbf.py.
The -C switch converts the planet-latest.osm.pnf to .o5m for later processing.
## EIA_bulk_download_parse, a work in progress

The U.S. Energy Information Adminstration (EIA) provides bulk data downloads however they are not ready to use. The following Python code downloads and then parses each file into a metadata csv and data.csv which includes only the series_id, YYYYMMDD and value.  

A few of the notable features related to this work:
- The EIA source text files contain many json files.  Each line is a complete fully formed json file.
- The json files contain a 'data' key which is a list of the historical data 
- I chose to create seperate files for the meta and the data thinking that the data would be smaller and more friendly to use in a relational database.  



## EIA download, parse, save to files

```
#EIA download files
import urllib.request
import os
import zipfile

#EIA_manifest = urllib.request.urlopen('http://api.eia.gov/bulk/manifest.txt')
#jmanifest = json.loads(EIA_manifest.readline())
#jmanifest.keys() # each key in 'datasets' is a json object?


#from https://www.eia.gov/opendata/bulkfiles.php
EIA_URL = 'http://api.eia.gov/bulk/'
EIA_archives = [
    #['http://api.eia.gov/bulk/SEDS.zip', 'SEDS.txt']
    ['http://api.eia.gov/bulk/ELEC.zip', 'ELEC.txt']
    , ['http://api.eia.gov/bulk/NG.zip', 'NG.txt']
    , ['http://api.eia.gov/bulk/PET.zip', 'PET.txt'] 
    , ['http://api.eia.gov/bulk/COAL.zip', 'COAL.txt']
    , ['http://api.eia.gov/bulk/STEO.zip', 'STEO.txt'] 
    , ['http://api.eia.gov/bulk/PET_IMPORTS.zip', 'PET_IMPORTS.txt']    
    #['http://api.eia.gov/bulk/INTL.zip', 'INTL.txt']
    #['http://api.eia.gov/bulk/TOTAL.zip', 'TOTAL.txt']
    #['http://api.eia.gov/bulk/EBA.zip', 'EBA.txt']
    #['http://api.eia.gov/bulk/EMISS.zip', 'EMISS.txt']
    #['http://api.eia.gov/bulk/IEO.zip', 'IEO.txt']
    #['http://api.eia.gov/bulk/NUC_STATUS.zip', 'NUC_STATUS.txt']
    ]

for line in range(0, len(EIA_archives)):
    print('URL = ', EIA_archives[line][0])
    print('filename = ', EIA_archives[line][1])
    #URL = EIA_archives[line][0]
    #filename = EIA_archives[line][1]
    #urllib.request.urlretrieve(URL, filename)

#Extract file
import zipfile
zip_ref = zipfile.ZipFile(path_to_zip_file, 'r')
zip_ref.extractall(directory_to_extract_to)
zip_ref.close()

#os.remove('*.zip')
#os.remove('*.txt')

for f in range(13):
    archive = zipfile.ZipFile(EIA_archives[f][0], 'r')
    EIAdata = archive.read(EIA_archives[f][1])
archive = zipfile.ZipFile(urllib.urlopen(EIA_archives[0][0]), 'r')
EIA_data_txt = archive.read(EIA_archives[0][1])

#test
archive = zipfile.ZipFile(urllib.request.urlopen('http://api.eia.gov/bulk/PET.zip'), 'r')
EIA_data_txt = archive.read('PET.txt')



#Parse data into meta and data files

import csv
import json

#destination files for data, suitable for import to relational database
metafile = open("EIA_meta.csv", "a")
datafile = open("EIA_data.csv", "a")

rawfile = open('C:\\Users\\Jeff\\Downloads\\EIA\\NG\\NG.txt')
jfile = json.loads(rawfile.readline())

jfile_dict = jfile.keys()


#loop through all lines of file
with open('C:\\Users\\Jeff\\Downloads\\EIA\\NG\\NG.txt') as lines:
    for line in lines:
        jfile = json.loads(line)

        if 'lat' in jfile_dict:
            _lat = '"' + jfile['lat'] + '", '
        else:
            _lat = '"",'
        
        if 'lon' in jfile_dict:
            _lon = '"' + jfile['lon'] + '", '
        else:
            _lon = '"",'

        if 'geography' in jfile_dict:
            _geography = '"' + jfile['geography'] + '", '
        else:
            _geography = '"",'

        if 'geography2' in jfile_dict:
            _geography2 = '"' + jfile['geography2'] + '", '
        else:
            _geography2 = '"",'

        if 'geoset_id' in jfile_dict:
            _geoset_id = '"' + jfile['geoset_id'] + '", '
        else:
            _geoset_id = '"",'

        #meta file
        #print(jfile['series_id'] + ', ' +
        metafile.write(jfile['series_id'] + ', ' +
            '"' + jfile['name'] + '", ' +
            '"' + jfile['units'] + '", ' +
            '"' + jfile['unitsshort'] + '", ' +
            '"' + jfile['f'] + '", ' +
            '"' + jfile['description'] + '", ' +
            '"' + jfile['copyright'] + '", ' +
            '"' + jfile['source'] + '", ' +
            _lat +
            _lon +
            _geography +
            _geography2 +
            _geoset_id +
            '"' + jfile['start'] + '", ' +
            '"' + jfile['end'] + '", ' +
            '"' + jfile['last_updated'] + '"' +
            "\n"
            )

        #data file
        _data = jfile['data']
        for rec in _data:
            datafile.write('"' + jfile['series_id'] + '", '
            + str(rec[0]) + ", " + str(rec[1]) + "\n")

file_in.close()
metafile.close()
datafile.close()
```

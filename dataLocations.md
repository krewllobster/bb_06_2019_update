# Better Banking Data Process

#### As of January 13th, 2020

## Table of Contents

- [Better Banking Data Process](#better-banking-data-process)
      - [As of January 13th, 2020](#as-of-january-13th-2020)
  - [Table of Contents](#table-of-contents)
  - [Updates](#updates)
  - [Existing Data](#existing-data)
    - [Qualified Census Tract Geographies](#qualified-census-tract-geographies)
  - [Data to Refresh](#data-to-refresh)
    - [FDIC Data](#fdic-data)
      - [FDIC All Reports](#fdic-all-reports)
      - [FDIC Branch Data](#fdic-branch-data)
      - [FDIC Folder Structure](#fdic-folder-structure)
    - [NCUA Data](#ncua-data)
      - [NCUA Call Reports](#ncua-call-reports)
        - [NCUA Branch Data](#ncua-branch-data)
        - [NCUA Institution Data](#ncua-institution-data)
        - [NCUA Report Data](#ncua-report-data)
      - [NCUA Folder Structure](#ncua-folder-structure)
    - [Low Income Credit Unions (LICUs)](#low-income-credit-unions-licus)
  - [Data Processing](#data-processing)
    - [Import FDIC Institution Data](#import-fdic-institution-data)
    - [Import FDIC Branch Data](#import-fdic-branch-data)
    - [Calculate FDIC Branch Statistics](#calculate-fdic-branch-statistics)
      - [Calculate centroid location](#calculate-centroid-location)
      - [Calculate Distance to Centroid](#calculate-distance-to-centroid)
    - [Import NCUA Institution Data](#import-ncua-institution-data)
      - [Field locations](#field-locations)
      - [Import `foicu.txt`](#import-foicutxt)
      - [Import `fs220` and `fs220D`](#import-fs220-and-fs220d)
    - [Import NCUA Branches](#import-ncua-branches)
    - [Calculate NCUA Branch Statistics (NCUA)](#calculate-ncua-branch-statistics-ncua)
      - [Calculate Distance to Centroid (NCUA)](#calculate-distance-to-centroid-ncua)
    - [Matching CDFI Banks & Credit Unions](#matching-cdfi-banks--credit-unions)
      - [Original process in older collection](#original-process-in-older-collection)
      - [As part of a data-update, follow the instructions in the CDFI_data folder](#as-part-of-a-data-update-follow-the-instructions-in-the-cdfidata-folder)

## Updates

| Report Date | Update Started | Update Complete |
| :---------- | :------------: | --------------: |
| 12/31/2018  |  August, 2019  |   January, 2020 |
| 6/30/2019   |  Jan 13, 2020  |             tbd |

## Existing Data

### Qualified Census Tract Geographies

- Contains ~75,000 census tracts stored as geoJSON that includes a "qct" boolean within the properties object
- These are used to determine if branches are located within qcts or not

If this needs to be updated, this is the [direct download link](https://www.ncua.gov/files/publications/resources-expansion/lid-qualifying-geographies-2017.zip) found on [this page](https://www.ncua.gov/support-services/credit-union-resources-expansion/field-membership-expansion/low-income-designation). The file it downloads contains a list of all census tracts (by FIPS) that are considered qualified, **as of 2017**

## Data to Refresh

### FDIC Data

---

#### FDIC All Reports

- [list of reports by date](https://www7.fdic.gov/sdi/download_large_list_outside.asp)
  - [direct download link](https://www7.fdic.gov/sdi/Resource/AllReps/All_Reports_20190630.zip)

#### FDIC Branch Data

- [list of fields & download](https://www7.fdic.gov/sod/dynaDownload.asp?barItem=6)

#### FDIC Folder Structure

```
|-- fdic_data
|   |-- all_reports
|   |   |-- all_reports_raw
|   |   |   +-- [all *.csv from FDIC download]
|   |   +-- All_reports_***_readme.htm
|   |-- branch_data
|       |-- branch_data_definitions
|       |   +-- "Definitions-Table 1.csv"
|       |   +-- "Deleted Variables-Table 1.csv"
|       |   +-- "Read_me-Table 1.csv"
|       |   +-- "Sheet1-Table 1.csv"
|       |-- branch_data_raw
|           +-- ALL_2019.csv
```

### NCUA Data

---

#### NCUA Call Reports

- [list of dates & download](https://www.ncua.gov/analysis/credit-union-corporate-call-report-data/quarterly-data)

After download, separate out the files into the following folders/sections

##### NCUA Branch Data

- `ATM.txt`
- `Credit Union Branch Information.txt`

##### NCUA Institution Data

- `Acct-DescTradeNames.txt`
- `TradeNames.txt` (This is where you get CU names from rather than the search and replace crap)
- `foicu.txt` (data for institutions)
- `FOICUDES.txt` (data key for institutions)

##### NCUA Report Data

- `fs220.txt`,
- `fs220A.txt`,
- `fs220B.txt`
- `fs220C.txt`
- `fs220D.txt`
- `fs220G.txt`
- `fs220H.txt`
- `fs220I.txt`,
- `fs220J.txt`
- `fs220K.txt`
- `fs220L.txt`
- `fs220M.txt`
- `fs220N.txt`

#### NCUA Folder Structure

```
|-- ncua_data
    |-- call_reports_raw
        |-- branch_data
            +-- ATM.txt
            +-- "Credit Union Branch Information.txt"
        |-- institution_data
            +-- Acct-DescTradeNames.txt
            +-- foicu.txt
            +-- FOICUDES.txt
            +-- TradeNames.txt
        |-- report_data
            +-- fs220.txt
            +-- fs220A.txt
            +-- fs220B.txt
            ...
            +-- fs220N.txt
        +-- Acct-DescGrants.txt
        +-- AcctDesc.txt
        +-- Grants.txt
        +-- Readme.txt
        +-- Report1.txt
```

### Low Income Credit Unions (LICUs)

// TODO

## Data Processing

All data will be going to a local mongoDb database.

A basic command in terminal to load a csv into mongo is the following:

```
mongoimport --db [databaseName] -c [collection] --type=csv --headerline
```

### Import FDIC Institution Data

This data comes from fdic_all_reports. We need the following fields:

- bkclass - common
- fed_rssd - common
- cert - common
- specgrp - common
- name - common
- address - common
- city - common
- state - common
- zip - common
- webaddr - common
- repdte - common
- asset - Assets and Liabilities
- lnlsnet - Net Loans and Leases
- dep - Assets and Liabilities
- depdom - Assets and Liabilities
- lnreres - Net Loans and Leases
- lnag - Net Loans and Leases
- lnreag - Net Loans and Leases
- lnci - Net Loans and Leases
- lncon - Net Loans and Leases
- lncrcd - Net Loans and Leases
- Lnrenr4 - Small Business Loans
- Lnci4 - Small Business Loans
- Lnreag4 - Small Business Loans
- Lnag4 - Small Business Loans
- mutual - common
- cb - common

We only need to import:
`All_Reports_20190630_Assets and Liabilities.csv`
`All_Reports_20190630_Net Loans and Leases.csv`
`All_Reports_20190630_Small Business Loans.csv`

However, after importing the first file, we will be merging using the --upsertFields command

```javascript
// first import command
mongoimport -d bb2020 -c fdic_all_reports --headerline --type=csv --file="All_Reports_20190630_Assets and Liabilities.csv"
```

The second and third import commands:

```javascript
// second import command. Note --mode and --upsertFields
mongoimport -d bb2020 -c fdic_all_reports --headerline --type=csv --file="All_Reports_20190630_Net Loans and Leases.csv" --mode merge --upsertFields=fed_rssd

// third import command. Note --mode and --upsertFields
mongoimport -d bb2020 -c fdic_all_reports --headerline --type=csv --file="All_Reports_20190630_Small Business Loans.csv" --mode merge --upsertFields=fed_rssd
```

Create an rssd field on each document:

```javascript
db.fdic_all_reports.aggregate([
  {
    $addFields: {
      rssd: '$fed_rssd'
    }
  },
  {
    $out: 'fdic_all_reports'
  }
]);
```

Now we index the `fed_rssd` field in the `fdic_all_reports` collection:

```javascript
db.fdic_all_reports.createIndex({ fed_rssd: 1 });
db.fdic_all_reports.createIndex({ rssd: 1 });
```

### Import FDIC Branch Data

While in the folder: ~/fdic_data/branch_data/branch_data_raw

```
mongoimport -d bb2020 -c fdic_branches --headerline --type=csv --file=ALL_2019.csv
```

First aggregation will filter out bank branches that either have no geolocation, or have a latitude/longitude of 0. It will create a location field and a 'rssd' field. The location and rssd will be indexed to facilitate future aggregations.

```
// First fdic_branches aggregation
db.fdic_branches_raw.aggregate([
  {
    $match: {
      $expr: {
        $and: [
          {
            $ne: [
              {
                $type: '$SIMS_LONGITUDE'
              },
              'string'
            ]
          },
          {
            $ne: [
              {
                $type: '$SIMS_LATITUDE'
              },
              'string'
            ]
          }
        ]
      }
    }
  },
  {
    $match: {
      $expr: {
        $and: [
          {
            $ne: [
              '$SIMS_LONGITUDE',
              0
            ]
          },
          {
            $ne: [
              '$SIMS_LATITUDE',
              0
            ]
          }
        ]
      }
    }
  },
  {
    $addFields: {
      location: {
        type: "Point",
        coordinates: [
          '$SIMS_LONGITUDE',
          '$SIMS_LATITUDE'
        ]
      },
      rssd: '$RSSDID',
      regulator: 'fdic'
    }
  },
  {
    $out: 'fdic_branches'
  }
])
```

Two index commands in the mongo client terminal:

```javascript
// create 2dsphere index on location field
db.fdic_branches.createIndex({ location: '2dsphere' });
db.fdic_branches.createIndex({ rssd: 1 });
```

### Calculate FDIC Branch Statistics

First we need to normalize the numbers on each fdic_branch document: (enter in mongo client)

```javascript
db.fdic_branches.find().forEach((x, i) => {
  const domesticDeposits = parseInt(x.DEPDOM.toString().replace(/,/g, ''));
  const branchDeposits = parseInt(x.DEPSUMBR.toString().replace(/,/g, ''));
  x.domesticDeposits = domesticDeposits;
  x.branchDeposits = branchDeposits;
  db.fdic_branches.save(x);
});
```

#### Calculate centroid location

We'll use the [average geolocation helper function](./helper_functions/averageGeolocation.js) to calculate the centroid of all branches:

```javascript
// define this averageGeolocation function in the mongo termiinal
const averageGeolocation = require('./helper_functions/averageGeolocation.js');

//group all branch locations by rssd and calculate the centroid
db.fdic_branches
  .aggregate([
    {
      $group: {
        _id: {
          rssd: '$rssd'
        },
        branchIds: {
          $push: '$_id'
        },
        coords: {
          $push: {
            latitude: '$SIMS_LATITUDE',
            longitude: '$SIMS_LONGITUDE'
          }
        }
      }
    }
  ])
  .forEach(({ _id: rssd, branchIds, coords }) => {
    // for each set of branch ids, find the avg. location and write it back to all
    // of the branches as a new location
    const { latitude, longitude } = averageGeolocation(coords);
    branchIds.forEach(id => {
      db.fdic_branches.findOneAndUpdate(
        { _id: id },
        {
          $set: {
            centroidLocation: {
              type: 'Point',
              coordinates: [longitude, latitude]
            }
          }
        }
      );
    });
  });
```

#### Calculate Distance to Centroid

The following is the haversine formula to find the distance between two points. We use this on each `fdic_branch` document to calculate the distance in meters between the branch location and the centroid location. This will eventually be used to calculate the Branch Density score.

```javascript
// define haversine formula in mongo terminal
const haversineDistance = require('./helper_functions/haversineDistance.js');
```

Now we iterate through all the fdic_branches, calculate the distance to the centroid and write it back to the branch as 'distanceToCentroid'.

```javascript
db.fdic_branches.find().forEach((x, i) => {
  const branchCoords = {
    latitude: x.location.coordinates[1],
    longitude: x.location.coordinates[0]
  };
  const centroidCoords = {
    latitude: x.centroidLocation.coordinates[1],
    longitude: x.centroidLocation.coordinates[0]
  };

  const distToCentroid = haversineDistance(branchCoords, centroidCoords);
  x.distanceToCentroid = distToCentroid;
  db.fdic_branches.save(x);
});
```

### Import NCUA Institution Data

This data comes from `ncua_data/call_reports_raw/report_data`.

#### Field locations

We need the following fields:

- RSSD
  - Rssd for id
  - `foicu.txt`
- CU_NUMBER
  - Credit Union charter number
  - `foicu.txt`
- CU_NAME
  - Credit Union Name
  - `foicu.txt`
- STREET
  - Street Addres
  - `foicu.txt`
- CITY
  - City
  - `foicu.txt`
- STATE
  - State
  - `foicu.txt`
- ZIP_CODE
  - Zip
  - `foicu.txt`
- Acct_891
  - website
  - `fs220D.txt`
- CYCLE_DATE
  - report date
  - `foicu.txt`
- CU_TYPE
  - credit union type
    1. Federal Credit Union
    2. Federally Insured State-Chartered Credit Union
    3. State Credit Union
  - `foicu.txt`
- ACCT_010
  - total assets
  - fs220.txt
- ACCT_025B
  - total loans
  - fs220.txt
- ACCT_018
  - total deposits
  - fs220.txt
- ACCT_703
  - housing lending
  - fs220.txt
- ACCT_387
  - small business lending (non agricultural business loans)
  - fs220.txt
- ACCT_042
  - small agricultural lending
  - fs220.txt
- MemberMinorityStatus
  - member minority status
  - fs220D.txt
- Acct_886H
  - supports mobile banking
  - fs220D.txt
- Acct_887J
  - supports bill pay
  - fs220D.txt

This leaves us importing only three documents for institutions:

1. `foicu.txt`
2. `fs220.txt`
3. `fs220D.txt`

#### Import `foicu.txt`

Inside the `ncua_data/call_reports_raw/institution_data` folder in terminal use:

```bash
mongoimport -d bb2020 -c ncua_all_reports --headerline --type=csv --file=foicu.txt
```

#### Import `fs220` and `fs220D`

We will merge in these second two files using CU_NUMBER as the merge key. Once in the report_data folder use:

```bash
mongoimport -d bb2020 -c ncua_all_reports --headerline --type=csv --file="fs220.txt" --mode merge --upsertFields=CU_NUMBER

mongoimport -d bb2020 -c ncua_all_reports --headerline --type=csv --file="fs220D.txt" --mode merge --upsertFields=CU_NUMBER
```

We'll do one aggregation here to fill out the credit union type and rssd. This will help match credit unions with CDFI data:

```javascript
db.ncua_all_reports.aggregate([
  {
    $addFields: {
      rssd: '$RSSD',
      creditUnionType: {
        $switch: {
          branches: [
            {
              case: {
                $eq: ['$CU_TYPE', 1]
              },
              then: 'Federal Credit Union'
            },
            {
              case: {
                $eq: ['$CU_TYPE', 2]
              },
              then: 'Federally Insured State-Chartered Credit Union'
            },
            {
              case: {
                $eq: ['$CU_TYPE', 3]
              },
              then: 'State Credit Union'
            }
          ]
        }
      }
    }
  },
  {
    $out: 'ncua_all_reports'
  }
]);
```

Create index on rssd and CU_NUMBER field:

```javascript
db.ncua_all_reports.createIndex({ rssd: 1 });
db.ncua_all_reports.createIndex({ CU_NUMBER: 1 });
```

### Import NCUA Branches

The file `ncua_data/call_reports_raw/branch_data/Credit Union Branch Information.txt` contains address information about each credit union branch. We need to convert this .txt file to a .csv file and then upload it to geocod.io for processing. We use the physical address for geocoding.

First import to mongo:

```bash
mongoimport -d bb2020 -c ncua_branches --headerline --type=csv --file="Credit Union Branch Information.txt"
```

Then bring rssd down from ncua_all_reports:

```javascript
db.ncua_branches.aggregate([
  {
    $lookup: {
      from: 'ncua_all_reports',
      localField: 'CU_NUMBER',
      foreignField: 'CU_NUMBER',
      as: 'hq'
    }
  },
  {
    $unwind: '$hq'
  },
  {
    $addFields: {
      rssd: '$hq.RSSD'
    }
  },
  {
    $project: {
      hq: 0
    }
  },
  {
    $out: 'ncua_branches'
  }
]);
```

Confirm that all the branches received an rssd (i.e. have a parent institution)

```javascript
db.ncua_branches.find({ rssd: { $exists: true } }).count();
// should equal
db.ncua_branches.count();
```

Then create an index on rssd:

```javascript
db.ncua_branches.createIndex({ rssd: 1 });
```

Now we can export this collection as a csv. In the `ncua_data/geocoding` folder run:

```bash
mongoexport -d bb2020 -c ncua_branches --fields rssd,CU_NUMBER,SiteId,CU_NAME,SiteName,SiteTypeName,MainOffice,PhysicalAddressLine1,PhysicalAddressLine2,PhysicalAddressCity,PhysicalAddressStateCode,PhysicalAddressPostalCode,PhysicalAddressCountyName,PhysicalAddressCountry --type=csv --out ncua_branches_for_geocoding.csv
```

Then submit to geocod.io for geocoding of addresses into latitude longitude. Then we'll essentially repeat the above steps.

Once you receive the file back, merge back into `ncua_branches` collection in mongo using SiteId as key:

```bash
mongoimport -d bb2020 -c ncua_branches --fieldFile="fieldfile.txt"  --columnsHaveTypes --type=csv  --file="geoCode_response.csv" --mode merge --upsertFields=SiteId
```

Now add the location field:

```javascript
db.ncua_branches.aggregate([
  {
    $addFields: {
      location: {
        type: 'Point',
        coordinates: ['$Longitude', '$Latitude']
      }
    }
  },
  {
    $out: 'ncua_branches'
  }
]);
```

### Calculate NCUA Branch Statistics (NCUA)

We'll use the [average geolocation helper function](./helper_functions/averageGeolocation.js) to calculate the centroid of all branches:

```javascript
// define this averageGeolocation function in the mongo termiinal
const averageGeolocation = require('./helper_functions/averageGeolocation.js');

//group all branch locations by rssd and calculate the centroid
db.ncua_branches
  .aggregate([
    {
      $group: {
        _id: {
          rssd: '$rssd'
        },
        branchIds: {
          $push: '$_id'
        },
        coords: {
          $push: {
            latitude: '$SIMS_LATITUDE',
            longitude: '$SIMS_LONGITUDE'
          }
        }
      }
    }
  ])
  .forEach(({ _id: rssd, branchIds, coords }) => {
    // for each set of branch ids, find the avg. location and write it back to all
    // of the branches as a new location
    const { latitude, longitude } = averageGeolocation(coords);
    branchIds.forEach(id => {
      db.fdic_branches.findOneAndUpdate(
        { _id: id },
        {
          $set: {
            centroidLocation: {
              type: 'Point',
              coordinates: [longitude, latitude]
            }
          }
        }
      );
    });
  });
```

#### Calculate Distance to Centroid (NCUA)

The following is the haversine formula to find the distance between two points. We use this on each `fdic_branch` document to calculate the distance in meters between the branch location and the centroid location. This will eventually be used to calculate the Branch Density score.

```javascript
// define haversine formula in mongo terminal
const haversineDistance = require('./helper_functions/haversineDistance.js');
```

Now we iterate through all the fdic_branches, calculate the distance to the centroid and write it back to the branch as 'distanceToCentroid'.

```javascript
db.fdic_branches.find().forEach((x, i) => {
  const branchCoords = {
    latitude: x.location.coordinates[1],
    longitude: x.location.coordinates[0]
  };
  const centroidCoords = {
    latitude: x.centroidLocation.coordinates[1],
    longitude: x.centroidLocation.coordinates[0]
  };

  const distToCentroid = haversineDistance(branchCoords, centroidCoords);
  x.distanceToCentroid = distToCentroid;
  db.fdic_branches.save(x);
});
```

### Matching CDFI Banks & Credit Unions

#### Original process in older collection

Create institution collection

```javascript
db.fdic_all_reports.find().forEach(x => {
  const newDoc = {
    name: x.name,
    city: x.city,
    state: x.stalp,
    zip: x.zip,
    address: x.address,
    rssd: x.fed_rssd,
    regulator: 'fdic',
    type: 'bank',
    website: x.webaddr
  };
  db.all_institutions.save(newDoc);
});

db.ncua_all_reports.find().forEach(x => {
  const newDoc = {
    name: x.CU_NAME,
    city: x.CITY,
    state: x.STATE,
    zip: x.ZIP_CODE,
    address: x.STREET,
    rssd: x.RSSD,
    regulator: 'ncua',
    type: x.creditUnionType,
    website: x.Acct_891
  };
  db.all_institutions.save(newDoc);
});
```

Next, create an exports folder under your root directory and navigate there in terminal and export the collection we just created:

```bash
mongoexport --db=bb2020 --collection=all_institutions --type=csv --fields=name,city,state,zip,address,regulator,type,website,rssd  --out=all_institutions_for_cdfi.csv
```

Matching CDFI fund data gives us the files in the CDFI_data folder.

#### As part of a data-update, follow the instructions in the CDFI_data folder

[Instructions here](CDFI_data/mergeCommands.md)

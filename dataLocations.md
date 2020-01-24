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
      - [Create 2d sphere index on NCUA branches](#create-2d-sphere-index-on-ncua-branches)
    - [Seting QCT Status for branches](#seting-qct-status-for-branches)
    - [SPM Query](#spm-query)
      - [NCUA SPM Base](#ncua-spm-base)
      - [FDIC SPM Base](#fdic-spm-base)
      - [Combine FDIC & NCUA SPM_BASE INTO ALL INSTITUTIONS](#combine-fdic--ncua-spmbase-into-all-institutions)
      - [Add score data](#add-score-data)
      - [Matching CDFI Banks & Credit Unions](#matching-cdfi-banks--credit-unions)
      - [Adding FDIC Minority Depository Institution Flag](#adding-fdic-minority-depository-institution-flag)
      - [STATUS](#status)
    - [Combining Branches into All_Branches Collection](#combining-branches-into-allbranches-collection)
      - [Convert FDIC & NCUA Branches into identical structure](#convert-fdic--ncua-branches-into-identical-structure)
    - [Add Zip Code Data to New DB](#add-zip-code-data-to-new-db)

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

Then create an index on rssd and a joint unique index on rssd & SiteId:

```javascript
db.ncua_branches.createIndex({ rssd: 1 });
db.ncua_branches.createIndex({ rssd: 1, SiteId: 1 }, { unique: true });
```

Now we can export this collection as a csv. In the `ncua_data/geocoding` folder run:

```bash
mongoexport -d bb2020 -c ncua_branches --fields rssd,CU_NUMBER,SiteId,CU_NAME,SiteName,SiteTypeName,MainOffice,PhysicalAddressLine1,PhysicalAddressLine2,PhysicalAddressCity,PhysicalAddressStateCode,PhysicalAddressPostalCode,PhysicalAddressCountyName,PhysicalAddressCountry --type=csv --out ncua_branches_for_geocoding.csv
```

Then submit to geocod.io for geocoding of addresses into latitude longitude. Then we'll essentially repeat the above steps.

Once you receive the file back, merge back into `ncua_branches` collection in mongo using SiteId as key:

```bash
mongoimport -d bb2020 -c ncua_branches --fieldFile="fieldfile.txt"  --columnsHaveTypes --type=csv  --file="geoCode_response.csv" --mode merge --upsertFields=rssd,SiteId
```

**hint**: If something is being weird, you may have a document in ncua_branches that is the headerline as the above command uses a field file. Just find and delete that one.

Then delete any that were not geolocated (4 in 6/30 update for example):

```javascript
db.ncua_branches.deleteMany({ Longitude: { $eq: 0 } });
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
            latitude: '$Latitude',
            longitude: '$Longitude'
          }
        }
      }
    }
  ])
  .forEach(({ _id: rssd, branchIds, coords }) => {
    const { latitude, longitude } = averageGeolocation(coords);
    branchIds.forEach(id => {
      db.ncua_branches.findOneAndUpdate(
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
db.ncua_branches.find().forEach((x, i) => {
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
  db.ncua_branches.save(x);
});
```

#### Create 2d sphere index on NCUA branches

```javascript
db.ncua_branches.createIndex({ location: '2dsphere' });
```

### Seting QCT Status for branches

We are using the `qct_geo` collection found in our older database `bb`

```javascript
const otherDb = db.getSiblingDB('bb');
db.ncua_branches.find().forEach((x, i) => {
  const { location } = x;
  const tract = otherDb.qct_geo.findOne(
    {
      geometry: {
        $geoIntersects: {
          $geometry: location
        }
      }
    },
    { geometry: 0 }
  );
  const isQct = tract ? tract.properties.qct : null;
  x.qct = isQct;
  db.ncua_branches.save(x);
  if (i % 100 === 0) {
    print(i, ' complete');
  }
});

db.fdic_branches.find().forEach((x, i) => {
  const { location } = x;
  const tract = otherDb.qct_geo.findOne(
    {
      geometry: {
        $geoIntersects: {
          $geometry: location
        }
      }
    },
    { geometry: 0 }
  );
  const isQct = tract ? tract.properties.qct : null;
  x.qct = isQct;
  db.fdic_branches.save(x);
});
```

### SPM Query

#### NCUA SPM Base

Create NCUA institutions collection

```javascript
db.ncua_all_reports.aggregate([
  {
    $project: {
      rssd: 1,
      cu_number: '$CU_NUMBER',
      regulator: 'ncua',
      bankInformation: {
        city: '$CITY',
        state: '$STATE',
        zip: '$ZIP_CODE',
        name: '$CU_NAME',
        streetAddress: '$STREET',
        creditUnionType: '$creditUnionType',
        year_opened: '$ISSUE_DATE',
        website: '$Acct_891'
      },
      meta: {
        reportDate: '$CYCLE_DATE',
        collection: 'ncua_all_reports',
        originId: '$_id'
      },
      coreStatistics: {
        totalAssets: {
          $divide: [
            '$ACCT_010',
            1000
          ]
        },
        totalAssetsThousands: {
          $divide: ['$ACCT_010', 1000]
        },
        totalAssetsDollars: '$ACCT_010',
        totalAssetsBillions: {
          $divide: ['$ACCT_010', 1000000000]
        },
        totalLoans: {
          $divide: [
            '$ACCT_025B',
            1000
          ]
        },
        totalDeposits: {
          $divide: [
            '$ACCT_018',
            1000
          ]
        },
        housingLending: {
          $divide: [
            '$ACCT_703',
            1000
          ]
        },
        smallBizLending: {
          $divide: [
            '$ACCT_387',
            1000
          ]
        },
        farmLending: {
          $divide: [
            '$ACCT_042',
            1000
          ]
        }
      },
      flags: {
        mcu: {
          $cond: {
            if: {
              $eq: ['$MemberMinorityStatus', 1]
            },
            then: true,
            else: false
          }
        },
        licu: {
          $cond: {
            if: {
              $eq: ['$LIMITED_INC', 1]
            },
            then: true,
            else: false
          }
        }
      }
    }
  },
  {
    $lookup: {
      from: 'ncua_branches',
      let: {
        inst_rssd: '$rssd'
      },
      pipeline: [
        {
          $match: {
            $expr: {
              $eq: ['$rssd', '$$inst_rssd']
            }
          }
        },
        {
          $group: {
            _id: null,
            countQct: {
              $sum: {
                $cond: {
                  if: {
                    $eq: ['$qct', true]
                  },
                  then: 1,
                  else: 0
                }
              }
            },
            countTotal: {
              $sum: 1
            },
            avgDistToCenter: {
              $avg: '$distanceToCentroid'
            }
          }
        },
        {
          $project: {
            _id: 0
          }
        }
      ],
      as: 'branchStatistics'
    }
  },
  {
    $unwind: '$branchStatistics'
  },
  {
    $out: 'ncua_institutions_06_2019'
  }
]);
```

#### FDIC SPM Base

Create FDIC institutions collection.

__NOTE!!!__ NCUA publishes actual dollar values for fields like total assets, total loans, etc. FDIC publishes those same values in thousands of dollars. In this query, we are converting all values that NCUA has as actual into thousands.

```javascript
db.fdic_all_reports.aggregate([
  {
    $project: {
      rssd: '$fed_rssd',
      cert: '$cert',
      regulator: 'fdic',
      bankInformation: {
        city: '$city',
        state: '$state',
        zip: '$zip',
        name: '$name',
        streetAddress: '$address',
        year_opened: '$estymd',
        website: '$webaddr'
      },
      meta: {
        reportDate: '$repdte',
        collection: 'fdic_all_reports',
        originId: '$_id'
      },
      coreStatistics: {
        totalAssets: '$asset',
        totalAssetsThousands: '$asset',
        totalAssetsDollars: {
          $multiply: ['$asset', 1000]
        },
        totalAssetsBillions: {
          $divide: ['$asset', 1000000]
        },
        totalLoans: '$lnlsnet',
        totalDeposits: '$depdom',
        housingLending: '$lnreres',
        smallBizLending: {
          $sum: ['$Lnrenr4', '$Lnci4']
        },
        farmLending: {
          $sum: ['Lnreag4', 'Lnag4']
        }
      },
      flags: {
        mutual: {
          $cond: {
            if: {
              $eq: ['$mutual', 1]
            },
            then: true,
            else: false
          }
        },
        communityBank: {
          $cond: {
            if: {
              $eq: ['$cb', 1]
            },
            then: true,
            else: false
          }
        }
      }
    }
  },
  {
    $lookup: {
      from: 'fdic_branches',
      let: {
        inst_rssd: '$rssd'
      },
      pipeline: [
        {
          $match: {
            $expr: {
              $eq: ['$rssd', '$$inst_rssd']
            }
          }
        },
        {
          $group: {
            _id: null,
            countQct: {
              $sum: {
                $cond: {
                  if: {
                    $eq: ['$qct', true]
                  },
                  then: 1,
                  else: 0
                }
              }
            },
            countTotal: {
              $sum: 1
            },
            avgDistToCenter: {
              $avg: '$distanceToCentroid'
            }
          }
        },
        {
          $project: {
            _id: 0
          }
        }
      ],
      as: 'branchStatistics'
    }
  },
  {
    $unwind: '$branchStatistics'
  },
  {
    $out: 'fdic_institutions_06_2019'
  }
]);
```

#### Adding FDIC Minority Depository Institution Flag

List of MDIs is found [on the FDIC website](https://www.fdic.gov/regulations/resources/minority/mdi.html)

June 2019 data found in [MDI_data](./MDI_data)

Load in the cert numbers as FDIC_MDI_06_2019 collection:

```bash
mongoimport -d bb2020 -c fdic_mdi_06_2019 --file=fdic_mdi_certs.csv --headerline --type=csv
```

Now run the following aggregation to look for FDIC institutions, then lookup if the cert matches an MDI. If so, then it is an MDI, otherwise not.

```javascript
db.fdic_institutions_06_2019.aggregate([
  {
    $lookup: {
      from: 'fdic_mdi_06_2019',
      localField: 'cert',
      foreignField: 'cert',
      as: 'mdi_list'
    }
  },
  {
    $addFields: {
      'flags.mdi': {
        $cond: {
          if: {
            $gt: [{ $size: '$mdi_list' }, 0]
          },
          then: true,
          else: false
        }
      }
    }
  },
  {
    $project: {
      mdi_list: 0
    }
  },
  {
    $out: 'fdic_institutions_06_2019'
  }
]);
```

#### Matching CDFI Banks & Credit Unions

Create a collection using [all_cdfis.csv](./CDFI_data/all_cdfis.csv) from the CDFI_data folder

```bash
mongoimport -d bb2020 -c all_cdfis --file=all_cdfis.csv --type=csv --headerline
```

Ensure rssd is indexed in both collections:

```javascript
db.all_institutions_spm_06_2019.ensureIndex({ rssd: 1 });
db.all_cdfis.ensureIndex({ rssd: 1 });
```

Then add this as a flag to `fdic_institutions_06_2019` and `ncua_institutions_06_2019` with the following aggregations:

```javascript
db.fdic_institutions_06_2019.aggregate([
  {
    $lookup: {
      from: 'all_cdfis',
      localField: 'rssd',
      foreignField: 'rssd',
      as: 'cdfi_record'
    }
  },
  {
    $unwind: '$cdfi_record'
  },
  {
    $addFields: {
      'flags.cdfi': {
        $cond: {
          if: {
            $eq: ['$cdfi_record.CDFI', 'TRUE']
          },
          then: true,
          else: false
        }
      }
    }
  },
  {
    $project: {
      cdfi_record: 0
    }
  },
  {
    $out: 'fdic_institutions_06_2019'
  }
]);

db.ncua_institutions_06_2019.aggregate([
  {
    $lookup: {
      from: 'all_cdfis',
      localField: 'rssd',
      foreignField: 'rssd',
      as: 'cdfi_record'
    }
  },
  {
    $unwind: '$cdfi_record'
  },
  {
    $addFields: {
      'flags.cdfi': {
        $cond: {
          if: {
            $eq: ['$cdfi_record.CDFI', 'TRUE']
          },
          then: true,
          else: false
        }
      }
    }
  },
  {
    $project: {
      cdfi_record: 0
    }
  },
  {
    $out: 'ncua_institutions_06_2019'
  }
]);
```

#### Combine FDIC & NCUA SPM_BASE INTO ALL INSTITUTIONS

Run the following commands in the mongo terminal:

```javascript
// save each fdic record into all_institutions
db.fdic_institutions_06_2019.find().forEach(x => {
  db.all_institutions_06_2019.save(x);
});

// save each ncua record into all_institutions
db.ncua_institutions_06_2019.find().forEach(x => {
  db.all_institutions_06_2019.save(x);
});
```

#### Add score data

Run this aggregation on all institutions to add social performance metrics and scores:

```javascript
db.all_institutions_06_2019.aggregate([
  {
    $addFields: {
      'branchStatistics.pctQctBranches': {
        $divide: ['$branchStatistics.countQct', '$branchStatistics.countTotal']
      },
      'branchStatistics.avgDistToCenterMiles': {
        $multiply: ['$branchStatistics.avgDistToCenter', 0.000621371]
      },
      weights: {
        qualityLending: 0.6,
        qctPresence: 0.2,
        branchDensity: 0.1,
        totalAssets: 0.1
      }
    }
  },
  {
    $addFields: {
      scores: {
        qualityLending: {
          $divide: [
            {
              $sum: [
                '$coreStatistics.housingLending',
                '$coreStatistics.smallBizLending',
                '$coreStatistics.farmLending'
              ]
            },
            '$coreStatistics.totalAssetsThousands'
          ]
        },
        qctPresence: '$branchStatistics.pctQctBranches',
        branchDensity: {
          $max: [
            {
              $divide: [
                {
                  $subtract: [250, '$branchStatistics.avgDistToCenterMiles']
                },
                250
              ]
            },
            0
          ]
        },
        totalAssets: {
          $divide: [
            1,
            {
              $add: [
                1,
                {
                  $divide: [
                    {
                      $pow: ['$coreStatistics.totalAssetsBillions', 2]
                    },
                    25
                  ]
                }
              ]
            }
          ]
        }
      }
    }
  },
  {
    $addFields: {
      weightedScores: {
        qualityLending: {
          $multiply: ['$scores.qualityLending', '$weights.qualityLending']
        },
        qctPresence: {
          $multiply: ['$scores.qctPresence', '$weights.qctPresence']
        },
        branchDensity: {
          $multiply: ['$scores.branchDensity', '$weights.branchDensity']
        },
        totalAssets: {
          $multiply: ['$scores.totalAssets', '$weights.totalAssets']
        }
      }
    }
  },
  {
    $addFields: {
      totalScores: {
        raw: {
          $sum: [
            '$scores.qualityLending',
            '$scores.qctPresence',
            '$scores.branchDensity',
            '$scores.totalAssets'
          ]
        },
        weighted: {
          $sum: [
            '$weightedScores.qualityLending',
            '$weightedScores.qctPresence',
            '$weightedScores.branchDensity',
            '$weightedScores.totalAssets'
          ]
        }
      }
    }
  },
  {
    $out: 'all_institutions_spm_06_2019'
  }
]);
```

And create an index on rssd:

```javascript
db.all_institutions_spm_06_2019.createIndex({rssd: 1})
```

#### STATUS

There should now be a main institution collection called `all_institutions_spm_06_2019`. For FDIC institutions, flags include MDI, Community Bank, and CDFI. For NCUA institutions, flags include MemberMinorityStatus, LowIncome, and CDFI.

### Combining Branches into All_Branches Collection

#### Convert FDIC & NCUA Branches into identical structure

We will be creating new collections ncua & fdic \_spm_base_06_2019 and mapping data to the same fields and turning them into geoJson objects:

```javascript
db.fdic_branches.aggregate([
  {
    $project: {
      rssd: 1,
      meta: {
        uid: {
          UNINUMBR: '$UNINUMBR'
        },
        collection: 'fdic_branches',
        originId: '$_id',
        reportDate: '$YEAR'
      },
      mainOffice: {
        $cond: {
          if: {
            $eq: ['$BKMO', 1]
          },
          then: true,
          else: false
        }
      },
      bankName: '$NAMEFULL',
      branchName: '$NAMEBR',
      address1: '$ADDRESBR',
      city: '$CITYBR',
      state: '$STALPBR',
      stateName: '$STNAMEBR',
      zip: '$ZIPBR',
      location: 1,
      regulator: 1,
      centroidLocation: 1,
      distanceToCentroid: 1,
      distanceToCentroidMiles: {
        $multiply: ['$distanceToCentroid', 0.000621371]
      },
      qct: 1
    }
  },
  {
    $out: 'fdic_branches_base_06_2019'
  }
]);

db.ncua_branches.aggregate([
  {
    $project: {
      rssd: 1,
      meta: {
        uid: {
          rssd: '$rssd',
          SiteId: '$SiteId'
        },
        collection: 'ncua_branches',
        originId: '$_id',
        reportDate: '$CYCLE_DATE'
      },
      mainOffice: {
        $cond: {
          if: {
            $eq: ['$MainOffice', 'Yes']
          },
          then: true,
          else: false
        }
      },
      bankName: '$CU_NAME',
      branchName: '$SiteName',
      address1: '$PhysicalAddressLine1',
      address2: '$PhysicalAddressLine2',
      city: '$PhysicalAddressCity',
      state: '$PhysicalAddressStateCode',
      zip: '$PhysicalAddressPostalCode',
      location: 1,
      regulator: 'ncua',
      centroidLocation: 1,
      distanceToCentroid: 1,
      distanceToCentroidMiles: {
        $multiply: ['$distanceToCentroid', 0.000621371]
      },
      qct: 1
    }
  },
  {
    $out: 'ncua_branches_base_06_2019'
  }
]);
```

Then combine them:

```javascript
db.fdic_branches_base_06_2019.find().forEach(x => {
  db.all_branches_06_2019.save(x);
});

db.ncua_branches_base_06_2019.find().forEach(x => {
  db.all_branches_06_2019.save(x);
});
```

Add indexes on rssd and location.

```javascript
db.all_branches_06_2019.createIndex({ rssd: 1 });
db.all_branches_06_2019.createIndex({ location: '2dsphere' });
```

### Add Zip Code Data to New DB

```javascript
const otherDb = db.getSiblingDB('bb');
otherDb.zips.find().forEach(x => {
  db.zips.save(x);
});
```

Create index on 'postalCode' and 'location' (2dsphere for location)

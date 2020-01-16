# Add CDFI Flags

## NCUA import & aggregation

### Import cdfi values

```bash
mongoimport -d bb2020 -c ncua_all_reports --fields=rssd,cdfi --type=csv --file="ncua_cdfis.csv" --mode merge --upsertFields=rssd
```

### Aggregate to convert to Boolean

```javascript
db.ncua_all_reports.aggregate([
  {
    $addFields: {
      cdfi: {
        $cond: {
          if: {
            $eq: ['$cdfi', 'TRUE']
          },
          then: true,
          else: false
        }
      }
    }
  },
  {
    $out: 'ncua_all_reports'
  }
]);
```

## FDIC import & aggregation

### Import CDFI Values

```bash
mongoimport -d bb2020 -c fdic_all_reports --headerline --type=csv --file="fdic_cdfis.csv" --mode merge --upsertFields=rssd
```

### Convert cdfi to boolean

```javascript
db.fdic_all_reports.aggregate([
  {
    $addFields: {
      cdfi: {
        $cond: {
          if: {
            $eq: ['$CDFI', 'TRUE']
          },
          then: true,
          else: false
        }
      }
    }
  },
  {
    $project: {
      CDFI: 0
    }
  },
  {
    $out: 'fdic_all_reports'
  }
]);
```

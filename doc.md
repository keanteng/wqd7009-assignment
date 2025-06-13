# Using HBase with Docker

## Code used

```bash
docker exec -it hbase bash
mkdir test
cat >> test.csv # can use vi if want to insert a lot of data
1,2020,199
2,2020,2992
3,2900,3929
^Z
awk 'BEGIN {FS=","; OFS="\t"} {$1=$1; print}' test/test.csv > test/test.tsv

hbase shell
create "table", "year", "power"
exit

separator=$(echo -e "\t")
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \
-Dimporttsv.separator=$separator \
-Dimporttsv.columns=HBASE_ROW_KEY,year:year,power:power \
table \
test/test.tsv
```

---

## Assignment use case documentation

### Importing Data

```bash
# load the data
ls
pwd
mkdir data
vi data/data.csv
awk 'BEGIN {FS=","; OFS="\t"} {$1=$1; print}' data/data.csv > data/data.tsv
cat data/data.tsv

# create a table in hbase
hbase shell
version
create "data_table", "year", "cf"
list
exit

# moving to hbase
separator=$(echo -e "\t")
hbase org.apache.hadoop.hbase.mapreduce.ImportTsv \
-Dimporttsv.separator=$separator \
-Dimporttsv.columns=HBASE_ROW_KEY,year:year,cf:energy_from_renewable_and_waste_sources,cf:total_energy_consumptions_of_primary_fuels_and_equivalents,cf:fraction_from_renewable_source_and_waste,cf:hydroelectric_power,cf:wind_wave_tidal,cf:solar_photovoltaic,cf:landfill_gas,cf:sewage_gas,cf:biogas_from_autogen,cf:municipal_solid_waste,cf:poultry_litter,cf:straw,cf:wood,cf:charcoal \
data_table \
data/data.tsv

```

> hadoop dfs -copyFromLocal data/data.tsv /tmp
> hdfs:///tmp/data.tsv

## DDL Command

No, using the alter command to modify a column family in HBase does not change the name of the column family. The alter command is used to modify properties of the column family, such as the number of versions, compression settings, etc. The name of the column family remains the same.

```bash
create 'abc', 'cf'
list
disable <'table name'>
is_disabled <'table name'>
enable <'table name'>
is_enabled <'table name'>
drop <'table name'>
exists <'table name'>

# working with alter
# Add a new column family
alter 'data_table', { NAME => 'new_column_family' }

# Modify an existing column family
alter 'data_table', { NAME => 'cf', VERSIONS => 5 }

# Delete a column family
alter 'data_table', { NAME => 'cf', METHOD => 'delete' }

# Verify the changes
describe 'data_table'
```

### DML Command

```bash
get "data_table", "1"
count "data_table"

# using sample as an example table in hbase
put "sample", "1", "cf:age", "999"
put "sample", "1", "cf:name", "hellow owrld"

delete "sample", "1", "cf:name" # coulmn name deleted
deleteall "sample", "1" # all row 1 disappear
truncate "sample" # whole table disappear
```

## Extra commands

```bash
whoami
table_help
status
show_filters
is_disabled <'table name'>
```

## Useful Analytics

```bash
# Filter rows with columns that have a specific prefix
scan 'data_table', { FILTER => "ColumnPrefixFilter('prefix')" }


```
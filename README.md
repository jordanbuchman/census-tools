# census-tools

These tools are for importing the 2010 census summary file 1 into a database that is not completely terrible. (Microsoft Access)

This readme assumes sqlite is being used, but I have also included a schema for mysql. The scripts likely need editing if mysql will be used. Another option is to follow the sqlite instructions and convert to mysql at the very end.

```bash
sudo apt-get update
sudo apt-get install sqlite3 libsqlite3-dev
```

## Directory Structure

```
  census-tools
  /          \
data        sql_scripts
           /     |     \
         bin    sql    template
```

## Acquiring Census Data

Download the census data from [this link to census.gov's summary file](https://www2.census.gov/census_2010/04-Summary_File_1/) and put it in a folder inside this repository (the exact path doesn't matter, it just needs to be a subdirectory of this folder).

You can run this command to download everything.

```bash
wget -m -e robots=off --no-parent https://www2.census.gov/census_2010/04-Summary_File_1/Urban_Rural_Update/
```

After downloading, you need to unzip all the `.ur1` files present in the download. You can accomplish this by running this command somewhere inside this repository.
```bash
find . -name "*.zip" | while read filename; do unzip -o -d "`dirname "$filename"`" "$filename"; done;
```

## Creating and Importing to the Database
NOTE: This will take a long time!
```bash
sql_scripts/bin/create_census_db
sql_scripts/bin/import_census
```

## Converting sqlite to mysql

dump_to_mysql.py can convert sqlite to mysqsl using the command:

```bash
sqlite3 SF1_2010 .dump | python ../sql_scripts/dump_to_mysql.py > dump.sql
```

[Original code](http://stackoverflow.com/a/1067365/6731537)

## Example Query

```sql
SELECT P0010001 
FROM SF1_00001 AS S 
JOIN (
  SELECT LOGRECNO
  FROM GEO_HEADER_SF1 AS G
  WHERE G.SUMLEV="871" AND G.STUSAB="NE" AND G.ZCTA5="68106"
) AS G 
ON S.LOGRECNO=G.LOGRECNO AND S.STUSAB="NE";
```

* P0010001 is the column for total population.
* SF1_00001 is the obscurely named table containing the population information.
* GEO_HEADER_SF1 is the table containing geographic information, we need this to find specific locations.
* LOGRECNO and STUSAB (State abbreviation) together are a unique identifier.
* SUMLEV is the summary level, or how the data is broken into chunks. "871" is per zipcode.
* ZCTA5 is the zip code in question.

## Resources

Here are some useful files to help navigate the terrible organization of the census data.

* [Offical Sf1 documentation](http://www.census.gov/prod/cen2010/doc/sf1.pdf)
* [Sf1 quick reference](https://wagda.lib.washington.edu/data/type/census/geodb/metadata/SF1qkRef_2010.pdf)

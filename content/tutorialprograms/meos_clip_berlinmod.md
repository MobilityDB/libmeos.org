---
title: "Clipping Trips to Geometries"
date: 2022-07-29T13:57:17+02:00
draft: false
---

[meos_clip_berlinmod.c](https://github.com/MobilityDB/MobilityDB/blob/develop/meos/examples/06_meos_clip_berlinmod.c)

This program reads a CSV file containing synthetic trip data in Brussels generated by the [MobilityDB-BerlinMOD](https://github.com/MobilityDB/MobilityDB-BerlinMOD) generator and computes the distance traversed by the trips in the 19 [Brussels municipalities](https://en.wikipedia.org/wiki/List_of_municipalities_of_the_Brussels-Capital_Region) (*communes* in French).

The output of the program is given next.
```
19 commune records read
Brussels region record read
Reading trip records
*******************************************************
55 trip records read.


                -----------------------------------------------------------------------------------------------------------------------------------------
                | Commmunes
    -----------------------------------------------------------------------------------------------------------------------------------------------------
Veh | Distance |     2       4       6       7       9      10      11      12      14      15      16      17      18      19   |  Inside | Outside
---------------------------------------------------------------------------------------------------------------------------------------------------------
  1 |  256.641 |  14.585   0.000   0.000   0.000   0.000   0.000   0.000   0.000   0.000   0.000   8.223  11.915   0.000   0.000 |  34.724 | 221.917
  2 |  126.289 |   0.000   0.000   0.000   0.000   0.000   0.000   0.000   0.000   0.000  77.281   4.583  28.809   0.000   0.000 | 110.673 |  15.616
  3 |  316.302 |   0.000   5.194  56.414   0.000   0.000   0.000   0.000  17.132   9.017   0.000  48.214   0.000  74.502   1.951 | 212.425 | 103.877
  4 |   90.095 |   0.000   0.000   0.000  18.094  35.514  10.832  11.738   0.000   0.000   0.000  13.917   0.000   0.000   0.000 |  90.095 |   0.000
  5 |  147.076 |   0.000   0.000   0.000   0.000   0.000   0.000   4.170   0.000   0.000   0.000 107.440   0.000   0.000   0.000 | 111.611 |  35.465
---------------------------------------------------------------------------------------------------------------------------------------------------------
    |  936.402 |  14.585   5.194  56.414  18.094  35.514  10.832  15.908  17.132   9.017  77.281 182.377  40.724  74.502   1.951 | 559.527 | 376.875
---------------------------------------------------------------------------------------------------------------------------------------------------------
```
Notice that communes having a total distance of zero are not shown above. To show all communes it suffices to set a Boolean flag in the program. 

A similar result can be obtained in MobilityDB with the following SQL queries, assuming that the CSV files have been previously loaded into the `trips`, `brusselsRegion`, and `communes` table.
```sql
SELECT vehid, 
  to_char(SUM(length(trip)) / 1e3,'999990D999') AS totdist, 
  to_char(SUM(length(atGeometry(trip, b.geom))) / 1e3,'999990D999') AS inside,
  to_char((SUM(length(trip)) - SUM(length(atGeometry(trip, b.geom)))) / 1e3, '999990D999') AS outside
FROM trips t, brusselsRegion b
GROUP BY GROUPING SETS ((vehid), ())
ORDER BY vehid;

 vehid |   totdist   |   inside    |   outside
-------+-------------+-------------+-------------
     1 |     256.641 |      34.724 |     221.917
     2 |     126.289 |     110.673 |      15.616
     3 |     316.302 |     212.425 |     103.877
     4 |      90.095 |      90.095 |       0.000
     5 |     147.076 |     111.611 |      35.465
       |     936.402 |     559.527 |     376.875

SELECT t.vehid, c.id AS commId, 
  to_char(SUM(length(atGeometry(t.trip, c.geom))) / 1e3,'999990D999') AS commdist
FROM trips t, communes c
GROUP BY GROUPING SETS ((vehid, c.id),(c.id))
HAVING SUM(length(atGeometry(t.trip, c.geom))) IS NOT NULL
ORDER BY t.vehid, c.id;

 vehid | commid |  commdist
-------+--------+-------------
     1 |      2 |      14.585
     1 |     16 |       8.223
     1 |     17 |      11.915
     2 |     15 |      77.281
     2 |     16 |       4.583
     2 |     17 |      28.809
     3 |      4 |       5.194
     3 |      6 |      56.414
     3 |     12 |      17.132
     3 |     14 |       9.017
     3 |     16 |      48.214
     3 |     18 |      74.502
     3 |     19 |       1.951
     4 |      7 |      18.094
     4 |      9 |      35.514
     4 |     10 |      10.832
     4 |     11 |      11.738
     4 |     16 |      13.917
     5 |     11 |       4.170
     5 |     16 |     107.440
       |      2 |      14.585
       |      4 |       5.194
       |      6 |      56.414
       |      7 |      18.094
       |      9 |      35.514
       |     10 |      10.832
       |     11 |      15.908
       |     12 |      17.132
       |     14 |       9.017
       |     15 |      77.281
       |     16 |     182.377
       |     17 |      40.724
       |     18 |      74.502
       |     19 |       1.951
```


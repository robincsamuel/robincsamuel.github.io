---
layout: page
title: Calculate distance between two latitude – longitude points using MySQL.
description: Here is a function to calculate distance between two latitude – longitude points. This function can be used to calculate distance either using input values or SQL join queries.
permalink: /calculate-distance-between-two-latitude-longitude-points-using-mysql/
---

# Calculate distance between two latitude – longitude points using MySQL.

Here is a function to calculate distance between two latitude – longitude points. This function can be used to calculate distance either using input values or SQL join queries. Function accepts four parameters, source latitude, source longitude, destination latitude, destination longitude and returns the distance between the points in kilometers.

```mysql
DELIMITER $$
CREATE FUNCTION `distance`(Q_LAT FLOAT, Q_LONG FLOAT, NAV_LAT FLOAT, NAV_LONG FLOAT) RETURNS float
BEGIN
    DECLARE radlat1 FLOAT;
    DECLARE radlat2 FLOAT;
    DECLARE radlon1 FLOAT;
    DECLARE radlon2 FLOAT;
    DECLARE theta FLOAT;
    DECLARE radtheta FLOAT;
    DECLARE dist FLOAT;
    DECLARE PI FLOAT;

    SET PI = PI();
    SET dist = 0;

    IF ((Q_LAT IS NULL OR Q_LAT = 0) OR (Q_LONG IS NULL OR Q_LONG = 0)
        OR (NAV_LAT IS NULL OR NAV_LAT = 0) OR (NAV_LONG IS NULL OR NAV_LONG = 0)) THEN
        RETURN dist;
    ELSE
        SET radlat1 = PI * (Q_LAT/180);
        SET radlat2 = PI * (NAV_LAT/180);
        SET radlon1 = PI * (Q_LONG/180);
        SET radlon2 = PI * (NAV_LONG/180);
        SET theta = Q_LONG-NAV_LONG;
        SET radtheta = PI * (theta/180);
        SET dist = SIN(radlat1) * SIN(radlat2) + COS(radlat1) * COS(radlat2) * COS(radtheta);
        SET dist = ACOS(dist);
        SET dist = dist * (180/PI);
        SET dist = dist * 60 * 1.1515;
        SET dist = dist * 1.609344;

        SET dist = CEILING(dist);

    RETURN dist;
    END IF;
END$$
```

Run the above SQL query to create a function named ‘distance’. After executing the query, function can be used to calculate distance between two points. For example, Simple query

```mysql
SELECT distance(9.992952,76.296208,10.014691,76.363939)
```

### Query with Join

```mysql
SELECT`id`, distance(9.992952,76.296208,`b`.`latitude`,`b`.`longitude`)
FROM `a` JOIN `table` as `c` on `a`.`id` = `b`.`user_id`;
```

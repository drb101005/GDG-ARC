WITH
Shots AS
(
 SELECT
  *,

  /* 101 is known Tag for 'goals' from goals table */
  (101 IN UNNEST(tags.id)) AS isGoal,

  /* Translate 0-100 (x,y) coordinates to absolute positions using "average"
  field dimensions of 105x68 before using in various distance calcs;
  LEAST used to cap shot locations to on-field (x, y) (i.e. no exact 100s) */
  LEAST(positions[ORDINAL(1)].x, 99.99999) * 105/100 AS shotXAbs,
  LEAST(positions[ORDINAL(1)].y, 99.99999) * 68/100 AS shotYAbs

 FROM
   `soccer.events`

 WHERE
   /* Includes both "open play" & free kick shots (including penalties) */
   eventName = 'Shot' OR
   (eventName = 'Free Kick' AND subEventName IN ('Free kick shot', 'Penalty'))
),

ShotsWithAngle AS
(
 SELECT
   Shots.*,

   /* Law of cosines to get 'open' angle from shot location to goal, given
    that goal opening is 7.32m, placed midway up at field end of (105, 34) */
   SAFE.ACOS(
     SAFE_DIVIDE(
       ( /* Squared distance between shot and 1 post, in meters */
         (POW(105 - shotXAbs, 2) + POW(34 + (7.32/2) - shotYAbs, 2)) +
         /* Squared distance between shot and other post, in meters */
         (POW(105 - shotXAbs, 2) + POW(34 - (7.32/2) - shotYAbs, 2)) -
         /* Squared length of goal opening, in meters */
         POW(7.32, 2)
       ),
       (2 *
         /* Distance between shot and 1 post, in meters */
         SQRT(POW(105 - shotXAbs, 2) + POW(34 + 7.32/2 - shotYAbs, 2)) *
         /* Distance between shot and other post, in meters */
         SQRT(POW(105 - shotXAbs, 2) + POW(34 - 7.32/2 - shotYAbs, 2))
       )
     )
   /* Translate radians to degrees */
   ) * 180 / ACOS(-1)
   AS shotAngle

 FROM
   Shots
)

SELECT
 ROUND(shotAngle, 0) AS ShotAngleRound0,

 COUNT(*) AS numShots,
 SUM(IF(isGoal, 1, 0)) AS numGoals,
 AVG(IF(isGoal, 1, 0)) AS goalPct

FROM
 ShotsWithAngle

GROUP BY
 ShotAngleRound0

ORDER BY
 ShotAngleRound0
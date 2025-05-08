SELECT
 Events.playerId,
 (Players.firstName || ' ' || Players.lastName) AS playerName,
 SUM(IF(Tags2Name.Label = 'assist', 1, 0)) AS numAssists

FROM
 `soccer.events` Events,
 Events.tags Tags

LEFT JOIN
 `soccer.tags2name` Tags2Name ON
   Tags.id = Tags2Name.Tag

LEFT JOIN
 `soccer.players` Players ON
   Events.playerId = Players.wyId

GROUP BY
 playerId, playerName

ORDER BY
 numAssists DESC
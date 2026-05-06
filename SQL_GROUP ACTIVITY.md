

1. Download the files
2. Import and create the table for the files

# CLEAN THE DATA
```SQL
SELECT 
    pokedex_messy.pokemon_id,
    pokedex_messy.name,
    MAX(UPPER(pokedex_messy.type1)) AS type1,
    MAX(UPPER(pokedex_messy.type2)) AS type2,
    pokemon_stats.hp,
    pokemon_stats.attack,
    pokemon_stats.defense,
    pokedex_messy.evolution_stage,
    pokedex_messy.evolves_to
FROM pokedex_messy
JOIN pokemon_stats
    ON pokedex_messy.pokemon_id = pokemon_stats.pokemon_id
   AND pokedex_messy.name = pokemon_stats.name
GROUP BY pokedex_messy.pokemon_id
ORDER BY CAST(pokedex_messy.pokemon_id AS integer) ASC;
```

```sql
-- SHORTCUT

SELECT 
    pm.pokemon_id,
    pm.name,
    MAX(UPPER(pm.type1)) AS type1,
    MAX(UPPER(pm.type2)) AS type2,
    ps.hp,
    ps.attack,
    ps.defense,
    pm.evolution_stage,
    pm.evolves_to
FROM pokedex_messy AS pm
JOIN pokemon_stats AS ps
    ON pm.pokemon_id = ps.pokemon_id
   AND pm.name = ps.name
GROUP BY pm.pokemon_id
ORDER BY CAST(pm.pokemon_id AS integer) ASC;
```

# CREATE THE NEW TABLE
```SQL
CREATE TABLE pokedex_clean AS
SELECT 
    pokedex_messy.pokemon_id,
    pokedex_messy.name,
    MAX(UPPER(pokedex_messy.type1)) AS type1,
    MAX(UPPER(pokedex_messy.type2)) AS type2,
    pokemon_stats.hp,
    pokemon_stats.attack,
    pokemon_stats.defense,
    pokedex_messy.evolution_stage,
    pokedex_messy.evolves_to
FROM pokedex_messy
JOIN pokemon_stats
    ON pokedex_messy.pokemon_id = pokemon_stats.pokemon_id
   AND pokedex_messy.name = pokemon_stats.name
GROUP BY pokedex_messy.pokemon_id
ORDER BY CAST(pokedex_messy.pokemon_id AS integer) ASC;
```

```sql
-- SHORTCUT
DROP TABLE IF EXISTS pokedex_clean;

CREATE TABLE pokedex_clean AS
SELECT 
    pm.pokemon_id,
    pm.name,
    MAX(UPPER(pm.type1)) AS type1,
    MAX(UPPER(pm.type2)) AS type2,
    ps.hp,
    ps.attack,
    ps.defense,
    pm.evolution_stage,
    pm.evolves_to
FROM pokedex_messy AS pm
JOIN pokemon_stats AS ps
    ON pm.pokemon_id = ps.pokemon_id
   AND pm.name = ps.name
GROUP BY pm.pokemon_id
ORDER BY CAST(pm.pokemon_id AS integer) ASC;
```
# VIEW ENEMY TEAM
```sql
SELECT *
FROM pokedex_clean
WHERE name IN (
    'Starmie',
    'Gyarados',
    'Lapras',
    'Vaporeon',
    'Jynx',
    'Alakazam'
);
```

# TO SEE THE HIGHEST STATS
```SQL
SELECT name, type1, hp + attack + defense AS 'Stats' FROM pokedex_clean ORDER BY Stats DESC;
```

# SORT THE TYPE_ADVANTAGES
```SQL
SELECT *
FROM type_advantage
ORDER BY attacking_type;
```

# TO QUERY THE POSSIBLE CANDIDATE MATCH_UPS
```SQL

WITH pokemon_rankings AS (
    SELECT 
        pokedex_clean.name,
        pokedex_clean.type1,
        pokedex_clean.type2,
        -- FIXED: Cast to FLOAT to prevent integer division errors
        ROUND(
            (CAST(pokedex_clean.hp AS FLOAT) + pokedex_clean.attack + pokedex_clean.defense) / 3.0,
            2
        ) AS stats_score,
        SUM(
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type1)
                  AND UPPER(defending_type) = UPPER(misty_team.type1)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
            *
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type1)
                  AND UPPER(defending_type) = UPPER(misty_team.type2)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
            *
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type2)
                  AND UPPER(defending_type) = UPPER(misty_team.type1)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
            *
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type2)
                  AND UPPER(defending_type) = UPPER(misty_team.type2)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
        ) AS matchup_score
    FROM pokedex_clean
    CROSS JOIN misty_team
    GROUP BY 
        pokedex_clean.name,
        pokedex_clean.type1,
        pokedex_clean.type2,
        pokedex_clean.hp,
        pokedex_clean.attack,
        pokedex_clean.defense
)

SELECT *
FROM pokemon_rankings
ORDER BY matchup_score DESC, stats_score DESC;
```

# CREATE THE CANDIDATE MATCHUPS / THE 12 POKEMONS

```SQL
DROP TABLE IF EXISTS candidate_matchups;

CREATE TABLE candidate_matchups AS
WITH pokemon_rankings AS (
    SELECT 
        pokedex_clean.name,
        pokedex_clean.type1,
        pokedex_clean.type2,
        ROUND(
            (CAST(pokedex_clean.hp AS FLOAT) + pokedex_clean.attack + pokedex_clean.defense) / 3.0,
            2
        ) AS stats_score,
        SUM(
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type1)
                  AND UPPER(defending_type) = UPPER(misty_team.type1)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
            *
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type1)
                  AND UPPER(defending_type) = UPPER(misty_team.type2)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
            *
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type2)
                  AND UPPER(defending_type) = UPPER(misty_team.type1)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
            *
            (
                SELECT CASE
                    WHEN effectiveness = 'super effective' THEN 2.0
                    WHEN effectiveness = 'not very effective' THEN 0.5
                    WHEN effectiveness = 'no effect' THEN 0.0
                    ELSE 1.0
                END
                FROM type_advantage
                WHERE UPPER(attacking_type) = UPPER(pokedex_clean.type2)
                  AND UPPER(defending_type) = UPPER(misty_team.type2)

                UNION ALL

                SELECT 1.0
                LIMIT 1
            )
        ) AS total_advantage
    FROM pokedex_clean
    CROSS JOIN misty_team
    WHERE 
        pokedex_clean.evolves_to IS NULL
        OR pokedex_clean.evolves_to = ''
        OR pokedex_clean.evolves_to = 'None'
    GROUP BY 
        pokedex_clean.name,
        pokedex_clean.type1,
        pokedex_clean.type2,
        pokedex_clean.hp,
        pokedex_clean.attack,
        pokedex_clean.defense
)

SELECT 
    name,
    type1,
    type2,
    stats_score,
    total_advantage,
    ROUND((total_advantage * 10) + stats_score, 2) AS final_score
FROM pokemon_rankings
ORDER BY final_score DESC;

SELECT *
FROM candidate_matchups;
```


# TO CREATE MISTY_TEAM
```SQL
DROP TABLE IF EXISTS misty_team;

CREATE TABLE misty_team (
    pokemon_id INTEGER,
    name TEXT,
    type1 TEXT,
    type2 TEXT,
    hp INTEGER,
    attack INTEGER,
    defense INTEGER,
    evolution_stage INTEGER,
    evolves_to TEXT
);

INSERT INTO misty_team (
    pokemon_id, name, type1, type2, hp, attack, defense, evolution_stage, evolves_to
)
SELECT
    pokemon_id, name, type1, type2, hp, attack, defense, evolution_stage, evolves_to
FROM pokedex_clean
WHERE pokemon_id IN (65, 121, 124, 130, 131, 134);
```

# TO CREATE CANDIDATE_MATCHUPS
```SQL
-- THIS ONLY LISTS THE ADVANTAGE TYPE COLUMN

DROP TABLE IF EXISTS candidate_matchups;

CREATE TABLE candidate_matchups AS
WITH type_values AS (  
SELECT   
UPPER(attacking_type) AS attacking_type,  
UPPER(defending_type) AS defending_type,  
  CASE effectiveness  
    WHEN 'super effective' THEN 2.0  
    WHEN 'not very effective' THEN 0.5  
    WHEN 'no effect' THEN 0.0  
    ELSE 1.0  
  END AS multiplier  
FROM type_advantage  
),  
candidates AS (  
SELECT  
  name,  
  type1,  
  type2,  
  attack,  
  ROUND((CAST(hp AS INTEGER) + CAST(attack AS INTEGER) + CAST(defense AS INTEGER)) / 3.0, 2) AS stat_score  
FROM pokedex_clean  
WHERE evolves_to IS NULL OR evolves_to = ''  
),  
matchups AS (  
SELECT   
 c.name AS candidate_name,  
 c.type1,  
 c.type2,  
 c.stat_score,  
 m.name AS opponent_name,  
  
COALESCE((SELECT multiplier FROM type_values WHERE attacking_type = c.type1 AND defending_type = m.type1), 1.0) *  
COALESCE((SELECT multiplier FROM type_values WHERE attacking_type = c.type1 AND defending_type = m.type2), 1.0) AS t1_mult,  
  
CASE WHEN c.type2 IS NOT NULL AND c.type2 != '' THEN  
   COALESCE((SELECT multiplier FROM type_values WHERE attacking_type = c.type2 AND defending_type = m.type1), 1.0) *  
   COALESCE((SELECT multiplier FROM type_values WHERE attacking_type = c.type2 AND defending_type = m.type2), 1.0)  
   ELSE 0.0  
  END AS t2_mult  
  
FROM candidates c  
CROSS JOIN misty_team m  
),  
best_attack AS (  
SELECT   
 candidate_name,  
 type1,  
 type2,  
 stat_score,  
MAX(t1_mult, t2_mult) AS best_multiplier  
FROM matchups  
)  
  
SELECT  
 candidate_name AS name,  
 type1,  
 type2,  
 stat_score,  
SUM(best_multiplier) AS total_advantage  
FROM best_attack  
GROUP BY  
 candidate_name,  
 type1,  
 type2,  
 stat_score  
ORDER BY  
 total_advantage DESC,  
 stat_score DESC;
```

# CREATE RANKED_CANDIDATES TABLE
```SQL
DROP TABLE IF EXISTS ranked_candidates;

CREATE TABLE ranked_candidates AS
WITH type_values AS (
    SELECT 
        UPPER(attacking_type) AS attacking_type,
        UPPER(defending_type) AS defending_type,
        CASE effectiveness
            WHEN 'super effective' THEN 2.0
            WHEN 'not very effective' THEN 0.5
            WHEN 'no effect' THEN 0.0
            ELSE 1.0
        END AS multiplier
    FROM type_advantage
),

candidates AS (
    SELECT
        name,
        type1,
        type2,
        attack,
        ROUND(
            (CAST(hp AS INTEGER) + CAST(attack AS INTEGER) + CAST(defense AS INTEGER)) / 3.0,
            2
        ) AS stat_score
    FROM pokedex_clean
    WHERE evolves_to IS NULL OR evolves_to = ''
),

matchups AS (
    SELECT 
        c.name AS candidate_name,
        c.type1,
        c.type2,
        c.stat_score,
        m.name AS opponent_name,

        COALESCE(
            (
                SELECT multiplier 
                FROM type_values 
                WHERE attacking_type = c.type1 
                  AND defending_type = m.type1
            ),
            1.0
        ) *
        COALESCE(
            (
                SELECT multiplier 
                FROM type_values 
                WHERE attacking_type = c.type1 
                  AND defending_type = m.type2
            ),
            1.0
        ) AS t1_mult,

        CASE 
            WHEN c.type2 IS NOT NULL AND c.type2 != '' THEN
                COALESCE(
                    (
                        SELECT multiplier 
                        FROM type_values 
                        WHERE attacking_type = c.type2 
                          AND defending_type = m.type1
                    ),
                    1.0
                ) *
                COALESCE(
                    (
                        SELECT multiplier 
                        FROM type_values 
                        WHERE attacking_type = c.type2 
                          AND defending_type = m.type2
                    ),
                    1.0
                )
            ELSE 0.0
        END AS t2_mult
    FROM candidates c
    CROSS JOIN misty_team m
),

best_attack AS (
    SELECT 
        candidate_name,
        type1,
        type2,
        stat_score,
        MAX(t1_mult, t2_mult) AS best_multiplier
    FROM matchups
)

SELECT
    candidate_name AS name,
    type1,
    type2,
    stat_score,
    SUM(best_multiplier) AS total_advantage,
    ROUND((SUM(best_multiplier) * 10) + stat_score, 2) AS final_score,
    ROW_NUMBER() OVER (
        ORDER BY 
            ROUND((SUM(best_multiplier) * 10 + stat_score), 2) DESC,
            SUM(best_multiplier) DESC,
            candidate_name ASC
    ) AS rank
FROM best_attack
GROUP BY
    candidate_name,
    type1,
    type2,
    stat_score
ORDER BY
    rank ASC;
```

# TO CREATE TWO_COMPATIBLE_TEAMS
```SQL
SELECT  
 name,  
CASE   
   WHEN rank % 2 = 1 THEN 'Team A'  
   ELSE 'Team B'  
END AS assigned_teams  
FROM ranked_candidates  
WHERE rank <= 12  
ORDER BY  
 assigned_teams,  
 rank ASC;
```
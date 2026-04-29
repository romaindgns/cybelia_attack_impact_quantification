'''cypher

WITH ['NoeudsPhysique','NoeudsActuator','NoeudsSensor','NoeudsS','NoeudsH','NoeudsEndUsers','NoeudsUsers','NoeudsOpérateurs'] AS NoeudsCibles

// Construire les "projections" = groupes de cibles
UNWIND NoeudsCibles AS proj
MATCH (t)
WHERE proj IN labels(t)
WITH proj, collect(t) AS targets, size(collect(t)) AS totalTargets

// Calcul pour tous les nœuds
MATCH (n)
WITH n, proj, targets, totalTargets

UNWIND targets AS t
WITH n, proj, t, totalTargets
WHERE n <> t

OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))
WITH n, proj, totalTargets, length(p) AS dist

WITH n, proj, totalTargets,
     CASE 
       WHEN dist IS NULL THEN 0.0
       ELSE 1.0 / dist
     END AS contrib

WITH n, proj, totalTargets, sum(contrib) / totalTargets AS score

WITH n, collect({proj: proj, score: score}) AS pairs

WITH n,
     reduce(m = {}, p IN pairs | apoc.map.setKey(m, p.proj, p.score)) AS scores

// Regroupement des données
RETURN labels(n) AS node,
       coalesce(scores.NoeudsPhysique, 0) AS PhysiqueOT,
       coalesce(scores.NoeudsActuator, 0) AS ActuatorOT,
       coalesce(scores.NoeudsSensor, 0) AS SensorOT,
       coalesce(scores.NoeudsS, 0) AS SoftwareIT,
       coalesce(scores.NoeudsH, 0) AS HardwareIT,
       coalesce(scores.NoeudsEndUsers, 0) AS EndUsers,
       coalesce(scores.NoeudsUsers, 0) AS Users,
       coalesce(scores.NoeudsOperators, 0) AS Operators
ORDER BY node
'''

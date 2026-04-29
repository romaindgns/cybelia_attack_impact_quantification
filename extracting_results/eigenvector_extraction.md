```cypher

WITH ['graphePhysiqueOT','grapheActuatorOT','grapheSensorOT','grapheCyberOT','grapheMissionOT','grapheSIT','grapheHIT','grapheCyberIT','grapheMissionIT','graphePpEndUsers','graphePpUsers','graphePpOperators','graphePpMis'] AS graphs

// Prendre tous les projections
CALL (graphs){
  WITH graphs
  UNWIND graphs AS g
  CALL gds.eigenvector.stream(g)
  YIELD nodeId, score
  RETURN gds.util.asNode(nodeId) AS n, g AS proj, score
}

// On associe les scores à chaque nœud
WITH n, collect({proj: proj, score: score}) AS pairs


WITH n,
     reduce(m = {}, p IN pairs | apoc.map.setKey(m, p.proj, p.score)) AS scores

// Regroupement des données
RETURN labels(n) AS node,
       coalesce(scores.graphePhysiqueOT, 0) AS PhysiqueOT,
       coalesce(scores.grapheActuatorOT, 0) AS ActuatorOT,
       coalesce(scores.grapheSensorOT, 0) AS SensorOT,
       coalesce(scores.grapheCyberOT, 0) AS CyberOT,
       coalesce(scores.grapheMissionOT, 0) AS MissionOT,
       coalesce(scores.grapheSIT, 0) AS SoftwareIT,
       coalesce(scores.grapheHIT, 0) AS HardwareIT,
       coalesce(scores.grapheCyberIT, 0) AS CyberIT,
       coalesce(scores.grapheMissionIT, 0) AS MissionIT,
       coalesce(scores.graphePpEndUsers, 0) AS EndUsers,
       coalesce(scores.graphePpUsers, 0) AS Users,
       coalesce(scores.graphePpOperators, 0) AS Operators,
       coalesce(scores.graphePpMis, 0) AS MissionPP
ORDER BY node
'''

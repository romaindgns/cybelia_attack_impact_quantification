# Cypher Queries: Graph Analytics with Harmonic Centrality on the Use Case 

Required: Projection queries used for the eigenvector centrality (at least the creation of temporary labels for the targeted nodes)

# Table of contents
1. [Harmonic Centrality OT](#1)
    1. [Eigenvector Physical](#1.1)
    2. [Eigenvector Sensor](#1.2)
    3. [Eigenvector Actuator](#1.3)
2. [Harmonic Centrality IT](#2)
    1. [Eigenvector Hardware](#2.1)
    2. [Eigenvector Software](2.2)
3. [Harmonic Centrality Service](#3)
    1. [Eigenvector Users](#3.1)
    2. [Eigenvector EndUsers](3.2)
    3. [Eigenvector Operators](#3.3)

## Harmonic Centrality OT <a name="1"></a>

### Harmonic Physical <a name="1.1"></a>

```cypher
//Harmonic Target Physical Normalized
MATCH (t:NoeudsPhysique)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

### Harmonic Sensor <a name="1.2"></a>

```cypher
//Harmonic Target OT Normalized
MATCH (t:NoeudsSensor)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

### Harmonic Actuator <a name="1.3"></a>

```cypher
//Harmonic Target OT Normalized
MATCH (t:NoeudsActuator)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

## Harmonic Centrality IT <a name="2"></a>

### Harmonic Hardware <a name="2.1"></a>

```cypher
//Harmonic Target IT Normalized Hard
MATCH (t:NoeudsH)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

### Harmonic Software <a name="2.2"></a>

```cypher
//Harmonic Target IT Normalized Soft
MATCH (t:NoeudsS)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

## Harmonic Centrality Service <a name="3"></a>

### Harmonic Users <a name="3.1"></a>

```cypher
//Harmonic Target PP Normalized User
MATCH (t:NoeudsUsers)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

### Harmonic EndUsers <a name="3.2"></a>

```cypher
//Harmonic Target PP Normalized User
MATCH (t:NoeudsEndUsers)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

### Harmonic Operators <a name="3.3"></a>

```cypher
//Harmonic Target PP Normalized User
MATCH (t:NoeudsOperators)
WITH collect(t) AS targets, size(collect(t)) AS totalTargets

//Parcourir tous les noeuds et calculer distances
MATCH (n)
WITH n, targets, totalTargets
UNWIND targets AS t
WITH n, t, totalTargets
WHERE n <> t
OPTIONAL MATCH p = shortestPath((n)-[*..5]->(t))  // limite à 5 sauts
WITH n, totalTargets, collect(length(p)) AS distances

//Filtrer et calculer la closeness ciblée
WITH n, totalTargets, [d IN distances WHERE d IS NOT NULL] AS validDistances
WHERE size(validDistances) > 0
WITH n, 
     reduce(total=0.0, d IN validDistances | total + (1.0 / d)) AS harmonicCentrality,
     totalTargets

//Retourner les résultats triés avec le label principal
RETURN labels(n)[0] AS node, harmonicCentrality / totalTargets AS closeness
ORDER BY closeness DESC
```

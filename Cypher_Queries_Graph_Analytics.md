# Cypher Queries: Graph Analytics on the Use Case 

# Table of contents
1. [Projection for Eigenvector Centrality using GDS](#1)
    1. [Projection OT](#1.1)
    2. [Projection IT](#1.2)
    3. [Projection Service](#1.3)
2. [Eigenvector Centrality OT](#2)
    1. [Eigenvector Physical](#2.1)
    2. [Eigenvector Sensor](#2.2)
    3. [Eigenvector Actuator](#2.3)
    4. [Eigenvector Cyber OT](#2.4)
    5. [Eigenvector Mission OT](#2.5)
3. Eigenvector Centrality IT(#3)
    1. [Eigenvector Hardware](#3.1)
    2. [Eigenvector Software](3.2)
    3. [Eigenvector Cyber IT](#3.3)
    4. [Eigenvector Mission IT](#3.4)
3. Eigenvector Centrality Services(#3)
    a. [Queries for the process](#3.1)
    b. [Queries for the process measurements](3.2)
    c. [Queries for the process](#3.3)
    d. [Queries for the process measurements](#3.4)
    e. [Queries for the process measurements](#3.5)
7. [Queries to link the components of the IT infrastructure](#4)
    1. [Liens IT UP](#4.1)
    2. [Liens IT DOWN](#4.2)
8. [Queries to link the components of the IT infrastructure](#5)
9. [Queries to link the components of the IT infrastructure](#6)
   
## Projection for Eigenvector Centrality using GDS  <a name="1"></a>

###Projection OT<a name="1.1"></a>
```cypher
//Projections OT

// Création de labels temporaires pour les noeuds PSA
MATCH (n)
WHERE n.couche IN ["actuator","sensor","physique"]
SET n:NoeudsPSA;

// Création de labels temporaires pour les noeuds P
MATCH (n)
WHERE n.couche IN ["physique"]
SET n:NoeudsPhysique;

// Création de labels temporaires pour les noeuds S
MATCH (n)
WHERE n.couche IN ["sensor"]
SET n:NoeudsSensor;

// Création de labels temporaires pour les noeuds A
MATCH (n)
WHERE n.couche IN ["actuator"]
SET n:NoeudsActuator;

//Projection PSA
CALL gds.graph.project(
  'graphePSA',
  'NoeudsPSA',
  ["PHYS_ELEC","PHYS_ATTACH","PHYS_CNTCT","ACT_ISOLATE_LINE","ACT_CUT_LINE","ACT_ADAPT_CRRT","ACT_ADAPT_VLTG","ACT_MDF_PWR","SENS_MECA","SENS_STATE","SENS_ELEC"]
);

//Projection Physical OT
CALL gds.graph.project(
  'graphePhysiqueOT',
  "NoeudsPhysique",
  ["PHYS_ELEC","PHYS_ATTACH","PHYS_CNTCT"]
);

//Projection Sensor OT
CALL gds.graph.project(
  'grapheSensorOT',
  "NoeudsSensor",
  ["SENS_MECA","SENS_STATE","SENS_ELEC"]
);

//Projection Actuator OT
CALL gds.graph.project(
  'grapheActuatorOT',
  "NoeudsActuator",
  ["ACT_ISOLATE_LINE","ACT_CUT_LINE","ACT_ADAPT_CRRT","ACT_ADAPT_VLTG","ACT_MDF_PWR"]
);

//Projection Cyber OT
CALL gds.graph.project(
  'grapheCyberOT',
  '*',
  ["CYBER_ORDER_OT","CYBER_MECA_OT","CYBER_ELEC_OT","CYBER_DATA_OT"]
);

//Projection Missions OT
CALL gds.graph.project(
  'grapheMissionOT',
  '*',
["MIS_TRSPRT","MIS_ALLOW_MNTNC","MIS_DSTRBT_ELEC","Mis_PRVD_ELEC","MIS_ALLOW_MSR","MIS_CMNCT_DATA","MIS_MSR","MIS_RCV_ORDER","MIS_CHCK_ORDER","MIS_TRSMT_ORDER","MIS_EXECUTE_ORDER"]
);
```
###Projection IT<a name="1.2"></a>
```cypher
//Projections IT

// Création de labels temporaires pour les noeuds SH
MATCH (n)
WHERE n.couche IN ["software","hardware"]
SET n:NoeudsSH;

// Création de labels temporaires pour les noeuds S
MATCH (n)
WHERE n.couche IN ["software"]
SET n:NoeudsS;

// Création de labels temporaires pour les noeuds H
MATCH (n)
WHERE n.couche IN ["hardware"]
SET n:NoeudsH;

//Projection SH IT
CALL gds.graph.project(
  'grapheSHIT',
  "NoeudsSH",
  ["SOFT_REQUEST","SOFT_ANSWER_REQUEST"]
);

//Projection S IT
CALL gds.graph.project(
  'grapheSIT',
  "NoeudsS",
  ["SOFT_REQUEST","SOFT_ANSWER_REQUEST"]
);

//Projection H IT
CALL gds.graph.project(
  'grapheHIT',
  "NoeudsH",
  ["SOFT_REQUEST","SOFT_ANSWER_REQUEST"]
);

//Projection Cyber IT
CALL gds.graph.project(
  'grapheCyberIT',
  '*',
  ["CYBER_DATA_IT"]
);

//Projection Missions IT
CALL gds.graph.project(
  'grapheMissionIT',
  '*',
  ["MIS_TRNSMT_DATA","MIS_PRTCT_NTWRK","MIS_CNNCT_NTWRK","MIS_USE_RSSRC","MIS_HOST_SRVC"]
);
```

###Projection Service <a name="1.3"></a>

```cypher
//Projections PP

// Création de labels temporaires pour les noeuds Users
MATCH (n)
WHERE n.couche IN ["user"]
SET n:NoeudsUsers;

// Création de labels temporaires pour les noeuds Opérateurs
MATCH (n)
WHERE n.couche IN ["opérateur"]
SET n:NoeudsOperators;

// Création de labels temporaires pour les noeuds end
MATCH (n)
WHERE n.couche IN ["endUsers"]
SET n:NoeudsEndUsers;

// Création de labels temporaires pour tout le monde
MATCH (n)
WHERE n.couche IN ["endUsers","opérateur","users"]
SET n:NoeudsPP;

//Projection PP MISSION
CALL gds.graph.project(
  'graphePpMis',
  '*',
["MIS_PP_MONITOR_OT","MIS_PP_COMMAND_OT","MIS_PP_MAINTAIN_OT","MIS_PP_REPAIR_OT","MIS_PP_BUSINESS","MIS_PP_MONITOR_IT","MIS_PP_COMMAND_IT","MIS_PP_MAINTAIN_IT","MIS_PP_REPAIR_IT"]
);

//Projection User
CALL gds.graph.project(
  'graphePpUsers',
  'NoeudsUsers',
  ["HUM_MNPLT","HUM_EXECUTE_ORDER","HUM_USE_SRVC","HUM_INTERACT","HUM_ACCESS"]
);

//Projection Operators
CALL gds.graph.project(
  'graphePpOperators',
  'NoeudsOperators',
  ["HUM_MNPLT","HUM_EXECUTE_ORDER","HUM_USE_SRVC","HUM_INTERACT","HUM_ACCESS"]
);

//Projection EndUsers
CALL gds.graph.project(
  'graphePpEndUsers',
  'NoeudsEndUsers',
  ["HUM_MNPLT","HUM_EXECUTE_ORDER","HUM_USE_SRVC","HUM_INTERACT","HUM_ACCESS"]
);

//Projection PP Humain 
CALL gds.graph.project(
  'graphePpHumain',
  'NoeudsPP',
  ["HUM_MNPLT","HUM_EXECUTE_ORDER","HUM_USE_SRVC","HUM_INTERACT","HUM_ACCESS"]
);
```
##Eigenvector Centrality OT <a name="2"></a>

###Eigenvector Physical <a name="2.1"></a>

```cypher
//Eigen OT Physique
CALL gds.eigenvector.stream('graphePhysiqueOT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Sensor <a name="2.2"></a>

```cypher
//Eigen OT Sensor
CALL gds.eigenvector.stream('grapheSensorOT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Actuator <a name="2.3"></a>

```cypher
//Eigen OT Actuator
CALL gds.eigenvector.stream('grapheActuatorOT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Cyber OT <a name="2.4"></a>

```cypher
//Eigen OT Cyber
CALL gds.eigenvector.stream('grapheCyberOT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Mission OT <a name="2.5"></a>

```cypher
//Eigen OT Mission
CALL gds.eigenvector.stream('grapheMissionOT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC;
```

##Eigenvector Centrality IT <a name="3"></a>

###Eigenvector Hardware <a name="3.1"></a>

```cypher
//Eigen IT Hardware
CALL gds.eigenvector.stream('grapheHIT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Software <a name="3.2"></a>

```cypher
//Eigen IT Software
CALL gds.eigenvector.stream('grapheSIT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Cyber IT <a name="3.3"></a>

```cypher
//Eigen IT Cyber
CALL gds.eigenvector.stream('grapheCyberIT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC
```

###Eigenvector Mission IT <a name="3.4"></a>

```cypher
//Eigen IT Mission
CALL gds.eigenvector.stream('grapheMissionIT')
YIELD nodeId, score
RETURN labels(gds.util.asNode(nodeId)) AS labels, score
ORDER BY score DESC;
```



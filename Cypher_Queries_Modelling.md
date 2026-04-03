# Cypher Queries: Modelling of the RTE Use Case 

## Queries to build the components of the electrical station

```cypher
CREATE
//Ligne vers Bus
(Ligne_E:LigneEntrée{couche:'physique'}),
(Sectio_E:SectionneurLigneE{couche:'actuator'}),
(Sectio_BE:SectionneurBus{couche:'actuator'}),
(TransC_LB:TransformateurCourantLigneBus{couche:'actuator'}),
(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'}),
(Hub_IED_LB:EnsembleIedLigneBus{couche:'actuator'}),
(Disj_LB:DisjoncteurLigneBus{couche:'actuator'}),

//BusEntrée
(BusE:JeuxDeBarreE{couche:'physique'}),
(TransT_BusE:TransformateurTensionBusE{couche:'actuator'}),
(Hub_IED_BusE:EnsembleIedBusE{couche:'actuator'}),
(Hub_Capteurs_BusE:EnsembleDeCapteursBusE{couche:'sensor'}),

//BusSortie
(BusS:JeuxDeBarreS{couche:'physique'}),
(TransT_BusS:TransformateurTensionBusS{couche:'actuator'}),
(Hub_IED_BusS:EnsembleIedBusS{couche:'actuator'}),
(Hub_Capteurs_BusS:EnsembleDeCapteursBusS{couche:'sensor'}),

//Ligne Transformateur
(Sectio_TP:SectionneurTransformateur{couche:'actuator'}),
(Sectio_S:SectionneurLigneS{couche:'actuator'}),
(TransC_TP:TransformateurCourantTransformateur{couche:'actuator'}),
(Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'}),
(Hub_IED_TP:EnsembleIedTransformateur{couche:'actuator'}),
(Disj_TP:DisjoncteurTransformateur{couche:'actuator'}),
(TransP_BT:TransformateurPuissanceBT{couche:'physique'}),

// Ligne de consommation qui sort du BusS
(Ligne_S:LigneSortie{couche:'physique'}),
(EndUsers:EndUsers{couche:'endUsers'}),

// Composants annexes de la station
(LMS:LocalMonitoringSystem{couche:'software'}),
(WorkSt:WorkStation{couche:'hardware'}),
(RTU:RemoteTerminalUnit{couche:'hardware'}),
//Opérateurs
(Opers:TechnicienStation{couche:'opérateur'})
```

## Queries to link the components of the station
### Queries for the process
```cypher
//Entrée d'électricité vers Bus Entrée
MATCH(Ligne_E:LigneEntrée{couche:'physique'}),
(Sectio_E:SectionneurLigneE)
MERGE (Ligne_E)-[:PHYS_ELEC]->(Sectio_E)
MERGE (Ligne_E)-[:MIS_TRSPRT]->(Sectio_E);

//Sectionneur Entrée
MATCH (Sectio_E:SectionneurLigneE{couche:'actuator'}),
(TransC_LB:TransformateurCourantLigneBus{couche:'actuator'})
MERGE (Sectio_E)-[:PHYS_ELEC]->(TransC_LB)
MERGE (Sectio_E)-[:ACT_ISOLATE_LINE]->(TransC_LB)
MERGE (Sectio_E)-[:MIS_ALLOW_MNTNC]->(TransC_LB);

//Transformateur de courant Entrée
MATCH (TransC_LB:TransformateurCourantLigneBus{couche:'actuator'}),
(Disj_LB:DisjoncteurLigneBus{couche:'actuator'})
MERGE (TransC_LB)-[:PHYS_ELEC]->(Disj_LB);

//Disjoncteur Entrée
MATCH (Disj_LB:DisjoncteurLigneBus{couche:'actuator'}),
(Sectio_BE:SectionneurBus{couche:'actuator'})
MERGE (Disj_LB)-[:PHYS_ELEC]->(Sectio_BE)
MERGE (Disj_LB)-[:ACT_CUT_LINE]->(Sectio_BE)
MERGE (Disj_LB)-[:MIS_PRTCT]->(Sectio_BE);

//Sectionneurs Entrée
MATCH(Sectio_BE:SectionneurBus{couche:'actuator'}),(BusE:JeuxDeBarreE{couche:'physique'})
MERGE (Sectio_BE)-[:PHYS_ELEC]->(BusE)
MERGE (Sectio_BE)-[:ACT_ISOLATE_LINE]->(BusE)
MERGE (Sectio_BE)-[:MIS_ALLOW_MNTNC]->(BusE);

//////////////////////////////////////////// ON EST ARRIVE AU BUS Entrée => Prise de tension ////////////////////////////////////////////

// BUS et TfT
MATCH (BusE:JeuxDeBarreE{couche:'physique'}),(TransT_BusE:TransformateurTensionBusE{couche:'actuator'})
MERGE (BusE)-[:MIS_DSTRBT_ELEC]->(TransT_BusE)
MERGE (BusE)-[:PHYS_ELEC]->(TransT_BusE);

// TfT vers Sectionneur TfP
MATCH (TransT_BusE:TransformateurTensionBusE{couche:'actuator'}),(Sectio_TP:SectionneurTransformateur{couche:'actuator'})
MERGE (TransT_BusE)-[:PHYS_ELEC]->(Sectio_TP);


// Sectionneur TfP vers TfC TfP
MATCH (Sectio_TP:SectionneurTransformateur{couche:'actuator'}),
(TransC_TP:TransformateurCourantTransformateur{couche:'actuator'})
MERGE (Sectio_TP)-[:PHYS_ELEC]->(TransC_TP)
MERGE (Sectio_TP)-[:ACT_ISOLATE_LINE]->(TransC_TP)
MERGE (Sectio_TP)-[:MIS_ALLOW_MNTNC]->(TransC_TP);

// TfC TfP vers Disjoncteur TfP
MATCH (TransC_TP:TransformateurCourantTransformateur{couche:'actuator'}),(Disj_TP:DisjoncteurTransformateur{couche:'actuator'})
MERGE (TransC_TP)-[:PHYS_ELEC]->(Disj_TP);

//Disjoncteur TfP vers TfP
MATCH(Disj_TP:DisjoncteurTransformateur{couche:'actuator'}),(TransP_BT:TransformateurPuissanceBT{couche:'physique'})
MERGE (Disj_TP)-[:PHYS_ELEC]->(TransP_BT)
MERGE (Disj_TP)-[:ACT_CUT_LINE]->(TransP_BT)
MERGE (Disj_TP)-[:MIS_PRTCT]->(TransP_BT);

// TfP vers Sectionneur BusS
MATCH (TransP_BT:TransformateurPuissanceBT{couche:'physique'}),
(Sectio_S:SectionneurLigneS{couche:'actuator'})
MERGE (TransP_BT)-[:PHYS_ELEC]->(Sectio_S)
MERGE (TransP_BT) -[:ACT_MDF_PWR]->(Sectio_S)
MERGE (TransP_BT) -[:ACT_ADAPT_PWR]->(Sectio_S);

// Sectionneur BusS vers BusS
MATCH (Sectio_S:SectionneurLigneS{couche:'actuator'}),(BusS:JeuxDeBarreS{couche:'physique'})
MERGE (Sectio_S)-[:PHYS_ELEC]->(BusS)
MERGE (Sectio_S)-[:ACT_ISOLATE_LINE]->(BusS)
MERGE (Sectio_S)-[:MIS_ALLOW_MNTNC]->(BusS);

// BusS vers TfT
MATCH (BusS:JeuxDeBarreS{couche:'physique'}),
(TransT_BusS:TransformateurTensionBusS{couche:'actuator'})
MERGE (BusS)-[:MIS_DSTRBT_ELEC]->(TransT_BusS)
MERGE (BusS)-[:PHYS_ELEC]->(TransT_BusS);

// TfT vers Ligne conso
MATCH (TransT_BusS:TransformateurTensionBusS{couche:'actuator'}),(Ligne_S:LigneSortie{couche:'physique'})
MERGE (TransT_BusS)-[:PHYS_ELEC]->(Ligne_S);

// Ligne sortie vers EndUser
MATCH (Ligne_S:LigneSortie{couche:'physique'}),(EndUsers:EndUsers{couche:'endUsers'})
MERGE (Ligne_S)-[:PHYS_ELEC]->(EndUsers)
MERGE (Ligne_S)-[:Mis_PRVD_ELEC]->(EndUsers);
// C'est la fin du process électrique
```

### Queries for the process measurements

```cypher
//Entrée d'électricité vers Bus Entrée

//Sectionneurs Entrée
MATCH (Sectio_E:SectionneurLigneE{couche:'actuator'}),
(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'})
MERGE (Sectio_E)-[:PHYS_ATTACH]->(Hub_Capteurs_LB);

//Sectionneur Bus
MATCH(Sectio_BE:SectionneurBus{couche:'actuator'}),
(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'})
MERGE (Sectio_BE)-[:PHYS_ATTACH]->(Hub_Capteurs_LB);

//TfC Entrée
MATCH (TransC_LB:TransformateurCourantLigneBus{couche:'actuator'}),(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'})
MERGE (TransC_LB)-[:MIS_ALLOW_MSR]->(Hub_Capteurs_LB)
MERGE (TransC_LB)-[:ACT_ADAPT_CRRT]->(Hub_Capteurs_LB)
MERGE (TransC_LB)-[:PHYS_ELEC]->(Hub_Capteurs_LB);

//Disjoncteur Entrée
MATCH (Disj_LB:DisjoncteurLigneBus{couche:'actuator'}),
(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'})
MERGE (Disj_LB)-[:PHYS_ATTACH]->(Hub_Capteurs_LB);

//Capteurs Entrée
MATCH (Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'}),(Hub_IED_LB:EnsembleIedLigneBus{couche:'actuator'})
MERGE (Hub_Capteurs_LB)-[:SENS_MECA]->(Hub_IED_LB)
MERGE (Hub_Capteurs_LB)-[:SENS_STATE]->(Hub_IED_LB)
MERGE (Hub_Capteurs_LB)-[:SENS_ELEC]->(Hub_IED_LB)
MERGE (Hub_Capteurs_LB)-[:CYBER_MECA_OT]->(Hub_IED_LB)
MERGE (Hub_Capteurs_LB)-[:CYBER_STATE_OT]->(Hub_IED_LB)
MERGE (Hub_Capteurs_LB)-[:CYBER_ELEC_OT]->(Hub_IED_LB)
MERGE (Hub_Capteurs_LB)-[:MIS_MSR]->(Hub_IED_LB);

//IEDs Entrée
MATCH (Hub_IED_LB:EnsembleIedLigneBus{couche:'actuator'}), (RTU:RemoteTerminalUnit{couche:'hardware'})
MERGE (Hub_IED_LB)-[:CYBER_DATA_OT]->(RTU)
MERGE (Hub_IED_LB)-[:MIS_CMNCT_DATA]->(RTU);

////////////////////////////////////////////Mesure sur le BUS Entrée////////////////////////////////////////////

//TfT BUS Entrée
MATCH (TransT_BusE:TransformateurTensionBusE{couche:'actuator'}),(Hub_Capteurs_BusE:EnsembleDeCapteursBusE{couche:'sensor'})
MERGE (TransT_BusE)-[:PHYS_ELEC]->(Hub_Capteurs_BusE)
MERGE (TransT_BusE)-[:ACT_ADAPT_VLTG]->(Hub_Capteurs_BusE)
MERGE (TransT_BusE)-[:MIS_ALLOW_MSR]->(Hub_Capteurs_BusE);

// Capteurs vers IED Bus Entrée
MATCH (Hub_Capteurs_BusE:EnsembleDeCapteursBusE{couche:'sensor'}),(Hub_IED_BusE:EnsembleIedBusE{couche:'actuator'})
MERGE (Hub_Capteurs_BusE)-[:SENS_MECA]->(Hub_IED_BusE)
MERGE (Hub_Capteurs_BusE)-[:SENS_ELEC]->(Hub_IED_BusE)
MERGE (Hub_Capteurs_BusE)-[:CYBER_MECA_OT]->(Hub_IED_BusE)
MERGE (Hub_Capteurs_BusE)-[:CYBER_ELEC_OT]->(Hub_IED_BusE)
MERGE (Hub_Capteurs_BusE)-[:MIS_MSR]->(Hub_IED_BusE);

//IED BUS Entrée
MATCH (Hub_IED_BusE:EnsembleIedBusE{couche:'actuator'}),(RTU:RemoteTerminalUnit{couche:'hardware'})
MERGE (Hub_IED_BusE)-[:CYBER_DATA_OT]->(RTU)
MERGE (Hub_IED_BusE)-[:MIS_CMNCT_DATA]->(RTU);

////////////////////////////////////////////Ligne du Transformateur et suite////////////////////////////////////////////
//Sectionneurs TfP
MATCH(Sectio_TP:SectionneurTransformateur{couche:'actuator'}),
(Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'})
MERGE (Sectio_TP)-[:PHYS_ATTACH]->(Hub_Capteurs_TP);

//
MATCH(Sectio_S:SectionneurLigneS{couche:'actuator'}),
(Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'})
MERGE (Sectio_S)-[:PHYS_ATTACH]->(Hub_Capteurs_TP);

//TfC TfP
MATCH (TransC_TP:TransformateurCourantTransformateur{couche:'actuator'}),(Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'})
MERGE (TransC_TP)-[:MIS_ALLOW_MSR]->(Hub_Capteurs_TP)
MERGE (TransC_TP)-[:ACT_ADAPT_CRRT]->(Hub_Capteurs_TP)
MERGE (TransC_TP)-[:PHYS_ELEC]->(Hub_Capteurs_TP);

//Disjoncteur TfP
MATCH (Disj_TP:DisjoncteurTransformateur{couche:'actuator'}),
(Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'})
MERGE (Disj_TP)-[:PHYS_ATTACH]->(Hub_Capteurs_TP);

//Capteurs TfP
MATCH (Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'}),(Hub_IED_TP:EnsembleIedTransformateur{couche:'actuator'})
MERGE (Hub_Capteurs_TP)-[:SENS_MECA]->(Hub_IED_TP)
MERGE (Hub_Capteurs_TP)-[:SENS_STATE]->(Hub_IED_TP)
MERGE (Hub_Capteurs_TP)-[:SENS_ELEC]->(Hub_IED_TP)
MERGE (Hub_Capteurs_TP)-[:CYBER_MECA_OT]->(Hub_IED_TP)
MERGE (Hub_Capteurs_TP)-[:CYBER_STATE_OT]->(Hub_IED_TP)
MERGE (Hub_Capteurs_TP)-[:CYBER_ELEC_OT]->(Hub_IED_TP)
MERGE (Hub_Capteurs_TP)-[:MIS_MSR]->(Hub_IED_TP);

//IEDs TfP
MATCH (Hub_IED_TP:EnsembleIedTransformateur{couche:'actuator'}), (RTU:RemoteTerminalUnit{couche:'hardware'})
MERGE (Hub_IED_TP)-[:CYBER_DATA_OT]->(RTU)
MERGE (Hub_IED_TP)-[:MIS_CMNCT_DATA]->(RTU);

////////////////////////////////////////////Mesure sur le BUS Entrée////////////////////////////////////////////

//TfT BUS Sortie
MATCH (TransT_BusS:TransformateurTensionBusS{couche:'actuator'}),
 (Hub_Capteurs_BusS:EnsembleDeCapteursBusS{couche:'sensor'})
MERGE (TransT_BusS)-[:MIS_ALLOW_MSR]->(Hub_Capteurs_BusS)
MERGE (TransT_BusS)-[:ACT_ADAPT_VLTG]->(Hub_Capteurs_BusS)
MERGE (TransT_BusS)-[:PHYS_ELEC]->(Hub_Capteurs_BusS);

//Capteurs BUS Sortie
MATCH (Hub_Capteurs_BusS:EnsembleDeCapteursBusS{couche:'sensor'}),(Hub_IED_BusS:EnsembleIedBusS{couche:'actuator'})
MERGE (Hub_Capteurs_BusS)-[:SENS_MECA]->(Hub_IED_BusS)
MERGE (Hub_Capteurs_BusS)-[:SENS_ELEC]->(Hub_IED_BusS)
MERGE (Hub_Capteurs_BusS)-[:CYBER_MECA_OT]->(Hub_IED_BusS)
MERGE (Hub_Capteurs_BusS)-[:CYBER_ELEC_OT]->(Hub_IED_BusS)
MERGE (Hub_Capteurs_BusS)-[:MIS_MSR]->(Hub_IED_BusS);

//IEDs BUS Sortie
MATCH (Hub_IED_BusS:EnsembleIedBusS{couche:'actuator'}),(RTU:RemoteTerminalUnit{couche:'hardware'})
MERGE (Hub_IED_BusS)-[:CYBER_DATA_OT]->(RTU)
MERGE (Hub_IED_BusS)-[:MIS_CMNCT_DATA]->(RTU);

//////////////////////////////////////////// Partie globale de monitoring//////////////////////////////////////////// 

MATCH (RTU:RemoteTerminalUnit{couche:'hardware'}),(LMS:LocalMonitoringSystem{couche:'software'})
MERGE (RTU)-[:CYBER_DATA_OT]->(LMS)
MERGE (RTU)-[:MIS_TRNSMT_DATA]->(LMS);

MATCH (LMS:LocalMonitoringSystem{couche:'software'}),
(WorkSt:WorkStation{couche:'hardware'})
MERGE (LMS)-[:CYBER_DATA_OT]->(WorkSt)
MERGE (LMS)-[:MIS_ANALYZE_DATA]->(WorkSt);

```





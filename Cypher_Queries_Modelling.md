# Cypher Queries: Modelling of the RTE Use Case 

# Table of contents
1. [Queries to build the components of the electrical station](#1)
2. [Queries to link the components of the station](#2)
    1. [Queries for the process](#2.1)
    2. [Queries for the process measurements](#2.2)
3. [Queries to the IT infrastructure's components](#3)

## Queries to build the components of the electrical station <a name="1"></a>

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

## Queries to link the components of the station <a name="2"></a>
### Queries for the process <a name="2.1"></a>
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

### Queries for the process measurements <a name="2.2"></a>

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
## Queries to the IT infrastructure's components <a name="3"></a>

```cypher
CREATE 
//Firewalls
(FireWSt:FirewallStation{couche:'software'}),
(FireWOTST:FirewallDCOTSTA{couche:'software'}),
(FireWITOT:FirewallDCITOT{couche:'software'}),
(FireWIT:FirewallIT{couche:'software'}),
(FireWDCIT:FirewallDCIT{couche:'software'}),
(FireWSOC:FirewallSOC{couche:'software'}),

//Routers
(RoutE:RouteurOTEast{couche:'hardware'}),
(RoutW:RouteurOTWest{couche:'hardware'}),
(RoutN:RouteurOTNorth{couche:'hardware'}),
(RoutS:RouteurOTSud{couche:'hardware'}),

//WAN
(WanRte:WANRte{couche:'software'}),

//Datacenter_OT_NETWORK
(OTNet:NetworkOT{couche:'cyber'}),
(Scad:Scada{couche:'software'}),
(ServScada:ServersScadaDataCOT{couche:'hardware'}),
(Dms:DistriMngmtSyst{couche:'software'}),
(ServDms:ServersDmsDataCOT{couche:'hardware'}),
(DnsOT:DomainNameSystOT{couche:'software'}),
(ServDns:ServersDnsDataCOT{couche:'hardware'}),
(LogsOT:StockageLogsOT{couche:'software'}),
(ServLogsOT:ServerslogsDataCOT{couche:'hardware'}),
(TAD:TerminalAccesDistant{couche:'software'}),
(ServTad:ServersTadDataCCOT{couche:'hardware'}),

//SOC
(SOCNet:NetworkSOC{couche:'cyber'}),
(Siem:Siem{couche:'software'}),
(ServSiem:ServersSiemSoc{couche:'hardware'}),
(LogsSoc:StockageLogsSoc{couche:'software'}),
(ServLogsSoc:ServersLogsSoc{couche:'hardware'}),
(WrkStSoc:WorkStationSoc{couche:'hardware'}),

//Datacenter_IT_NETWORK
(DCITNet:NetworkDCIT{couche:'cyber'}),
(Mail:MailIT{couche:'software'}),
(ServsMailDCIT:ServersMailDataCIT{couche:'hardware'}),
(DbIT:DataBaseIT{couche:'software'}),
(ServsDbDCIT:ServersDatabaseDataCIT{couche:'hardware'}),
(AdIT:ActiveDirectoryIT{couche:'software'}),
(ServsAdDCIT:ServersActiveDirectoryDataCIT{couche:'hardware'}),
(LogsIT:StockageLogsIT{couche:'software'}),
(ServsLogsDCIT:ServersLogsDataCIT{couche:'hardware'}),

//Organisational_NETWORK
(IntITNet:InternalNetworkIT{couche:'cyber'}),
(OpeITNet:OperatorNetworkIT{couche:'cyber'}),
(SuppITNet:SupportNetworkIT{couche:'cyber'}),
(WrkStOpScada:WorkStationOperatorScada{couche:'hardware'}),
(WrkStInternal:WorkStationInternal{couche:'hardware'}),
(WrkStIT:WorkStationIT{couche:'hardware'}),

//Users
(Ituser:UserOfITServices{couche:'user'}),
(ItGuy:ItGuy{couche:'user'}),
(SocOper:SocOperator{couche:'user'}),
(ScadOper:ScadaOperator{couche:'user'});
```

## Queries to link the components of the IT infrastructure
### Liens IT UP

```cypher
//Liens cyber partants de la station et remontant dans le réseau

/////////////////////////////////////Liens entre RTU, FW et routeurs/////////////////////////////////////
MATCH(RTU:RemoteTerminalUnit{couche:'hardware'}),(FireWSt:FirewallStation{couche:'software'})
MERGE (RTU)-[:CYBER_DATA_OT]->(FireWSt)
MERGE (RTU)-[:MIS_TRNSMT_DATA]->(FireWSt);

MATCH(FireWSt:FirewallStation{couche:'software'}),(RoutS:RouteurOTSud{couche:'hardware'})
MERGE (FireWSt)-[:CYBER_DATA_OT]->(RoutS)
MERGE (FireWSt)-[:MIS_PRTCT_NTWRK]->(RoutS);

MATCH(RoutS:RouteurOTSud{couche:'hardware'}),(RoutE:RouteurOTEast{couche:'hardware'})
MERGE (RoutS)-[:CYBER_DATA_OT]->(RoutE)
MERGE (RoutS)-[:MIS_CNNCT_NTWRK]->(RoutE);

MATCH(RoutS:RouteurOTSud{couche:'hardware'}),(RoutW:RouteurOTWest{couche:'hardware'})
MERGE (RoutS)-[:CYBER_DATA_OT]->(RoutW)
MERGE (RoutS)-[:MIS_CNNCT_NTWRK]->(RoutW);

MATCH(RoutE:RouteurOTEast{couche:'hardware'}),(RoutN:RouteurOTNorth{couche:'hardware'})
MERGE (RoutE)-[:CYBER_DATA_OT]->(RoutN)
MERGE (RoutE)-[:MIS_CNNCT_NTWRK]->(RoutN);

MATCH(RoutW:RouteurOTWest{couche:'hardware'}),(RoutN:RouteurOTNorth{couche:'hardware'})
MERGE (RoutW)-[:CYBER_DATA_OT]->(RoutN)
MERGE (RoutW)-[:MIS_CNNCT_NTWRK]->(RoutN);

MATCH(RoutN:RouteurOTNorth{couche:'hardware'}),(FireWOTST:FirewallDCOTSTA{couche:'software'})
MERGE (RoutN)-[:CYBER_DATA_OT]->(FireWOTST)
MERGE (RoutN)-[:MIS_CNNCT_NTWRK]->(FireWOTST);

/////////////////////////////////////Liens vers DCOT/////////////////////////////////////

MATCH(FireWOTST:FirewallDCOTSTA{couche:'software'}),(OTNetwork:NetworkOT{couche:'cyber'})
MERGE(FireWOTST)-[:CYBER_DATA_OT]->(OTNetwork)
MERGE(FireWOTST)-[:MIS_PRTCT_NTWRK]->(OTNetwork);

//SCADA
MATCH(OTNetwork:NetworkOT{couche:'cyber'}),(ServScada:ServersScadaDataCOT{couche:'hardware'})
MERGE (OTNetwork)-[:SOFT_REQUEST]->(ServScada)
MERGE (OTNetwork)-[:CYBER_DATA_OT]->(ServScada)
MERGE (OTNetwork)-[:MIS_TRNSMT_DATA]->(ServScada);

MATCH(ServScada:ServersScadaDataCOT{couche:'hardware'}),(Scad:Scada{couche:'software'})
MERGE(ServScada)-[:CYBER_DATA_OT]->(Scad)
MERGE(ServScada)-[:SOFT_REQUEST]->(Scad)
MERGE(ServScada)-[:MIS_USE_RSSRC]->(Scad);

//DMS
MATCH(OTNetwork:NetworkOT{couche:'cyber'}),(ServDms:ServersDmsDataCOT{couche:'hardware'})
MERGE(OTNetwork)-[:CYBER_DATA_OT]->(ServDms)
MERGE(OTNetwork)-[:SOFT_REQUEST]->(ServDms)
MERGE(OTNetwork)-[:MIS_TRNSMT_DATA]->(ServDms);

MATCH(ServDms:ServersDmsDataCOT{couche:'hardware'}),(Dms:DistriMngmtSyst{couche:'software'})
MERGE(ServDms)-[:CYBER_DATA_OT]->(Dms)
MERGE(ServDms)-[:SOFT_REQUEST]->(Dms)
MERGE(ServDms)-[:MIS_USE_RSSRC]->(Dms);

//DNS
MATCH(OTNetwork:NetworkOT{couche:'cyber'}),(ServDns:ServersDnsDataCOT{couche:'hardware'})
MERGE(OTNetwork)-[:CYBER_DATA_OT]->(ServDns)
MERGE(OTNetwork)-[:SOFT_REQUEST]->(ServDns)
MERGE(OTNetwork)-[:MIS_TRNSMT_DATA]->(ServDns);

MATCH(ServDns:ServersDnsDataCOT{couche:'hardware'}),(DnsOT:DomainNameSystOT{couche:'software'})
MERGE (ServDns)-[:CYBER_DATA_OT]->(DnsOT)
MERGE (ServDns)-[:SOFT_REQUEST]->(DnsOT)
MERGE (ServDns)-[:MIS_USE_RSSRC]->(DnsOT);

//LogsOT
MATCH(OTNet:NetworkOT{couche:'cyber'}),(ServLogsOT:ServerslogsDataCOT{couche:'hardware'})
MERGE(OTNet)-[:CYBER_DATA_OT]->(ServLogsOT)
MERGE(OTNet)-[:SOFT_REQUEST]->(ServLogsOT)
MERGE(OTNet)-[:MIS_TRNSMT_DATA]->(ServLogsOT);

MATCH(ServLogsOT:ServerslogsDataCOT{couche:'hardware'}),(LogsOT:StockageLogsOT{couche:'software'})
MERGE (ServLogsOT)-[:CYBER_DATA_OT]->(LogsOT)
MERGE (ServLogsOT)-[:SOFT_REQUEST]->(LogsOT)
MERGE (ServLogsOT)-[:MIS_USE_RSSRC]->(LogsOT);

//Tad
MATCH(OTNetwork:NetworkOT{couche:'cyber'}),(ServTad:ServersTadDataCCOT{couche:'hardware'})
MERGE(OTNetwork)-[:CYBER_DATA_OT]->(ServTad)
MERGE(OTNetwork)-[:SOFT_REQUEST]->(ServTad)
MERGE(OTNetwork)-[:MIS_TRNSMT_DATA]->(ServTad);

MATCH(ServTad:ServersTadDataCCOT{couche:'hardware'}),(TAD:TerminalAccesDistant{couche:'software'})
MERGE(ServTad)-[:CYBER_DATA_OT]->(TAD)
MERGE(ServTad)-[:SOFT_REQUEST]->(TAD)
MERGE(ServTad)-[:MIS_USE_RSSRC]->(TAD);

//Vers WAN
MATCH(OTNetwork:NetworkOT{couche:'cyber'}),(FireWITOT:FirewallDCITOT{couche:'software'})
MERGE(OTNetwork)-[:CYBER_DATA_OT]->(FireWITOT)
MERGE(OTNetwork)-[:SOFT_ANSWER_REQUEST]->(FireWITOT)
MERGE(OTNetwork)-[:MIS_TRNSMT_DATA]->(FireWITOT);

MATCH (FireWITOT:FirewallDCITOT{couche:'software'}),(WanRte:WANRte{couche:'software'})
MERGE(FireWITOT)-[:CYBER_DATA_OT]->(WanRte)
MERGE(FireWITOT)-[:SOFT_ANSWER_REQUEST]->(WanRte)
MERGE(FireWITOT)-[:MIS_PRTCT_NTWRK]->(WanRte);


/////////////////////////////////////On passe au Wan et Network SOC/////////////////////////////////////
MATCH(WanRte:WANRte{couche:'software'}),(FireWSOC:FirewallSOC{couche:'software'})
MERGE(WanRte)-[:CYBER_DATA_IT]->(FireWSOC)
MERGE(WanRte)-[:CYBER_DATA_OT]->(FireWSOC)
MERGE(WanRte)-[:MIS_TRNSMT_DATA]->(FireWSOC);

MATCH(FireWSOC:FirewallSOC{couche:'software'}),(SOCNet:NetworkSOC{couche:'cyber'})
MERGE(FireWSOC)-[:CYBER_DATA_IT]->(SOCNet)
MERGE(FireWSOC)-[:CYBER_DATA_OT]->(SOCNet)
MERGE(FireWSOC)-[:MIS_PRTCT_NTWRK]->(SOCNet);

//Siem
MATCH (SOCNet:NetworkSOC{couche:'cyber'}),(ServSiem:ServersSiemSoc{couche:'hardware'})
MERGE(SOCNet)-[:SOFT_REQUEST]->(ServSiem)
MERGE(SOCNet)-[:CYBER_DATA_IT]->(ServSiem)
MERGE(SOCNet)-[:CYBER_DATA_OT]->(ServSiem)
MERGE(SOCNet)-[:MIS_TRNSMT_DATA]->(ServSiem);

MATCH(ServSiem:ServersSiemSoc{couche:'hardware'}),(Siem:Siem{couche:'software'})
MERGE(ServSiem)-[:SOFT_REQUEST]->(Siem)
MERGE(ServSiem)-[:CYBER_DATA_IT]->(Siem)
MERGE(ServSiem)-[:CYBER_DATA_OT]->(Siem)
MERGE(ServSiem)-[:MIS_USE_RSSRC]->(Siem);

//Logs
MATCH (SOCNet:NetworkSOC{couche:'cyber'}),(ServLogsSoc:ServersLogsSoc{couche:'hardware'})
MERGE(SOCNet)-[:SOFT_REQUEST]->(ServLogsSoc)
MERGE(SOCNet)-[:CYBER_DATA_IT]->(ServLogsSoc)
MERGE(SOCNet)-[:CYBER_DATA_OT]->(ServLogsSoc)
MERGE(SOCNet)-[:MIS_TRNSMT_DATA]->(ServLogsSoc);

MATCH(ServLogsSoc:ServersLogsSoc{couche:'hardware'}),(LogsSoc:StockageLogsSoc{couche:'software'})
MERGE(ServLogsSoc)-[:SOFT_REQUEST]->(LogsSoc)
MERGE(ServLogsSoc)-[:CYBER_DATA_IT]->(LogsSoc)
MERGE(ServLogsSoc)-[:CYBER_DATA_OT]->(LogsSoc)
MERGE(ServLogsSoc)-[:MIS_USE_RSSRC]->(LogsSoc);

MATCH (SOCNet:NetworkSOC{couche:'cyber'}),(WrkStSoc:WorkStationSoc{couche:'hardware'})
MERGE(SOCNet)-[:SOFT_ANSWER_REQUEST]->(WrkStSoc)
MERGE(SOCNet)-[:CYBER_DATA_IT]->(WrkStSoc)
MERGE(SOCNet)-[:CYBER_DATA_OT]->(WrkStSoc)
MERGE(SOCNet)-[:MIS_TRNSMT_DATA]->(WrkStSoc);

//////////////////////////////////////Liens WAN IT NETWORK/////////////////////////////////////
MATCH(WanRte:WANRte{couche:'software'}),(FireWDCIT:FirewallDCIT{couche:'software'})
MERGE(WanRte)-[:SOFT_REQUEST]->(FireWDCIT)
MERGE(WanRte)-[:CYBER_DATA_IT]->(FireWDCIT)
MERGE(WanRte)-[:MIS_TRNSMT_DATA]->(FireWDCIT);

MATCH(FireWDCIT:FirewallDCIT{couche:'software'}),(DCITNet:NetworkDCIT{couche:'cyber'})
MERGE(FireWDCIT)-[:SOFT_REQUEST]->(DCITNet)
MERGE(FireWDCIT)-[:CYBER_DATA_IT]->(DCITNet)
MERGE(FireWDCIT)-[:MIS_PRTCT_NTWRK]->(DCITNet);

//MAILS
MATCH(DCITNet:NetworkDCIT{couche:'cyber'}),(ServsMailDCIT:ServersMailDataCIT{couche:'hardware'})
MERGE(DCITNet)-[:SOFT_REQUEST]->(ServsMailDCIT)
MERGE(DCITNet)-[:CYBER_DATA_IT]->(ServsMailDCIT)
MERGE(DCITNet)-[:MIS_TRNSMT_DATA]->(ServsMailDCIT);

MATCH(ServsMailDCIT:ServersMailDataCIT{couche:'hardware'}),(Mail:MailIT{couche:'software'})
MERGE(ServsMailDCIT)-[:SOFT_REQUEST]->(Mail)
MERGE(ServsMailDCIT)-[:CYBER_DATA_IT]->(Mail)
MERGE(ServsMailDCIT)-[:MIS_USE_RSSRC]->(Mail);

//DATABASE
MATCH (DCITNet:NetworkDCIT{couche:'cyber'}),(ServsDbDCIT:ServersDatabaseDataCIT{couche:'hardware'})
MERGE(DCITNet)-[:CYBER_DATA_IT]->(ServsDbDCIT)
MERGE(DCITNet)-[:SOFT_REQUEST]->(ServsDbDCIT)
MERGE(DCITNet)-[:MIS_TRNSMT_DATA]->(ServsDbDCIT);

MATCH(ServsDbDCIT:ServersDatabaseDataCIT{couche:'hardware'}),(DbIT:DataBaseIT{couche:'software'})
MERGE(ServsDbDCIT)-[:SOFT_REQUEST]->(DbIT)
MERGE(ServsDbDCIT)-[:CYBER_DATA_IT]->(DbIT)
MERGE(ServsDbDCIT)-[:MIS_USE_RSSRC]->(DbIT);

//ACTIVE DIRECTORY
MATCH (DCITNet:NetworkDCIT{couche:'cyber'}),(ServsAdDCIT:ServersActiveDirectoryDataCIT{couche:'hardware'})
MERGE(DCITNet)-[:SOFT_REQUEST]->(ServsAdDCIT)
MERGE(DCITNet)-[:CYBER_DATA_IT]->(ServsAdDCIT)
MERGE(DCITNet)-[:MIS_TRNSMT_DATA]->(ServsAdDCIT);

MATCH(ServsAdDCIT:ServersActiveDirectoryDataCIT{couche:'hardware'}),(AdIT:ActiveDirectoryIT{couche:'software'})
MERGE(ServsAdDCIT)-[:SOFT_REQUEST]->(AdIT)
MERGE(ServsAdDCIT)-[:CYBER_DATA_IT]->(AdIT)
MERGE(ServsAdDCIT)-[:MIS_USE_RSSRC]->(AdIT);

//LOGS
MATCH (DCITNet:NetworkDCIT{couche:'cyber'}),(ServsLogsDCIT:ServersLogsDataCIT{couche:'hardware'})
MERGE(DCITNet)-[:SOFT_REQUEST]->(ServsLogsDCIT)
MERGE(DCITNet)-[:CYBER_DATA_IT]->(ServsLogsDCIT)
MERGE(DCITNet)-[:MIS_TRNSMT_DATA]->(ServsLogsDCIT);

MATCH(ServsLogsDCIT:ServersLogsDataCIT{couche:'hardware'}),(LogsIT:StockageLogsIT{couche:'software'})
MERGE(ServsLogsDCIT)-[:SOFT_REQUEST]->(LogsIT)
MERGE(ServsLogsDCIT)-[:CYBER_DATA_IT]->(LogsIT)
MERGE(ServsLogsDCIT)-[:MIS_TRNSMT_DATA]->(LogsIT);

///////////////////////////////////// WAN IT NETWORK internes /////////////////////////////////////

MATCH(WanRte:WANRte{couche:'software'}),(FireWIT:FirewallIT{couche:'software'})
MERGE(WanRte)-[:SOFT_ANSWER_REQUEST]->(FireWIT)
MERGE(WanRte)-[:CYBER_DATA_IT]->(FireWIT)
MERGE(WanRte)-[:CYBER_DATA_OT]->(FireWIT)
MERGE(WanRte)-[:MIS_TRNSMT_DATA]->(FireWIT);

//
MATCH(FireWIT:FirewallIT{couche:'software'}),(IntITNet:InternalNetworkIT{couche:'cyber'})
MERGE(FireWIT)-[:SOFT_ANSWER_REQUEST]->(IntITNet)
MERGE(FireWIT)-[:CYBER_DATA_IT]->(IntITNet)
MERGE(FireWIT)-[:MIS_PRTCT_NTWRK]->(IntITNet);

MATCH(IntITNet:InternalNetworkIT{couche:'cyber'}),(WrkStInternal:WorkStationInternal{couche:'hardware'})
MERGE(IntITNet)-[:SOFT_ANSWER_REQUEST]->(WrkStInternal)
MERGE(IntITNet)-[:CYBER_DATA_IT]->(WrkStInternal)
MERGE(IntITNet)-[:MIS_TRNSMT_DATA]->(WrkStInternal);

//
MATCH(FireWIT:FirewallIT{couche:'software'}),(OpeITNet:OperatorNetworkIT{couche:'cyber'})
MERGE(FireWIT)-[:SOFT_ANSWER_REQUEST]->(OpeITNet)
MERGE(FireWIT)-[:CYBER_DATA_OT]->(OpeITNet)
MERGE(FireWIT)-[:CYBER_DATA_IT]->(OpeITNet)
MERGE(FireWIT)-[:MIS_PRTCT_NTWRK]->(OpeITNet);

MATCH(OpeITNet:OperatorNetworkIT{couche:'cyber'}),(WrkStOpScada:WorkStationOperatorScada{couche:'hardware'})
MERGE(OpeITNet)-[:SOFT_ANSWER_REQUEST]->(WrkStOpScada)
MERGE(OpeITNet)-[:CYBER_DATA_OT]->(WrkStOpScada)
MERGE(OpeITNet)-[:CYBER_DATA_IT]->(WrkStOpScada)
MERGE(OpeITNet)-[:MIS_TRNSMT_DATA]->(WrkStOpScada);

//
MATCH(FireWIT:FirewallIT{couche:'software'}),(SuppITNet:SupportNetworkIT{couche:'cyber'})
MERGE(FireWIT)-[:SOFT_ANSWER_REQUEST]->(SuppITNet)
MERGE(FireWIT)-[:CYBER_DATA_IT]->(SuppITNet)
MERGE(FireWIT)-[:MIS_PRTCT_NTWRK]->(SuppITNet);

MATCH(SuppITNet:SupportNetworkIT{couche:'cyber'}),(WrkStIT:WorkStationIT{couche:'hardware'})
MERGE(SuppITNet)-[:SOFT_ANSWER_REQUEST]->(WrkStIT)
MERGE(SuppITNet)-[:CYBER_DATA_IT]->(WrkStIT)
MERGE(SuppITNet)-[:MIS_TRNSMT_DATA]->(WrkStIT);
```

###Liens IT DOWN

```cypher
// Liens partant des stations et softwares redescendants dans vers le reséau et la partie OT
///////////////////////////////////// WAN IT NETWORK internes /////////////////////////////////////

//
MATCH(WrkStIT:WorkStationIT{couche:'hardware'}),(SuppITNet:SupportNetworkIT{couche:'cyber'})
MERGE(WrkStIT)-[:SOFT_REQUEST]->(SuppITNet)
MERGE(WrkStIT)-[:CYBER_DATA_IT]->(SuppITNet)
MERGE(WrkStIT)-[:MIS_TRNSMT_DATA]->(SuppITNet);
//
MATCH(SuppITNet:SupportNetworkIT{couche:'cyber'}),(FireWIT:FirewallIT{couche:'software'})
MERGE(SuppITNet)-[:SOFT_REQUEST]->(FireWIT)
MERGE(SuppITNet)-[:CYBER_DATA_IT]->(FireWIT)
MERGE(SuppITNet)-[:MIS_TRNSMT_DATA]->(FireWIT);
//
MATCH(FireWIT:FirewallIT{couche:'software'}),(WanRte:WANRte{couche:'software'})
MERGE(FireWIT)-[:SOFT_REQUEST]->(WanRte)
MERGE(FireWIT)-[:CYBER_DATA_IT]->(WanRte)
MERGE(FireWIT)-[:MIS_PRTCT_NTWRK]->(WanRte);
//
MATCH(WrkStInternal:WorkStationInternal{couche:'hardware'}),(IntITNet:InternalNetworkIT{couche:'cyber'})
MERGE(WrkStInternal)-[:SOFT_REQUEST]->(IntITNet)
MERGE(WrkStInternal)-[:CYBER_DATA_IT]->(IntITNet)
MERGE(WrkStInternal)-[:MIS_TRNSMT_DATA]->(IntITNet);
//
MATCH(IntITNet:InternalNetworkIT{couche:'cyber'}),(FireWIT:FirewallIT{couche:'software'})
MERGE(IntITNet)-[:SOFT_REQUEST]->(FireWIT)
MERGE(IntITNet)-[:CYBER_DATA_IT]->(FireWIT)
MERGE(IntITNet)-[:MIS_PRTCT_NTWRK]->(FireWIT);
//
MATCH(WrkStOpScada:WorkStationOperatorScada{couche:'hardware'}),(OpeITNet:OperatorNetworkIT{couche:'cyber'})
MERGE(WrkStOpScada)-[:SOFT_REQUEST]->(OpeITNet)
MERGE(WrkStOpScada)-[:CYBER_DATA_IT]->(OpeITNet)
MERGE(WrkStOpScada)-[:CYBER_DATA_OT]->(OpeITNet)
MERGE(WrkStOpScada)-[:MIS_TRNSMT_DATA]->(OpeITNet);
//
MATCH(OpeITNet:OperatorNetworkIT{couche:'cyber'}),(FireWIT:FirewallIT{couche:'software'})
MERGE(OpeITNet)-[:SOFT_REQUEST]->(FireWIT)
MERGE(OpeITNet)-[:CYBER_DATA_IT]->(FireWIT)
MERGE(OpeITNet)-[:CYBER_DATA_OT]->(FireWIT)
MERGE(OpeITNet)-[:MIS_TRNSMT_DATA]->(FireWIT);

/////////////////////////////////////On passe au Wan et Network SOC/////////////////////////////////////
MATCH(SOCNet:NetworkSOC{couche:'cyber'}),(FireWSOC:FirewallSOC{couche:'software'})
MERGE(SOCNet)-[:CYBER_DATA_IT]->(FireWSOC)
MERGE(SOCNet)-[:MIS_TRNSMT_DATA]->(FireWSOC);

MATCH(FireWSOC:FirewallSOC{couche:'software'}),(WanRte:WANRte{couche:'software'})

MERGE(FireWSOC)-[:CYBER_DATA_IT]->(WanRte)
MERGE(FireWSOC)-[:MIS_PRTCT_NTWRK]->(WanRte);

//Siem
MATCH(Siem:Siem{couche:'software'}),(ServSiem:ServersSiemSoc{couche:'hardware'})
MERGE(Siem)-[:SOFT_ANSWER_REQUEST]->(ServSiem)
MERGE(Siem)-[:CYBER_DATA_IT]->(ServSiem)
MERGE(Siem)-[:CYBER_DATA_OT]->(ServSiem)
MERGE(Siem)-[:MIS_HOST_SRVC]->(ServSiem);
//
MATCH(ServSiem:ServersSiemSoc{couche:'hardware'}),(SOCNet:NetworkSOC{couche:'cyber'})
MERGE(ServSiem)-[:SOFT_ANSWER_REQUEST]->(SOCNet)
MERGE(ServSiem)-[:CYBER_DATA_IT]->(SOCNet)
MERGE(ServSiem)-[:CYBER_DATA_OT]->(SOCNet)
MERGE(ServSiem)-[:MIS_TRNSMT_DATA]->(SOCNet);

//Logs
MATCH(LogsSoc:StockageLogsSoc{couche:'software'}),(ServLogsSoc:ServersLogsSoc{couche:'hardware'})
MERGE(LogsSoc)-[:SOFT_ANSWER_REQUEST]->(ServLogsSoc)
MERGE(LogsSoc)-[:CYBER_DATA_IT]->(ServLogsSoc)
MERGE(LogsSoc)-[:CYBER_DATA_OT]->(ServLogsSoc)
MERGE(LogsSoc)-[:MIS_HOST_SRVC]->(ServLogsSoc);
//
MATCH(ServLogsSoc:ServersLogsSoc{couche:'hardware'}),(SOCNet:NetworkSOC{couche:'cyber'})
MERGE(ServLogsSoc)-[:SOFT_ANSWER_REQUEST]->(SOCNet)
MERGE(ServLogsSoc)-[:CYBER_DATA_IT]->(SOCNet)
MERGE(ServLogsSoc)-[:CYBER_DATA_OT]->(SOCNet)
MERGE(ServLogsSoc)-[:MIS_TRNSMT_DATA]->(SOCNet);

//Workstations
MATCH(WrkStSoc:WorkStationSoc{couche:'hardware'}),(SOCNet:NetworkSOC{couche:'cyber'})
MERGE(WrkStSoc)-[:SOFT_REQUEST]->(SOCNet)
MERGE(WrkStSoc)-[:CYBER_DATA_IT]->(SOCNet)
MERGE(WrkStSoc)-[:MIS_TRNSMT_DATA]->(SOCNet);

//////////////////////////////////////Liens WAN DC IT NETWORK/////////////////////////////////////
MATCH(DCITNet:NetworkDCIT{couche:'cyber'}),(FireWDCIT:FirewallDCIT{couche:'software'})
MERGE(DCITNet)-[:SOFT_ANSWER_REQUEST]->(FireWDCIT)
MERGE(DCITNet)-[:CYBER_DATA_IT]->(FireWDCIT)
MERGE(DCITNet)-[:MIS_TRNSMT_DATA]->(FireWDCIT);

MATCH(FireWDCIT:FirewallDCIT{couche:'software'}),(WanRte:WANRte{couche:'software'})
MERGE(FireWDCIT)-[:SOFT_ANSWER_REQUEST]->(WanRte)
MERGE(FireWDCIT)-[:CYBER_DATA_IT]->(WanRte)
MERGE(FireWDCIT)-[:MIS_PRTCT_NTWRK]->(WanRte);

//MAILS
MATCH(Mail:MailIT{couche:'software'}),(ServsMailDCIT:ServersMailDataCIT{couche:'hardware'})
MERGE(Mail)-[:SOFT_ANSWER_REQUEST]->(ServsMailDCIT)
MERGE(Mail)-[:CYBER_DATA_IT]->(ServsMailDCIT)
MERGE(Mail)-[:MIS_HOST_SRVC]->(ServsMailDCIT);
//
MATCH(ServsMailDCIT:ServersMailDataCIT{couche:'hardware'}),(DCITNet:NetworkDCIT{couche:'cyber'})
MERGE(ServsMailDCIT)-[:SOFT_ANSWER_REQUEST]->(DCITNet)
MERGE(ServsMailDCIT)-[:CYBER_DATA_IT]->(DCITNet)
MERGE(ServsMailDCIT)-[:MIS_TRNSMT_DATA]->(DCITNet);

//DATABASE
MATCH(DbIT:DataBaseIT{couche:'software'}),(ServsDbDCIT:ServersDatabaseDataCIT{couche:'hardware'})
MERGE(DbIT)-[:SOFT_ANSWER_REQUEST]->(ServsDbDCIT)
MERGE(DbIT)-[:CYBER_DATA_IT]->(ServsDbDCIT)
MERGE(DbIT)-[:MIS_HOST_SRVC]->(ServsDbDCIT);
//
MATCH(ServsDbDCIT:ServersDatabaseDataCIT{couche:'hardware'}),(DCITNet:NetworkDCIT{couche:'cyber'})
MERGE(ServsDbDCIT)-[:SOFT_ANSWER_REQUEST]->(DCITNet)
MERGE(ServsDbDCIT)-[:CYBER_DATA_IT]->(DCITNet)
MERGE(ServsDbDCIT)-[:MIS_TRNSMT_DATA]->(DCITNet);

//ACTIVE DIRECTORY
MATCH(AdIT:ActiveDirectoryIT{couche:'software'}),(ServsAdDCIT:ServersActiveDirectoryDataCIT{couche:'hardware'})
MERGE(AdIT)-[:SOFT_ANSWER_REQUEST]->(ServsAdDCIT)
MERGE(AdIT)-[:CYBER_DATA_IT]->(ServsAdDCIT)
MERGE(AdIT)-[:MIS_HOST_SRVC]->(ServsAdDCIT);
//
MATCH(ServsAdDCIT:ServersActiveDirectoryDataCIT{couche:'hardware'}),(DCITNet:NetworkDCIT{couche:'cyber'})
MERGE(ServsAdDCIT)-[:SOFT_ANSWER_REQUEST]->(DCITNet)
MERGE(ServsAdDCIT)-[:CYBER_DATA_IT]->(DCITNet)
MERGE(ServsAdDCIT)-[:MIS_TRNSMT_DATA]->(DCITNet);

//LOGS
MATCH(LogsIT:StockageLogsIT{couche:'software'}),(ServsLogsDCIT:ServersLogsDataCIT{couche:'hardware'})
MERGE(LogsIT)-[:SOFT_ANSWER_REQUEST]->(ServsLogsDCIT)
MERGE(LogsIT)-[:CYBER_DATA_IT]->(ServsLogsDCIT)
MERGE(LogsIT)-[:MIS_HOST_SRVC]->(ServsLogsDCIT);
//
MATCH(ServsLogsDCIT:ServersLogsDataCIT{couche:'hardware'}),(DCITNet:NetworkDCIT{couche:'cyber'})
MERGE(ServsLogsDCIT)-[:SOFT_ANSWER_REQUEST]->(DCITNet)
MERGE(ServsLogsDCIT)-[:CYBER_DATA_IT]->(DCITNet)
MERGE(ServsLogsDCIT)-[:MIS_TRNSMT_DATA]->(DCITNet);

/////////////////////////////////////Liens vers DCOT/////////////////////////////////////

MATCH(OTNetwork:NetworkOT{couche:'cyber'}),(FireWOTST:FirewallDCOTSTA{couche:'software'})
MERGE(OTNetwork)-[:CYBER_DATA_OT]->(FireWOTST)
MERGE(OTNetwork)-[:MIS_TRNSMT_DATA]->(FireWOTST);

//SCADA
MATCH(Scad:Scada{couche:'software'}),(ServScada:ServersScadaDataCOT{couche:'hardware'})
MERGE(Scad)-[:SOFT_ANSWER_REQUEST]->(ServScada)
MERGE(Scad)-[:CYBER_DATA_OT]->(ServScada)
MERGE(Scad)-[:MIS_HOST_SRVC]->(ServScada);
//
MATCH(ServScada:ServersScadaDataCOT{couche:'hardware'}),(OTNetwork:NetworkOT{couche:'cyber'})
MERGE(ServScada)-[:SOFT_ANSWER_REQUEST]->(OTNetwork)
MERGE(ServScada)-[:CYBER_DATA_OT]->(OTNetwork)
MERGE(ServScada)-[:MIS_TRNSMT_DATA]->(OTNetwork);

//DMS
MATCH(Dms:DistriMngmtSyst{couche:'software'}),(ServDms:ServersDmsDataCOT{couche:'hardware'})
MERGE(Dms)-[:SOFT_ANSWER_REQUEST]->(ServDms)
MERGE(Dms)-[:CYBER_DATA_OT]->(ServDms)
MERGE(Dms)-[:MIS_HOST_SRVC]->(ServDms);
//
MATCH(ServDms:ServersDmsDataCOT{couche:'hardware'}),(OTNetwork:NetworkOT{couche:'cyber'})
MERGE(ServDms)-[:SOFT_ANSWER_REQUEST]->(OTNetwork)
MERGE(ServDms)-[:CYBER_DATA_OT]->(OTNetwork)
MERGE(ServDms)-[:MIS_TRNSMT_DATA]->(OTNetwork);

//DNS
MATCH(DnsOT:DomainNameSystOT{couche:'software'}),(ServDns:ServersDnsDataCOT{couche:'hardware'})
MERGE (DnsOT)-[:SOFT_ANSWER_REQUEST]->(ServDns)
MERGE (DnsOT)-[:CYBER_DATA_OT]->(ServDns)
MERGE (DnsOT)-[:MIS_HOST_SRVC]->(ServDns);
//
MATCH(ServDns:ServersDnsDataCOT{couche:'hardware'}),(OTNetwork:NetworkOT{couche:'cyber'})
MERGE(ServDns)-[:SOFT_ANSWER_REQUEST]->(OTNetwork)
MERGE(ServDns)-[:CYBER_DATA_OT]->(OTNetwork)
MERGE(ServDns)-[:MIS_TRNSMT_DATA]->(OTNetwork);

//LogsOT
MATCH(LogsOT:StockageLogsOT{couche:'software'}),(ServLogsOT:ServerslogsDataCOT{couche:'hardware'})
MERGE(LogsOT)-[:SOFT_ANSWER_REQUEST]->(ServLogsOT)
MERGE(LogsOT)-[:CYBER_DATA_OT]->(ServLogsOT)
MERGE(LogsOT)-[:MIS_USE_RSSRC]->(ServLogsOT);
//
MATCH(ServLogsOT:ServerslogsDataCOT{couche:'hardware'}),(OTNet:NetworkOT{couche:'cyber'})
MERGE(ServLogsOT)-[:SOFT_ANSWER_REQUEST]->(OTNet)
MERGE(ServLogsOT)-[:CYBER_DATA_OT]->(OTNet)
MERGE(ServLogsOT)-[:MIS_TRNSMT_DATA]->(OTNet);

//Tad
MATCH(TAD:TerminalAccesDistant{couche:'software'}),(ServTad:ServersTadDataCCOT{couche:'hardware'})
MERGE(TAD)-[:SOFT_ANSWER_REQUEST]->(ServTad)
MERGE(TAD)-[:CYBER_DATA_OT]->(ServTad)
MERGE(TAD)-[:MIS_USE_RSSRC]->(ServTad);

MATCH(ServTad:ServersTadDataCCOT{couche:'hardware'}),(OTNetwork:NetworkOT{couche:'cyber'})
MERGE(ServTad)-[:SOFT_ANSWER_REQUEST]->(OTNetwork)
MERGE(ServTad)-[:CYBER_DATA_OT]->(OTNetwork)
MERGE(ServTad)-[:MIS_TRNSMT_DATA]->(OTNetwork);

//Vers WAN
MATCH(WanRte:WANRte{couche:'software'}),(FireWITOT:FirewallDCITOT{couche:'software'})
MERGE(WanRte)-[:SOFT_REQUEST]->(FireWITOT)
MERGE(WanRte)-[:CYBER_DATA_OT]->(FireWITOT)
MERGE(WanRte)-[:MIS_TRNSMT_DATA]->(FireWITOT);

MATCH(FireWITOT:FirewallDCITOT{couche:'software'}),(OTNetwork:NetworkOT{couche:'cyber'})
MERGE(FireWITOT)-[:SOFT_REQUEST]->(OTNetwork)
MERGE(FireWITOT)-[:CYBER_DATA_OT]->(OTNetwork)
MERGE(FireWITOT)-[:MIS_PRTCT_NTWRK]->(OTNetwork);

/////////////////////////////////////Liens entre RTU, FW et routeurs/////////////////////////////////////
MATCH(FireWSt:FirewallStation{couche:'software'}),(RTU:RemoteTerminalUnit{couche:'hardware'})
MERGE (FireWSt)-[:CYBER_DATA_OT]->(RTU)
MERGE (FireWSt)-[:MIS_PRTCT_NTWRK]->(RTU);

MATCH(RoutS:RouteurOTSud{couche:'hardware'}),(FireWSt:FirewallStation{couche:'software'})
MERGE (RoutS)-[:CYBER_DATA_OT]->(FireWSt)
MERGE (RoutS)-[:MIS_TRNSMT_DATA]->(FireWSt);

//
MATCH(RoutE:RouteurOTEast{couche:'hardware'}),(RoutS:RouteurOTSud{couche:'hardware'})
MERGE(RoutE)-[:CYBER_DATA_OT]->(RoutS)
MERGE(RoutE)-[:MIS_CNNCT_NTWRK]->(RoutS);

//
MATCH(RoutW:RouteurOTWest{couche:'hardware'}),(RoutS:RouteurOTSud{couche:'hardware'})
MERGE(RoutW)-[:CYBER_DATA_OT]->(RoutS)
MERGE(RoutW)-[:MIS_CNNCT_NTWRK]->(RoutS);

//
MATCH(RoutN:RouteurOTNorth{couche:'hardware'}),(RoutE:RouteurOTEast{couche:'hardware'})
MERGE(RoutN)-[:CYBER_DATA_OT]->(RoutE)
MERGE(RoutN)-[:MIS_CNNCT_NTWRK]->(RoutE);

//
MATCH(RoutN:RouteurOTNorth{couche:'hardware'}),(RoutW:RouteurOTWest{couche:'hardware'})
MERGE(RoutN)-[:CYBER_DATA_OT]->(RoutW)
MERGE(RoutN)-[:MIS_CNNCT_NTWRK]->(RoutW);

//
MATCH(FireWOTST:FirewallDCOTSTA{couche:'software'}),(RoutN:RouteurOTNorth{couche:'hardware'})
MERGE(FireWOTST)-[:CYBER_DATA_OT]->(RoutN)
MERGE(FireWOTST)-[:MIS_PRTCT_NTWRK]->(RoutN);
```

## Queries for the interactions between Technicians in the station and it's components

```cypher
//Types de liens

// Vis à vis de OT
// Physical: contact et danger dans le sens Composant élec => opérateur
// Mission: Repair, maintain,  ? 
// Humain: Have access, Manipulate, execute Order ,  ?
// Pas de cyber ou de SH, pas de PSA

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(WorkSt:WorkStation{couche:'hardware'})
MERGE (Opers)-[:HUM_ACCESS]->(WorkSt)
MERGE (Opers)-[:HUM_INTERACT]->(WorkSt)
MERGE (Opers)-[:HUM_USE_SRVC]->(WorkSt)
MERGE (Opers)-[:MIS_PP_MONITOR_OT]->(WorkSt)
MERGE (Opers)-[:MIS_PP_COMMAND_OT]->(WorkSt);

//Liens Physiques OP vers composants Ligne Entrée
MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Sectio_E:SectionneurLigneE{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Sectio_E)
MERGE (Sectio_E)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(Sectio_E)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Sectio_E)
MERGE (Opers)-[:HUM_MNPLT]->(Sectio_E)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Sectio_E)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Sectio_E);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Sectio_BE:SectionneurBusE{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Sectio_BE)
MERGE (Sectio_BE)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(Sectio_BE)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Sectio_BE)
MERGE (Opers)-[:HUM_MNPLT]->(Sectio_BE)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Sectio_BE)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Sectio_BE);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(TransC_LB:TransformateurCourantLigneBus{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(TransC_LB)
MERGE (TransC_LB)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(TransC_LB)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(TransC_LB)
MERGE (Opers)-[:HUM_MNPLT]->(TransC_LB)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(TransC_LB)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(TransC_LB);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_Capteurs_LB)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_Capteurs_LB)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_Capteurs_LB)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_Capteurs_LB)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_Capteurs_LB)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_Capteurs_LB);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_IED_LB:EnsembleIedLigneBus{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_IED_LB)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_IED_LB)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_IED_LB)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_IED_LB)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_IED_LB)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_IED_LB);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Disj_LB:DisjoncteurLigneBus{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Disj_LB)
MERGE (Disj_LB)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(Disj_LB)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Disj_LB)
MERGE (Opers)-[:HUM_MNPLT]->(Disj_LB)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Disj_LB)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Disj_LB);

//Liens Physiques OP vers composants Ligne Transformateur
MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Sectio_TP:SectionneurTransformateur{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Sectio_TP)
MERGE (Sectio_TP)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(Sectio_TP)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Sectio_TP)
MERGE (Opers)-[:HUM_MNPLT]->(Sectio_TP)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Sectio_TP)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Sectio_TP);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Sectio_S:SectionneurLigneS{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Sectio_S)
MERGE (Sectio_S)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(Sectio_S)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Sectio_S)
MERGE (Opers)-[:HUM_MNPLT]->(Sectio_S)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Sectio_S)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Sectio_S);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(TransC_TP:TransformateurCourantTransformateur{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(TransC_TP)
MERGE (TransC_TP)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(TransC_TP)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(TransC_TP)
MERGE (Opers)-[:HUM_MNPLT]->(TransC_TP)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(TransC_TP)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(TransC_TP);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_Capteurs_TP:EnsembleDeCapteursTransformateur{couche:'sensor'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_Capteurs_TP)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_Capteurs_TP)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_Capteurs_TP)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_Capteurs_TP)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_Capteurs_TP)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_Capteurs_TP);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_IED_TP:EnsembleIedTransformateur{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_IED_TP)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_IED_TP)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_IED_TP)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_IED_TP)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_IED_TP)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_IED_TP);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Disj_TP:DisjoncteurTransformateur{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Disj_TP)
MERGE (Disj_TP)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(Disj_TP)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Disj_TP)
MERGE (Opers)-[:HUM_MNPLT]->(Disj_TP)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Disj_TP)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Disj_TP);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(TransP_BT:TransformateurPuissanceBT{couche:'physique'})
MERGE (Opers)-[:PHYS_CNTCT]->(TransP_BT)
MERGE (TransP_BT)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(TransP_BT)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(TransP_BT)
MERGE (Opers)-[:HUM_MNPLT]->(TransP_BT)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(TransP_BT)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(TransP_BT);

//Liens Physiques OP vers composants BUSE
MATCH (Opers:TechnicienStation{couche:'opérateur'}),(TransT_BusE:TransformateurTensionBusE{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(TransT_BusE)
MERGE (TransT_BusE)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(TransT_BusE)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(TransT_BusE)
MERGE (Opers)-[:HUM_MNPLT]->(TransT_BusE)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(TransT_BusE)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(TransT_BusE);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_IED_BusE:EnsembleIedBusE{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_IED_BusE)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_IED_BusE)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_IED_BusE)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_IED_BusE)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_IED_BusE)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_IED_BusE);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_Capteurs_BusE:EnsembleDeCapteursBusE{couche:'sensor'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_Capteurs_BusE)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_Capteurs_BusE)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_Capteurs_BusE)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_Capteurs_BusE)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_Capteurs_BusE)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_Capteurs_BusE);

//Liens Physiques OP vers composants BUSS
MATCH (Opers:TechnicienStation{couche:'opérateur'}),(TransT_BusS:TransformateurTensionBusE{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(TransT_BusS)
MERGE (TransT_BusS)-[:PHYS_DNGR]-(Opers)
MERGE (Opers)-[:HUM_ACCESS]->(TransT_BusS)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(TransT_BusS)
MERGE (Opers)-[:HUM_MNPLT]->(TransT_BusS)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(TransT_BusS)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(TransT_BusS);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_IED_BusS:EnsembleIedBusE{couche:'actuator'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_IED_BusS)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_IED_BusS)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_IED_BusS)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_IED_BusS)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_IED_BusS)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_IED_BusS);

MATCH (Opers:TechnicienStation{couche:'opérateur'}),(Hub_Capteurs_BusS:EnsembleDeCapteursBusE{couche:'sensor'})
MERGE (Opers)-[:PHYS_CNTCT]->(Hub_Capteurs_BusS)
MERGE (Opers)-[:HUM_ACCESS]->(Hub_Capteurs_BusS)
MERGE (Opers)-[:HUM_EXECUTE_ORDER]->(Hub_Capteurs_BusS)
MERGE (Opers)-[:HUM_MNPLT]->(Hub_Capteurs_BusS)
MERGE (Opers)-[:MIS_PP_MAINTAIN_OT]->(Hub_Capteurs_BusS)
MERGE (Opers)-[:MIS_PP_REPAIR_OT]->(Hub_Capteurs_BusS);
```
## Queries for the interactions between Operators, Humans and the workstations
```cypher
//Liens utilisateurs
MATCH(Ituser:UserOfITServices{couche:'user'}),(WrkStInternal:WorkStationInternal{couche:'hardware'})
MERGE (Ituser)-[:HUM_ACCESS]->(WrkStInternal)
MERGE (Ituser)-[:HUM_INTERACT]->(WrkStInternal)
MERGE (WrkStInternal)-[:HUM_INTERACT]->(Ituser)
MERGE (Ituser)-[:HUM_USE_SRVC]->(WrkStInternal)
MERGE (Ituser)-[:MIS_PP_BUSINESS]->(WrkStInternal);

MATCH(SocOper:SocOperator{couche:'user'}),(WrkStSoc:WorkStationSoc{couche:'hardware'})
MERGE (SocOper)-[:HUM_ACCESS]->(WrkStSoc)
MERGE (SocOper)-[:HUM_INTERACT]->(WrkStSoc)
MERGE (WrkStSoc)-[:HUM_INTERACT]->(SocOper)
MERGE (SocOper)-[:HUM_USE_SRVC]->(WrkStSoc)
MERGE (SocOper)-[:MIS_PP_MONITOR_IT]->(WrkStSoc);

MATCH(ScadOper:ScadaOperator{couche:'user'}),(WrkStOpScada:WorkStationOperatorScada{couche:'hardware'})
MERGE (ScadOper)-[:HUM_ACCESS]->(WrkStOpScada)
MERGE (ScadOper)-[:HUM_INTERACT]->(WrkStOpScada)
MERGE (WrkStOpScada)-[:HUM_INTERACT]->(ScadOper)
MERGE (ScadOper)-[:HUM_USE_SRVC]->(WrkStOpScada)
MERGE (ScadOper)-[:MIS_PP_MONITOR_OT]->(WrkStOpScada)
MERGE (ScadOper)-[:MIS_PP_COMMAND_OT]->(WrkStOpScada);

MATCH(ItGuy:ItGuy{couche:'user'}),(WrkStIT:WorkStationIT{couche:'hardware'})
MERGE (ItGuy)-[:HUM_ACCESS]->(WrkStIT)
MERGE (ItGuy)-[:HUM_INTERACT]->(WrkStIT)
MERGE (WrkStIT)-[:HUM_INTERACT]->(ItGuy)
MERGE (ItGuy)-[:HUM_USE_SRVC]->(WrkStIT)
MERGE (ItGuy)-[:MIS_PP_COMMAND_IT]->(WrkStIT)
MERGE (ItGuy)-[:MIS_PP_MAINTAIN_IT]->(WrkStIT)
MERGE (ItGuy)-[:MIS_PP_REPAIR_IT]->(WrkStIT);
```



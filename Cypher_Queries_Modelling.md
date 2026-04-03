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
## Queries to the IT infrastructure's components

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




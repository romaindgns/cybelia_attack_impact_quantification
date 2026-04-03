```cypher
CREATE
//Ligne vers Bus
(Ligne_E:LigneEntrée{couche:'physique'}),
(Sectio_E:SectionneurLigneE{couche:'actuator'}),
(Sectio_BE:SectionneurBus{couche:'actuator'}),
(TransC_LB:TransformateurCourantLigneBus{couche:'actuator'}),
(Hub_Capteurs_LB:EnsembleDeCapteursLigneBus{couche:'sensor'}),
(Hub_IED_LB:EnsembleIedLigneBus{couche:'actuator'}),
(Disj_LB:DisjoncteurLigneBus{couche:'actuator'});
```



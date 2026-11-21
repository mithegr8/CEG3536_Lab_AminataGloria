# JOURNAL.md : Journal d'équipe, CEG 3536, projet final (automne 2026)

- Équipe (5 membres) : Aminata, Gloria, Racine, Idriss, Mouhammad
- Section : A01
- Dépôt Git : https://github.com/mithegr8/CEG3536_Lab_AminataGloria
- Variante L4-A attribuée : 0
- Journal du laboratoire 3 : `JOURNAL_Lab3.md`

# Partie I

## Partie I : Rôles et rotation

| Séance | Réalise | Valide | Pilote I²C écrit par chacun | Pilote SPI ou routine assembleur de chacun |
|---|---|---|---|---|
| Séance 1 (20 nov.) | Aminata, Gloria, Idriss (essais T1 à T4 sur la carte) | Racine, Mouhammad | Aminata : pilotes I²C (i2c.c, ioexp.c, lcd.c, touchpad.c) ; modification de la commande `w` avec Idriss | Aminata : `trame_xor8` (somme.s), `exti_l4.s` |
| Entre les séances | | | | |
| Séance 2 | | | | |

## Partie I : Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| Pas de l'horloge I²C | (PRESC + 1) / 4 MHz | (0 + 1) / 4 MHz = 250 ns | T1 |
| SCL bas / SCL haut | (SCLL + 1) × pas / (SCLH + 1) × pas | 20 × 250 ns = 5,0 µs (min. 4,7) / 16 × 250 ns = 4,0 µs (min. 4,0) | T1 |
| Fréquence SCL (avec les délais de synchronisation) | 1 / (5,0 + 4,0 µs + synchronisation ≈ 1 µs) | ≈ 100 kHz, un peu moins (exigé : 90 à 100 kHz) | T1 |
| `TIMINGR` retenu | PRESC = 0, SCLDEL = 4, SDADEL = 2, SCLH = 0x0F, SCLL = 0x13 | 0x00420F13 | T1 |
| Durée d'une écriture de deux octets de données | 29 bits (START, adresse, 2 octets, ACK, STOP) / fréquence SCL | prévu ≈ 290 µs ; mesuré au DWT : 1501 cycles = 375 µs, logiciel compris | T1 (DWT) |
| Débit SPI | PCLK2 / 2^(BR + 1) | BR = 001 : 4 MHz / 4 = 1 MHz | T7 |
| Pire cas de réécriture du LCD | 32 caractères × (position + caractère) × 1 tick | 64 ms (< 70 ms) | T3 |
| Transactions I²C par seconde au repos | commutateurs toutes les 10 ms ; DEL et LCD seulement sur changement | ≈ 100 par seconde, compteur `i2c_transactions()` lu par la commande `o` | T10 |

## Partie I : Relevés des modules

| Module | Relevé | Valeur | Preuve (fichier) |
|---|---|---|---|
| Bus I²C | Adresses trouvées par `i2c_scan` | 0x2D, 0x38, 0x39, 0x3E, 0x5A (5 adresses, aucune autre) : `I,5,2D,38,39,3E,5A*09` | PROJET.docx, T1 |
| Bus I²C | `I2C2->ISR` après une transaction ; `TIMINGR` ; `CR2` | ISR = 0x00000001 (TXE = 1, BUSY = 0) ; TIMINGR = 0x00420F13 ; CR2 = 0x02010472 (dernière transaction : lecture de 1 octet à 0x39, AUTOEND) | PROJET.docx, T1 (SFRs) |
| Extenseur des DEL | DEL observées | Y3 suit Touch En ; R4 s'allume avec E-Stop | photos T2-c |
| Extenseur des commutateurs | Événements produits | un `E,…,10,n` par appui sur SWn, aucun au relâchement ; SW6 et SW7 ensemble : deux lignes au même instant (724257 ms) | PROJET.docx, T2 |
| LCD | Octets de commande (fonction, allumage, effacement, mode) | 0x28, 0x0C, 0x01, 0x06 : l'afficheur s'initialise et affiche les deux pages | photos T3 |
| Rétroéclairage | Registres rouge / vert / bleu (0x01 à 0x03 à l'adresse 0x2D) | rouge en ARRÊT, vert en AVANT, bleu en ARRIÈRE, rouge clignotant en URGENCE | photos T3 |
| TouchPad | Lecture de l'état sur demande THI | une ligne `K,` par appui et `K,0` au relâchement : la lecture fonctionne | PROJET.docx, T4 |
| Capteur environnemental | Identifiant lu ; page utilisée pour chaque registre | | |
| Capteur environnemental | Température / humidité / pression au repos ; écart avec le capteur analogique | | |
| PIR | Durée du niveau haut ; `pir_t_ms` de deux fronts successifs | | |

Table des touches (électrode → touche), relevée au débogueur :

| Électrode | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Touche | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | Fn | Next |

Relevé fait avec la commande `k` (séance 1) : touches appuyées dans l'ordre 0 à 9, Fn, Next ;
valeurs lues 1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024, 2048 (un bit par électrode).
Table reportée dans `tp_table` (touchpad.c).

## Partie I : Essais hôte (E11)

| Plateforme | Compilateur et version | `test_evq` (réussis / échecs) | `test_tableau` (réussis / échecs) | Date et validation Git |
|---|---|---|---|---|
| Windows | | | | |
| Linux | gcc 13.3.0 (Ubuntu 24.04) | 9 / 0 | 13 / 0 | 20 nov. 2026 |
| STM32 (cible) | arm-none-eabi-gcc | commande `q` : | commande `f` : `F,echecs=0` | 20 nov. 2026 |

Cas ajoutés à `tableau_test.c` (au-delà des 6 fournis) :

| Cas | Description | Résultat |
|---|---|---|
| 7 | MARCHE_ARRIÈRE, mode MANUEL, 112,0 tr/min, plafond 30 % | réussi (hôte) |
| 8 | ARRÊT_ÉVÉNEMENT, cause DÉFAUT | réussi (hôte) |
| 9 | Fin de trajet (ARRÊT, cause FIN) | réussi (hôte) |
| 10 | Page VERROU : ligne 2 identique à la page de conduite | réussi (hôte) |
| 11 | Mesure environnementale non valide | réussi (hôte) |
| 12 | Températures négatives (capteur et analogique) | réussi (hôte) |
| 13 | Vitesse à quatre chiffres (1234,5 tr/min) | réussi (hôte) |

## Partie I : Banc de `trame_xor8` (T9)

| Cas | Attendu | Obtenu | Verdict |
|---|---|---|---|
| Chaîne vide | 0x00 | 0x00 (simulateur Cortex-M) | réussi |
| « A » | 0x41 | 0x41 (simulateur Cortex-M) | réussi |
| « T,0 » | 0x48 | 0x48 (simulateur Cortex-M) | réussi |
| « 123456789 » | 0x31 | 0x31 (simulateur Cortex-M) | réussi |
| Octets 0x00 à 0xFF | 0x00 | 0x00 (simulateur Cortex-M) | réussi |

Sur la carte, six trames prises dans les captures (lignes I, T et E) ont été recalculées à la main :
les six sommes `*HH` sont exactes.

## Partie I : Journal des séances

### Séance 1 : vendredi 20 novembre 2026, réalise : Aminata, Gloria, Idriss ; valide : Racine, Mouhammad
- Objectifs : pilote I²C et balayage (E1, T1), extenseurs (E2, T2), LCD et rétroéclairage (E3, T3),
  TouchPad (E4, T4), premières commandes du TouchPad (E5, T5).
- Fait :
  - T1 réussi : 5 adresses trouvées, TIMINGR = 0x00420F13, écriture de 2 octets mesurée au DWT,
    écriture vers une adresse vide refusée (NACK) et comptée, bus libre ensuite.
  - T2 : un événement par appui sur un commutateur, aucun au relâchement ; deux commutateurs
    ensemble donnent deux événements ; DEL Y3 (Touch En) et R4 (E-Stop) vérifiées.
  - T3 : les quatre états affichés avec la bonne couleur (photos), page des capteurs par SW8,
    test du formatage sur la cible `F,echecs=0`.
  - T4 : relevé de la table des touches avec `k` (les 12 touches répondent).
- Décisions :
  - une seule transaction I²C par tick, par priorité : TouchPad sur THI, commutateurs toutes les
    10 ms, DEL si l'image change, balayage, LCD un caractère à la fois ;
  - aucune transaction I²C dans une interruption : PIR et THI lèvent seulement un drapeau ;
  - la commande `w` a été modifiée par Aminata et Idriss pour faire, en une fois, la mesure DWT d'une
    écriture de 2 octets et l'écriture vers une adresse vide (lignes `W,…`).
- Difficultés et solutions :
  - la console de STM32CubeIDE affiche les trames mais n'envoie pas les caractères tapés :
    les commandes ont été envoyées depuis un terminal série externe sur COM4 ;
  - les trames T arrivent toutes les 283 ms au lieu de 200 ms et la colonne `ticks_perdus`
    augmente de 83 par trame : la boucle prend plus de 1 ms par tour. Cause probable : le
    formatage des deux lignes du LCD à chaque tick (4 MHz, code -O0). Correction préparée :
    appeler `tableau_step` toutes les 50 ms.
- Mesures :
  - écriture de 2 octets : 1501 cycles = 375 µs. Ce temps comprend le logiciel (appel, attentes,
    -O0) : 29 bits / 375 µs = 77 kHz est une borne inférieure du débit. Débit attendu d'après
    TIMINGR : environ 95 kHz ;
  - adresse vide : statut 1 (NACK), erreurs + 1, busy = 0 ; la colonne `i2c_erreurs` des
    trames passe de 0 à 1 puis reste stable.
- Validations Git :

### Entre les séances : travail sur hôte
- Fait (tableau.c, test_tableau, fsm.c, lecture de l4a_defaut.c) :
- Validations Git :

### Séance 2 : `<date>` : réalise : `<nom>` / valide : `<nom>`
- Objectifs :
- Fait :
- Décisions :
- Difficultés et solutions :
- Mesures (capteur, PIR, télémétrie, L4-A) :
- Validations Git :

## Partie I : Composante L4-A : diagnostic d'un défaut TouchPad/I²C

Variante : `<n>`, validation Git de la version défectueuse :, de la version corrigée :

1. Symptôme (attendu, observé, conditions de reproduction) :

2. Hypothèses, classées avant toute mesure :

| Rang | Hypothèse | Niveau (lignes et broches / adressage et protocole / configuration du composant / interruption et garde) | Ce qu'on observerait si elle était vraie |
|---|---|---|---|
| 1 | | | |
| 2 | | | |
| 3 | | | |

3. Mesures :

| Hypothèse visée | Mesure (registre, balayage, relecture, compteur) | Résultat | Preuve (fichier) |
|---|---|---|---|
| | | | |

4. Essai discriminant, prédiction écrite **avant** l'essai (date et heure) :

   Résultat de l'essai :

5. Interprétation (hypothèses écartées, hypothèse retenue, pourquoi) :

6. Correction et non-régression (les 12 touches, la garde Touch En) :

7. Conclusion et limites (ce qui est démontré, ce qui est supposé, ce qui reste à vérifier) :

## Partie I : Table des états et des priorités (en vue du jalon J3)

| État | Événement (par priorité décroissante) | Condition | État suivant | Sorties (moteur, DEL, avertisseur, LCD) |
|---|---|---|---|---|
| ARRÊT | | | | |
| MARCHE_AVANT | | | | |
| MARCHE_ARRIÈRE | | | | |
| ARRÊT_URGENCE | | | | |
| ARRÊT_ÉVÉNEMENT | | | | |

Priorités des interruptions (valeur, justification) : E-Stop, encodeur, SysTick, PIR, THI -

## Partie I : Tableau des essais (T1 à T10)

| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Bus I²C et balayage | 20 nov. | `I,5,2D,38,39,3E,5A*09` ; TIMINGR = 0x00420F13 ; 375 µs (1501 cycles) pour 2 octets ; adresse vide : NACK, erreur comptée, busy = 0 | réussi ; débit mesuré au DWT : 77 kHz au minimum, logiciel compris | PROJET.docx, T1 |
| T2 Extenseurs | 20 nov. | un `E,…,10,1` par appui sur SW1, aucun au relâchement ; SW6 + SW7 : deux événements ; Y3 et R4 allumées | partiel | PROJET.docx, T2 ; photos |
| T3 LCD et rétroéclairage | 20 nov. | ARRÊT rouge, AVANT vert, ARRIÈRE bleu, URGENCE rouge clignotant (état 3 dans les trames) ; page des capteurs par SW8 ; `F,echecs=0` | réussi | photos ; PROJET.docx, T3 |
| T4 TouchPad et garde | 20 nov. | relevé `k` : 1, 2, 4, …, 2048 pour 0 à 9, Fn, Next | partiel | PROJET.docx, T4 |
| T5 Commandes du TouchPad | | | | |
| T6 PIR | | | | |
| T7 Capteur environnemental | | | | |
| T8 Événements et priorités | | | | |
| T9 Télémétrie contrôlée | | | | |
| T10 Robustesse et non-régression | | | | |

# Partie II

## Partie II : Rôles et rotation

| Période | Réalise | Valide | Module sans matériel de chacun | Routine assembleur ou intégration de chacun |
|---|---|---|---|---|
| Entre les séances 1 et 2 (prise en main de l'outil) | | | | |
| Séance 2 (24 / 27 nov.) | | | | |
| Jalon J3 (27 nov.) | | | | |
| Entre les séances 2 et 3 | | | | |
| Séance 3 (1er / 4 déc.) | | | | |
| Démonstration finale (4 / 8 déc.) | | | | |

## Partie II : Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| Fronts par fenêtre à 100 tr/min | 100 × 748 / 600 | | T14 |
| Résolution de la vitesse à 100 tr/min | 1 front / fronts par fenêtre | | T14 |
| Constante Q16.16 de la fenêtre de 100 ms | 600 × 65 536 / 748, arrondie | | banc du lab. 3 |
| Fronts du trajet de référence | 105 × 748 | | T13 |
| Durée prévue du trajet de référence | somme des tours / (tr/min / 60) | | T13 |
| Commande initiale du régulateur à 150 tr/min | 150 × 100 / 340 | | T14 |
| Durée de masquage admise | 200 cycles / 4 MHz | | T11 |
| Occupation de la file au pire cas prévu | | | T11 |

## Partie II : Automate (L5-B)

| Date | Version de la table (somme) | Résultat du vérificateur | Erreurs et contre-exemples | Correction |
|---|---|---|---|---|
| | | | | |

Invariants imposés et ajoutés par l'équipe :

| Invariant | Type | Pourquoi | Résultat |
|---|---|---|---|
| | | | |

Version fautive conservée : fichier `…`, validation Git `…`, contre-exemple :

```
(copier ici le contre-exemple produit par l'outil)
```

Correspondance entre les événements de la file et les entrées du modèle
(traduction faite dans `fsm.c`) :

| Événement, garde | Entrée du modèle |
|---|---|
| | |

Comparaison des traces (au moins trois séquences) :

| Séquence | Pas | Hôte Windows | Hôte Linux | Cible (commande `g`) |
|---|---|---|---|---|
| | | | | |

## Partie II : File d'événements et sections critiques (L5-A)

| Question | Réponse de l'équipe | Preuve |
|---|---|---|
| Producteurs (gestionnaires) et consommateur | | |
| Instructions placées dans la section critique | | |
| Politique de débordement retenue, et pourquoi | | |
| Mécanisme de garantie pour E-Stop et défaut | | |
| `crit_autotest` (commande `c`) : cas réussis | | |
| `crit_max_cycles` relevé (repos, scénario S3, rafale) | | |
| `evq_max` relevé (repos, scénario S3, rafale) | | |
| Rafale `e` : événements acceptés, refusés, consommés | | |
| Délai de E-Stop pendant une rafale | | |

## Partie II : Réglage du régulateur

Consigner chaque essai, y compris ceux qui échouent : c'est la justification
des gains retenus.

| Essai | Kp (Q8) | Ki (Q8) | Consigne (tr/min) | Vitesse à 2 s | Erreur (%) | Dépassement | Observation |
|---|---|---|---|---|---|---|---|
| 1 | 26 | 26 | | | | | |
| | | | | | | | |

Gains retenus : Kp = `…`, Ki = `…`, raison :

## Partie II : Relevés de la partie II

| Essai | Relevé | Valeur | Preuve (fichier) |
|---|---|---|---|
| T13 | Distance aux changements de segment (20, 60, 75) et à la fin (105) | | |
| T13 | Reprise après un arrêt au feu rouge : distance avant / après | | |
| T14 | Erreur 2 s après chaque changement de consigne (quatre segments) | | |
| T14 | Plafond abaissé pendant S2 : commande et vitesse | | |
| T15 | Délai entre la commande `s` et l'état DÉFAUT | | |
| T15 | Délai entre la commande `n` et l'état DÉFAUT | | |
| T16 | Limitation à 42,0 °C ; arrêt à 52,0 °C ; sortie à 25,0 °C | | |
| T18 | Initialisation : durée de INIT, ressources vérifiées | | |
| T18 | Démarrage avec un module absent (commande `d`) | | |

## Partie II : Portage (L5-C)

| | Windows x86-64 | Linux x86-64 | STM32 |
|---|---|---|---|
| Compilateur et version | | | |
| Options | | | |
| Tailles : `int`, `long`, pointeur, `size_t` | | | |
| Fichiers remplacés (couche d'adaptation) | | | |
| `test_evq` / `test_tableau` | | | |
| `test_trajet` / `test_regul` / `test_surv` | | | |
| `test_automate` (traces) | | | |
| Rejeu de la trace du scénario `…` | | | |
| Rejeu de la trace du scénario `…` | | | |

Différences observées et explication :

## Partie II : Décision collective et désaccord (L5-D)

Décision technique prise en équipe :

| Élément | Contenu |
|---|---|
| Question à trancher | |
| Options envisagées | |
| Critères | |
| Choix et raison | |
| Résultat observé après coup | |

Désaccord (réel, ou scénario de `dossier/scenario_desaccord.md`) :

| Élément | Contenu |
|---|---|
| Objet du désaccord | |
| Position de chacun | |
| Démarche suivie | |
| Issue | |

## Partie II : Séances

### Séance 2 : date : `…`

- Objectifs :
- Réalisé :
- Problèmes rencontrés, hypothèses, essais :
- Validations Git (auteur, identifiant, objet) :
- À faire avant la séance 3 :

### Entre les séances 2 et 3

- Réalisé :
- Validations Git :

### Séance 3 : date : `…`

- Objectifs :
- Réalisé :
- Problèmes rencontrés, hypothèses, essais :
- Validations Git :
- À faire avant la démonstration finale :

### Démonstration finale : date : `…`

- Scénarios S1 à S5 : résultat de chacun
- Questions posées et réponses à approfondir dans le rapport :

## Vidéo de présentation (énoncé, section 7.2)

Durée visée : 7 à 10 minutes. Filmer les scénarios dès qu'ils fonctionnent, sans attendre la veille de la remise.

| Séquence | Durée prévue | Qui parle | Ce qui est montré (véhicule, afficheur, télémétrie, code) | Tournée le |
|---|---|---|---|---|
| Présentation de l'équipe, du véhicule et de l'architecture | 1 min | | | |
| Démonstration : conduite manuelle, trajet autonome, arrêt et reprise, E-Stop, défaut injecté | 4 min | | | |
| Explication technique du membre A : `…` | 1 min 30 | | | |
| Explication technique du membre B : `…` | 1 min 30 | | | |
| Bilan : limites, ce qui serait fait autrement | 1 min | | | |

Durée finale : `…`, fichier ou lien : `…`, outils de montage : `…`

## Sources et outils

| Source ou outil | Usage |
|---|---|
| Compléments du laboratoire 4 et du projet | |
| Outil `automate.py` | |
| Corrigé du laboratoire 3 (le cas échéant) | |
| Fiches techniques, manuel | |
| Outils d'IA générative (outil, version, usage) ou « aucun usage » | Claude (Anthropic), Opus 5.5 : aide à l'écriture du code de la séance 1, explications des modules, préparation des essais et mise en forme du journal ; chaque résultat est vérifié sur la carte |

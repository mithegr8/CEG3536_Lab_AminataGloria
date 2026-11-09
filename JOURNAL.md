# JOURNAL.md — Journal d'équipe, CEG 3536, laboratoire 2 « Propulsion et mesure de vitesse » (automne 2026)

Équipe : Aminata et Gloria — Section : `A01 ` — Dépôt Git : `https://github.com/mithegr8/CEG3536_Lab_AminataGloria`

Projet : `CEG3536_Lab2_<équipe>` (STM32CubeIDE, NUCLEO-L552ZE-Q, plateforme Leafy)

## Rôles et rotation

Chaque séance, un membre réalise et l'autre valide ; les rôles sont inversés d'une séance à l'autre. Chaque membre doit avoir configuré ou mesuré au moins un des trois périphériques (PWM, encodeur, SysTick).

| Séance | Réalise | Valide | Périphérique configuré ou mesuré par chacun | Fichiers écrits ou modifiés par chacun |

| Séance 1 | `Gloria` | `Aminata` | | |
| Séance 2 | `Aminata` | `Gloria` | | |

## Calculs préalables

| Grandeur | Formule | Valeur | Vérifiée par |
|---|---|---|---|
| Fréquence PWM | 4 MHz / ((PSC + 1) × (ARR + 1)) | | T3 |
| CCR1 pour une consigne de p % | p × (ARR + 1) / 100 | | T3 |
| Fronts par tour (arbre de sortie) | | 748 | T5 |
| Vitesse en tr/min | fronts par 500 ms × 120 / 748 | | T6 |
| Fronts attendus par 500 ms à 100 % | 348 tr/min × 748 / 120 | | T6 |
| Recharge de SysTick (tick de 1 ms) | 4 MHz × 1 ms − 1 | | T10 |
| Durée de la rampe 0 → 100 % | 100 pas × 10 ms | | T4 |

## Journal des séances

### Séance 1 — `16 Octobre` — réalise : `<nom>` / valide : `<nom>`
- Objectifs :
- Fait (fichiers `motor.s`, `encoder.s`, `systick.s`, `buzzer.s`, `lab2.h`) :
- Décisions (sens de rotation réel de l'arbre pour AI1 = 1, constante `ENC_SENS_INVERSE`, ordre d'initialisation, refus de changement de sens si CCR1 ≠ 0) :
- Difficultés et solutions :
- Mesures :
- Validations Git (qui, message) :

### Séance 2 — `23 Octobre` — réalise : `<nom>` / valide : `<nom>`
- Objectifs :
- Fait (`fsm.s`, `estop.s`, intégration, essais T1 à T10) :
- Décisions :
- Difficultés et solutions (débogueur : Live Expressions, Suspend/Resume, points d'arrêt) :
- Mesures :
- Validations Git (qui, message) :

## Tableau des essais (T1 à T10)

| Essai | Date | Résultat observé | Verdict | Preuve (captures) |
|---|---|---|---|---|
| T1 Réinitialisation | | ARRÊT, CCR1 = 0, STB = 0, buzzer silencieux | | images 1 à 3 (Live Expressions, SFR TIM1, SFR GPIO) |
| T2 Cycle User et sens | | rouge, verte, rouge, bleue, rouge ; AI1/AI2/STB | | images 4 à 6 |
| T3 PWM 25 / 50 / 75 / 100 % | | PSC, ARR, CCR1 ; fréquence et rapport cyclique | | images 7 à 14 |
| T4 Rampe | | `pwm_pct` et `ticks_ms` | | images 15 et 16 |
| T5 Encodeur | | `enc_count` dans les deux sens | | images 17 à 20 |
| T6 Vitesse (avant, arrière) | | `vitesse_fronts` sur 3 fenêtres par palier | | images 21 à 28 |
| T7 E-Stop | | CCR1 = 0, STB = 0, `estop_flag` | | images 29 à 38 |
| T8 Urgence et acquittement | | User ignoré, acquittement refusé puis accepté | | images 39 à 42 |
| T9 Buzzer | | durées des bips (`ticks_ms`) | | images 43 à 51 |
| T10 SysTick et robustesse | | `ticks_ms` sur 10 s et 60 s ; cas R1 à R6 | | images 52 à 56 |

## Mesures consignées

| Mesure | Exigence | Valeur mesurée | Écart | Cause de l'écart |
|---|---|---|---|---|
| Fréquence PWM | 20 kHz ± 5 % | | | |
| Rapport cyclique (25 / 50 / 75 / 100 %) | ± 2 points | | | |
| Rampe 0 → 100 % | 1 s ± 10 % | | | |
| Descente 100 → 25 % | 750 ms ± 10 % | | | |
| Fronts par seconde à 100 % | environ 4 400 | | | |
| Vitesse à 100 % | environ 348 tr/min | | | |
| Délai E-Stop vers PWM nulle | moins de 1 ms | | | |
| Bip court / double bip | 100 ms ± 10 % | | | |
| Période de SysTick | 1 ms ± 1 % | | | |

## Échéancier J1 : prévu et réel

| Laboratoire | Prévu (J1) | Réel | Écart et explication | Ajustement pour les laboratoires 3 à 5 |
|---|---|---|---|---|
| Laboratoire 1 | | | | |
| Laboratoire 2 | | | | |

## Fiche périphérique « Moteur, encodeurs et buzzer »

- Configuration (registres et valeurs) : voir section 2.3 du rapport
- Rôle dans le véhicule :
- Cas d'essai, résultats, trace : voir le tableau des essais
- Fiche « DEL et boutons » du laboratoire 1 jointe et mise à jour : oui 

## Déclaration des sources et de l'usage d'outils d'IA générative
- Sources: énoncé du laboratoire 2 (CEG3536, Automne 2026, version 1.3), complément de démarrage fourni sur Brightspace, code de départ du laboratoire 1, manuel de référence STM32L5 RM0438 et manuel de programmation Cortex-M PM0264

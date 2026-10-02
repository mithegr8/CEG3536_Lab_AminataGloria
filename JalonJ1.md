# JOURNAL — AminataGloria


## Membres et rôles
- Aminata : réalise (séance 0), valide (séance 1)
- Gloria : valide (séance 0), réalise (séance 1)
## Jalon J1 (au plus tard le vendredi 25 septembre 2026, validé dans Git)
## J1 — 25 septembre
- Exigences E1–E9 reformulées
- Échéancier labs 1–5 (dates du plan de cours)
- Répartition et rotation des rôles
## Séance 0 — 15/18 sept.
- Présents : Aminata, Gloria
- Fait : projet, LD3, débogueur
- Mesures : MODER = 0xABF7FFFF
- Problèmes : non clarte des instructions de lab


### Rôles et rotation
| Séance | Réalise | Valide (essais, mesures, relecture) |
|---|---|---|
| Séance 0 |Gloria | Aminata|
| Séance 1 |Aminata |Gloria |
| Séance 2 | | |

### Séance 1 — 25 Sept 2026 — réalise : Aminata / valide : Gloria
- Objectifs : E1, E2 et E3 démontrables
- Fait : E1 E2 E3 demontrables
- Décisions :completer button.s et fsm_step
- Difficultés et solutions : non compilation et non calibrage de delay_ms intructions non claires
- Essais et mesures : E1 E2 E3 : rouge-vert-rouge-bleu-rouge - > en 4 etapes
- Validations Git : soumis au TA

### Séance 2 — 2 Octobre — réalise : Gloria / valide : Aminata
- Objectifs : Implementer le E_stop et le boutton Touch
- Fait : Implementation, compilation et demonstration au TA
- Difficultés et solutions : NA
- Essais et mesures : clignotement de E_stop 2fois/seconde
- Validations Git : soumis au TA

## Tableau des essais (T1 à T10)
| Essai | Date | Résultat observé | Verdict | Preuve (fichier) |
|---|---|---|---|---|
| T1 Réinitialisation |25 Septembre 2026|rouge allumé apres reinitialisation | OK| fsm.s|
| T2 Cycle User | 25 Septembre 2026|rouge -vert-rouge-bleu-rouge |OK| fsm.s, buttons.s|
| T3 Anti-rebond |25 Septembre 2026 |un appui = un evenement|OK | button.s|
| T4 Niveaux logiques | 25 Septembre 2026| Verifié lors de l'implémentation des boutons|OK | buttons.s|
| T5 E-Stop |02 Octobre 2026 |E_stop avec rouge stationnaire en etat d'urgence | OK| estop.s, fsm.s|
| T6 Clignotement |02 Octobre 2026|Evolution de E_STOP rouge clignotant environ 2 fois/seconde | OK| fsm.s|
| T7 Acquittement | 02 Octobre 2026|Apres relachement de E_STOP,TOUCH En permet de revenir a l'arret normal |OK |fsm.s |
| T8 User ignoré en urgence |02 Octobre  2026| lorsque E_STOP est relache, B_USER est ignoré| OK|fsm_step |
| T9 Touch En hors urgence | 02 Octobre 2026|TOUCH En fonctionne hors urgence et declench l'extinction temporaire des DEL |OK|fsm.s |


## Routine conservée pour L3-A
- Routine : `button_pressed`
- Interface :prend en argument l'identifiant du bouton et retourne 1 lorsque l'appui  valide est detecté sinon 0
- Cas d'essai : apui repété et mainyien pour verifier l'anti rebond

## Déclaration des sources et de l'usage d'outils d'IA générative
- Sources : Notes de cours et lab et tutoriels
- Outils d'IA (outil, version, usage) ou « aucun usage » : 
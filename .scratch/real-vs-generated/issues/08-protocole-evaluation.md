# Quel protocole d'évaluation ?

Type: grilling
Status: open
Blocked by: 05

## Question

Fixer le protocole complet :
- proportions train / validation / test du split classique ;
- métriques : accuracy, F1, AUC, matrice de confusion, résultats par générateur ;
- déroulé exact de l'évaluation sur les 1 ou 2 générateurs mis de côté : ce qui est retiré de l'entraînement, ce qui est mesuré, ce qu'on compare.

## Comments

**2026-11-23, issu du ticket « Quel dataset et quel sous-ensemble ? »** : le protocole couvre deux expériences avec le même modèle, l'expérience « tous générateurs » (split classique) et l'expérience « générateurs mis de côté » (split classique, plus un test sur les 2 500 images de Midjourney et les 2 500 de BigGAN). Reste à fixer quelles images réelles accompagnent ce test sur générateurs jamais vus, et comment comparer les deux expériences.

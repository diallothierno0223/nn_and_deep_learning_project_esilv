# Quel protocole d'évaluation ?

Type: grilling
Status: resolved
Blocked by: 05

## Question

Fixer le protocole complet :
- proportions train / validation / test du split classique ;
- métriques : accuracy, F1, AUC, matrice de confusion, résultats par générateur ;
- déroulé exact de l'évaluation sur les 1 ou 2 générateurs mis de côté : ce qui est retiré de l'entraînement, ce qui est mesuré, ce qu'on compare.

## Comments

**2026-11-23, issu du ticket « Quel dataset et quel sous-ensemble ? »** : le protocole couvre deux expériences avec le même modèle, l'expérience « tous générateurs » (split classique) et l'expérience « générateurs mis de côté » (split classique, plus un test sur les 2 500 images de Midjourney et les 2 500 de BigGAN). Reste à fixer quelles images réelles accompagnent ce test sur générateurs jamais vus, et comment comparer les deux expériences.

## Answer

Le protocole s'applique aux deux expériences.

**Split** : 70 / 15 / 15 (train / validation / test), stratifié par classe et par générateur, graine fixe.

**Métriques** : accuracy, F1, AUC ROC, matrice de confusion, et taux de détection par générateur.

**Test sur générateurs mis de côté** (expérience « générateurs mis de côté ») : les 2 500 images Midjourney et les 2 500 BigGAN, face aux images réelles du jeu de test (~875). On rapporte le taux de détection par générateur mis de côté et l'AUC.

**Seuil de décision** : 0,5.

**Tableau final de comparaison**, une ligne par configuration :
- expérience « tous générateurs » → son jeu de test ;
- expérience « générateurs mis de côté » → son jeu de test ;
- expérience « générateurs mis de côté » → Midjourney ;
- expérience « générateurs mis de côté » → BigGAN.

S'y ajoutent les contrôles anti-raccourci du ticket prétraitement : scores par générateur et test de perturbation.

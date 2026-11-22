# Quel dataset et quel sous-ensemble ? Quels générateurs mettre de côté ?

Type: grilling
Status: claimed
Blocked by: 01, 02, 03, 04, 09

## Question

Choisir le dataset, puis le sous-ensemble utilisé (nombre d'images par classe, générateurs inclus) pour qu'il tienne dans le budget Kaggle/Colab. Choisir aussi les 1 ou 2 générateurs mis de côté pour l'évaluation sur des générateurs jamais vus.

## Comments

**2026-11-23, position de l'utilisateur** (pas encore la résolution) :
- Le dataset `cartografia/unbiased-tiny-genimage` convient à l'utilisateur, qui demande toutefois si un dataset débiaisé au contenu plus varié (paysages, personnes…) existe : voir le ticket « Existe-t-il un dataset débiaisé au contenu plus varié que GenImage ? ».
- Déséquilibre (5 828 réelles contre 17 500 générées) : plutôt que de pondérer la perte, sous-échantillonner les images générées pour obtenir à peu près autant d'images générées que de réelles, réparties équitablement entre les générateurs d'entraînement. Ordre de grandeur : avec 5 générateurs à l'entraînement (2 mis de côté), ~1 150 images par générateur.

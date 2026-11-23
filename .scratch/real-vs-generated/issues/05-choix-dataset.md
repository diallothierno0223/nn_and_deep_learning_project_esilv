# Quel dataset et quel sous-ensemble ? Quels générateurs mettre de côté ?

Type: grilling
Status: resolved
Blocked by: 01, 02, 03, 04, 09

## Question

Choisir le dataset, puis le sous-ensemble utilisé (nombre d'images par classe, générateurs inclus) pour qu'il tienne dans le budget Kaggle/Colab. Choisir aussi les 1 ou 2 générateurs mis de côté pour l'évaluation sur des générateurs jamais vus.

## Comments

**2026-11-23, position de l'utilisateur** (pas encore la résolution) :
- Le dataset `cartografia/unbiased-tiny-genimage` convient à l'utilisateur, qui demande toutefois si un dataset débiaisé au contenu plus varié (paysages, personnes…) existe : voir le ticket « Existe-t-il un dataset débiaisé au contenu plus varié que GenImage ? ».
- Déséquilibre (5 828 réelles contre 17 500 générées) : plutôt que de pondérer la perte, sous-échantillonner les images générées pour obtenir à peu près autant d'images générées que de réelles, réparties équitablement entre les générateurs d'entraînement. Ordre de grandeur : avec 5 générateurs à l'entraînement (2 mis de côté), ~1 150 images par générateur.

## Answer

**Dataset unique : `cartografia/unbiased-tiny-genimage`.** Tailles mesurées le 2026-11-23 :
- images réelles (`Nature`) : 5 828 images de ~450 à 500 px, variables ;
- BigGAN : 128 px ;
- ADM, GLIDE, VQDM : 256 px ;
- SD 1.5, Wukong : 512 px ;
- Midjourney : 1024 px.

**Deux expériences, avec exactement le même modèle** (choix de l'utilisateur) :

1. **Expérience « tous générateurs »** : les 7 générateurs mélangés. On garde les 5 828 images réelles et on tire environ 833 images par générateur (≈ 5 828 / 7), pour un jeu équilibré 50/50 avec la même part par générateur. Split classique train / validation / test. Elle mesure le score quand tous les générateurs sont vus à l'entraînement.
2. **Expérience « générateurs mis de côté »** : Midjourney et BigGAN sont exclus de l'entraînement. On garde les 5 828 images réelles et on tire environ 1 165 images de chacun des 5 autres générateurs (ADM, GLIDE, VQDM, SD 1.5, Wukong). Split classique train / validation / test. Le test sur générateurs jamais vus utilise les 2 500 images de Midjourney et les 2 500 de BigGAN. Elle mesure la généralisation à des générateurs jamais vus.

Midjourney et BigGAN ont été choisis comme générateurs mis de côté parce que ce sont les deux tailles extrêmes, et qu'ils représentent deux familles différentes : un GAN, et un générateur commercial récent.

Tous les tirages se font avec une graine fixe.

Conséquence pour le prétraitement : l'expérience « tous générateurs » met BigGAN (128 px) et Midjourney (1024 px) à l'entraînement. Le biais de taille doit donc être traité pour ces deux générateurs aussi : un crop de 224 px est impossible sur BigGAN.

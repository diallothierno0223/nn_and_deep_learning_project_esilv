# Quel prétraitement exact ?

Type: grilling
Status: claimed
Blocked by: 02, 05

## Question

Fixer la chaîne de prétraitement exacte appliquée aux deux classes :
- taille finale ;
- resize ou crop ;
- normalisation propre au backbone ;
- éventuelle recompression JPEG ;
- suppression des métadonnées.

Fixer aussi le contrôle qui prouve qu'aucun raccourci de format ne subsiste.

## Comments

**2026-11-23, contrainte issue du ticket « Quel dataset et quel sous-ensemble ? »** : l'expérience « tous générateurs » entraîne aussi sur BigGAN (128 px) et Midjourney (1024 px), alors que les réelles font ~450–500 px. Le prétraitement doit éviter un biais de taille pour toutes les tailles (128, 256, 512, 1024 px). Un crop de 224 px sans redimensionnement est impossible sur BigGAN : il faut trancher entre un crop commun plus petit (≤ 128 px), un redimensionnement préalable identique pour tous, du padding, ou autre chose. Le même prétraitement doit s'appliquer aux deux expériences.

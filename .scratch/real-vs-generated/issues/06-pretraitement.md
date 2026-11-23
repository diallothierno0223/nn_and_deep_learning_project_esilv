# Quel prétraitement exact ?

Type: grilling
Status: resolved
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

## Answer

Chaîne identique pour les deux classes et les deux expériences :
1. **Décodage** de l'image en pixels RGB. Les métadonnées disparaissent au décodage.
2. **Crop de 128 × 128 px à résolution native**, sans redimensionnement préalable : aléatoire à l'entraînement, centré en validation et en test. 128 px est la taille de BigGAN, la plus petite du dataset.
3. **Agrandissement du crop à 224 × 224 px** : même facteur ×1,75 pour toutes les images, donc pas de biais de redimensionnement entre classes.
4. **Normalisation** avec `keras.applications.resnet50.preprocess_input`.

Pas de recompression JPEG supplémentaire : le dataset est déjà tout en JPEG qualité 96. L'augmentation flou + JPEG aléatoire reste dans le brouillard (data augmentation).

Contrôles anti-raccourci, sans entraînement supplémentaire :
- **scores par générateur** : un écart extrême entre générateurs est suspect ;
- **test de perturbation**, dans l'esprit de BIAS-ID : appliquer la même recompression JPEG ou un léger flou aux images de test des deux classes, puis vérifier que les prédictions ne basculent pas vers une classe.

Contrepartie acceptée : sur Midjourney (1024 px), le crop ne couvre qu'une petite partie de l'image, ce qui rend Grad-CAM moins parlant.

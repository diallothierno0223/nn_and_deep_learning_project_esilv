# Quels backbones et quelle stratégie de fine-tuning ?

Type: grilling
Status: resolved
Blocked by: 03, 05

## Question

Choisir 1 à 3 backbones CNN de `keras.applications` et la stratégie de transfer learning :
- feature extraction seule, puis dégel partiel ou total ;
- nombre de couches dégelées ;
- tête de classification ;
- taux d'apprentissage pour chaque phase.

## Answer

Backbone : **ResNet50** pré-entraîné sur ImageNet (`keras.applications`), seul backbone du projet.

Le même modèle sert aux deux expériences.

- **Tête de classification** : GlobalAveragePooling2D → Dropout(0,3) → Dense(1, sigmoïde). Perte : entropie croisée binaire.
- **Phase 1** : backbone gelé, seule la tête est entraînée. Adam, lr = 1e-3, ~5 epochs.
- **Phase 2** : dégel du dernier bloc (`conv5`). Adam, lr = 1e-5, ~10 epochs, arrêt anticipé sur la perte de validation (en restaurant les meilleurs poids).
- **Détails** :
  - BatchNorm maintenues en mode inférence (`training=False`), comme le recommande Keras ;
  - batch de 64 ;
  - précision mixte.

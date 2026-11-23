# Document de cadrage : image réelle vs image générée

Projet du cours ESILV « Neural Networks & Deep Learning ». Tous les choix ci-dessous sont verrouillés : le notebook se code à partir de ce document, sans nouvelle décision. Chaque décision renvoie au ticket qui la justifie, dans `.scratch/real-vs-generated/`. Vocabulaire : [GLOSSARY.md](../GLOSSARY.md).

## 1. Sujet et contraintes

- **Tâche** : classification binaire d'une image en **image réelle** (photo prise par un appareil) ou **image générée** (entièrement produite par un modèle d'IA, quel qu'il soit).
- **Objectif** : reconnaître une image générée quel que soit son générateur, y compris un générateur jamais vu à l'entraînement.
- **Contraintes** : CNN + transfer learning obligatoires, TensorFlow / Keras, GPU gratuits Kaggle ou Colab.
- **Livrables** : un notebook, une présentation PowerPoint, un oral.
- **Hors périmètre** :
  - baseline entraîné from scratch ;
  - ViT ou CLIP à la place du CNN ;
  - deepfakes et images retouchées ;
  - art fait à la main ;
  - jeu de test complémentaire (Extended Synthbuster).

## 2. Dataset ([ticket](../.scratch/real-vs-generated/issues/05-choix-dataset.md))

**`cartografia/unbiased-tiny-genimage`** sur Kaggle (2,53 Go, CC BY-NC-SA 4.0, usage académique). C'est une version réduite de GenImage, déjà débiaisée côté format : tout est en JPEG qualité 96.

| Dossier | Classe | Images | Taille |
|---|---|---|---|
| `Nature` | réelle (ImageNet) | 5 828 | ~450–500 px, variable |
| `BigGAN` | générée | 2 500 | 128 × 128 |
| `ADM`, `glide`, `VQDM` | générée | 2 500 chacun | 256 × 256 |
| `stable_diffusion_v_1_5`, `wukong` | générée | 2 500 chacun | 512 × 512 |
| `Midjourney` | générée | 2 500 | 1024 × 1024 |

Pourquoi ce dataset : GenImage brut oppose des JPEG réels à des PNG générés, et le modèle apprend alors « JPEG = réel » ([ticket biais](../.scratch/real-vs-generated/issues/02-biais-de-format.md)). Aucun dataset plus varié n'est à la fois prêt à l'emploi et sans biais ([ticket](../.scratch/real-vs-generated/issues/09-dataset-plus-varie.md)).

## 3. Les deux expériences

Le même modèle et le même prétraitement servent aux deux expériences. Toutes deux sont **équilibrées 50/50** : on garde les 5 828 images réelles et on sous-échantillonne les images générées, à part égale par générateur, avec une graine fixe.

| | Expérience « tous générateurs » | Expérience « générateurs mis de côté » |
|---|---|---|
| Générateurs à l'entraînement | les 7 | ADM, GLIDE, VQDM, SD 1.5, Wukong |
| Images générées par générateur | ~833 | ~1 165 |
| Évaluation | split classique | split classique, plus test sur les 2 500 Midjourney et les 2 500 BigGAN |
| Question | quel score quand tous les générateurs sont connus ? | le modèle généralise-t-il à des générateurs jamais vus ? |

Midjourney et BigGAN sont mis de côté parce que ce sont les deux tailles extrêmes, et qu'ils représentent deux familles : un GAN et un générateur commercial récent.

## 4. Prétraitement ([ticket](../.scratch/real-vs-generated/issues/06-pretraitement.md))

La chaîne est identique pour toutes les images :

1. Décodage en pixels RGB. Les métadonnées disparaissent.
2. **Crop de 128 × 128 px à résolution native**, sans redimensionnement préalable : aléatoire à l'entraînement, centré en validation et en test.
3. **Agrandissement à 224 × 224 px**, avec le même facteur ×1,75 pour toutes les images.
4. `keras.applications.resnet50.preprocess_input`.

Pas de recompression systématique. Pourquoi : un redimensionnement différent selon la classe laisserait des traces que le modèle apprendrait à la place du vrai signal. Avec un crop natif puis un facteur unique, toutes les images subissent exactement la même transformation.

**Mise à jour après l'entraînement n° 1** ([rapport](rapport_train1.md)) : le modèle avait appris « net = réel ». On ajoute donc, à l'entraînement seulement et avant le crop, une **augmentation appliquée aux deux classes** : flou gaussien (probabilité 0,5, σ entre 0 et 2) et recompression JPEG (probabilité 0,5, qualité entre 50 et 100). Le test de perturbation est élargi à des dégradations plus fortes ou jamais vues : JPEG 40, flou σ = 2,5, réduction ×0,5 puis agrandissement.

## 5. Modèle et entraînement ([ticket](../.scratch/real-vs-generated/issues/07-backbones-finetuning.md))

- **Backbone** : ResNet50 pré-entraîné sur ImageNet (`include_top=False`).
- **Tête** : GlobalAveragePooling2D → Dropout(0,3) → Dense(1, sigmoïde). Perte : entropie croisée binaire.
- **Phase 1** : backbone gelé. Adam, lr = 1e-3, 10 epochs au maximum.
- **Phase 2** : dégel de `conv5`. Adam, lr = 1e-5, 30 epochs au maximum.
- **Arrêt anticipé sur les deux phases** (patience 4, restauration des meilleurs poids). Entraînement n° 1 : 5 + 10 epochs, arrêt anticipé en phase 2 seulement.
- **Détails** :
  - BatchNorm en mode inférence (`training=False`) ;
  - batch de 64 ;
  - précision mixte ;
  - graine fixe.
- **Budget** : pas de recherche d'hyperparamètres. Une exécution par expérience, soit **2 entraînements**. Les poids sont sauvegardés après chaque entraînement.

## 6. Évaluation ([ticket](../.scratch/real-vs-generated/issues/08-protocole-evaluation.md))

- **Split** : 70 / 15 / 15, stratifié par classe et par générateur, graine fixe.
- **Métriques** : accuracy, F1, AUC ROC, matrice de confusion, taux de détection par générateur. Seuil de décision : 0,5.
- **Test sur générateurs mis de côté** : les 2 500 Midjourney et les 2 500 BigGAN, face aux images réelles du jeu de test (~875). On rapporte le taux de détection par générateur et l'AUC.
- **Tableau final**, une ligne par configuration :
  - « tous générateurs » → son jeu de test ;
  - « générateurs mis de côté » → son jeu de test ;
  - « générateurs mis de côté » → Midjourney ;
  - « générateurs mis de côté » → BigGAN.
- **Contrôles anti-raccourci** :
  - scores par générateur : un écart extrême est suspect ;
  - test de perturbation : même recompression JPEG ou léger flou appliqués aux deux classes du test, en vérifiant que les prédictions ne basculent pas vers une classe.

## 7. Explicabilité ([ticket](../.scratch/real-vs-generated/issues/10-brouillard-restant.md))

Grad-CAM sur le modèle de l'expérience « générateurs mis de côté », couche `conv5_block3_out`. Environ 8 exemples :
- 2 images réelles ;
- 2 images générées bien classées ;
- 2 erreurs ;
- 2 images Midjourney ou BigGAN.

## 8. Rédaction du notebook

Le notebook s'exécute tel quel sur Colab ou Kaggle, de haut en bas, sans manipulation manuelle. Règles :
- chaque section commence par une cellule markdown avec un titre ;
- chaque fonction a une docstring ;
- les lignes longues ou peu évidentes sont commentées.

Sections, dans cet ordre :

1. Installation des packages
2. Imports
3. Téléchargement du dataset
4. Construction des sous-ensembles équilibrés et du split
5. Prétraitement (pipeline `tf.data`)
6. Construction du modèle
7. Expérience « tous générateurs » : entraînement
8. Expérience « générateurs mis de côté » : entraînement
9. Évaluation et tableau comparatif
10. Contrôles anti-raccourci
11. Grad-CAM
12. Conclusion et limites

## 9. Fil conducteur de l'oral

1. Le problème.
2. Le piège des biais (JPEG, taille) et comment on l'évite.
3. La méthode.
4. Les résultats des deux expériences.
5. Les contrôles anti-raccourci.
6. Grad-CAM.
7. Les limites et les pistes : plusieurs crops par image en évaluation, autres backbones, jeu de test au contenu varié.

Le détail des slides se fera une fois les résultats obtenus.

## 10. Reste à compléter

- Équipe et échéance.

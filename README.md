# Image réelle vs image générée : détection par CNN et transfer learning

Projet du cours ESILV « Neural Networks & Deep Learning ». Ce fichier résume tout le cadrage du projet : le sujet, les choix verrouillés, ce que la recherche préalable a appris, et la suite. Le notebook n'est pas encore écrit : il se codera à partir de ce document.

---

## 1. Le sujet

- **Tâche** : classer une image en **image réelle** (photo d'une scène qui existe, prise avec un appareil photo ou un téléphone) ou en **image générée** (entièrement produite par un modèle d'IA : Stable Diffusion, Midjourney, GAN…).
- **Objectif** : reconnaître une image générée **quel que soit le générateur**, y compris un générateur que le modèle n'a jamais vu à l'entraînement.
- **Contraintes du cours** : CNN et transfer learning obligatoires. On peut ajouter d'autres techniques, mais jamais remplacer le CNN (pas de Vision Transformer à la place).
- **Framework** : TensorFlow / Keras.
- **Calcul** : GPU gratuits de Google Colab ou Kaggle.
- **Livrables** : un notebook, une présentation PowerPoint, un oral.

### Vocabulaire

| Terme | Sens |
|---|---|
| **Image réelle** | Photo prise par une personne avec un appareil photo ou un téléphone. Éviter « image humaine » : un dessin fait à la main n'en est pas une. |
| **Image générée** | Image entièrement produite par un modèle d'IA, quel qu'il soit. Éviter « fake » : ça évoque les deepfakes et les retouches, hors sujet ici. |
| **Générateur** | Le modèle d'IA qui a produit une image générée (ex. Stable Diffusion 1.5, Midjourney v5). |
| **Générateur mis de côté** | Générateur dont aucune image n'est vue à l'entraînement. On s'en sert uniquement pour tester la généralisation (ici Midjourney et BigGAN). |

### Hors périmètre

- Un CNN entraîné from scratch pour comparer : l'intérêt du transfer learning se justifie à l'oral (gain de temps, meilleure précision) sans expérience dédiée.
- Remplacer le CNN par un ViT ou CLIP : interdit par le sujet.
- Les deepfakes, images retouchées ou montages : seules les images entièrement générées comptent.
- L'art et les dessins faits à la main : la classe réelle ne contient que des photos.
- Un jeu de test au contenu plus varié (Extended Synthbuster) : on s'en tient à GenImage seul.

---

## 2. Ce que la recherche préalable a appris

### Le piège principal : les biais de format et de taille

GenImage est le dataset de référence pour cette tâche, mais il contient un piège bien documenté (Grommelt et al., *Fake or JPEG?*, arXiv 2403.17608) :

- Les **images réelles** sont des JPEG d'ImageNet (qualité ~96) de tailles variables. Les **images générées** sont des PNG non compressés, à une taille fixe par générateur.
- Un modèle apprend alors « **JPEG = réel** » au lieu d'apprendre « photo = réel ». Exemple : un ResNet50 entraîné sur GenImage classe correctement 80 % de photos réelles en PNG… et 100 % des mêmes photos une fois converties en JPEG.
- Le modèle apprend aussi la **taille** et les **traces de redimensionnement**. Redimensionner tout le monde à la même taille ne suffit pas : chaque classe est agrandie ou réduite d'un facteur différent, et ces différences laissent des traces exploitables (Rajan et al., arXiv 2410.11835).
- Corriger ces biais fait gagner **environ 11 points** de précision sur des générateurs jamais vus (ResNet50 : de 71,7 % à 82,7 %).

### Les datasets étudiés

| Dataset | Verdict |
|---|---|
| **GenImage** complet (NeurIPS 2023) | Environ 500 Go, 8 générateurs. Trop gros pour Kaggle ou Colab en entier. |
| **unbiased-tiny-genimage** (Kaggle) | **Retenu.** 2,5 Go, déjà débiaisé côté format : tout est en JPEG qualité 96. |
| tiny-genimage (Kaggle, 8,4 Go) | Pas débiaisé, il faudrait le faire nous-mêmes. |
| Extended Synthbuster (Kaggle, 5,2 Go) | Contenu varié (paysages, personnes) face à FLUX et SD 3.5. Envisagé comme jeu de test complémentaire, finalement écarté. |
| OpenFake | Seule option variée pour l'entraînement, mais à construire et débiaiser soi-même. Trop risqué. |
| WildRF, Chameleon, DRCT-2M, ELSA D3, Semi-Truths, ImagiNet, COCO_AI, Community Forensics | Écartés : biais de format, accès restreint, pas sur Kaggle, images retouchées ou taille énorme. |

### Les backbones CNN

- Dans `keras.applications`, les candidats sérieux sont ResNet50 (la référence de la littérature, Wang et al. 2020), EfficientNetB0 / V2B0 et ConvNeXtTiny.
- **Le prétraitement compte plus que le choix du backbone** : sur GenImage, changer de backbone rapporte moins que de supprimer les biais.
- Coût estimé : sur un GPU T4 en précision mixte, une epoch de 10 000 images prend entre 30 secondes et 2 minutes avec ResNet50. Kaggle offre 30 h de GPU par semaine et des sessions de 12 h. Le calcul n'est pas un problème.

Les notes détaillées, avec toutes les sources, sont dans [.scratch/real-vs-generated/research/](.scratch/real-vs-generated/research/). Celles des biais de format et des backbones sont sur les branches `research/02-biais-de-format` et `research/03-backbones-keras`.

---

## 3. Les choix verrouillés

### Dataset : `cartografia/unbiased-tiny-genimage`

Lien : https://www.kaggle.com/datasets/cartografia/unbiased-tiny-genimage (licence CC BY-NC-SA 4.0, usage académique).

| Dossier | Classe | Images | Taille |
|---|---|---|---|
| `Nature` | réelle (ImageNet) | 5 828 | ~450–500 px, variable |
| `BigGAN` | générée | 2 500 | 128 × 128 |
| `ADM`, `glide`, `VQDM` | générée | 2 500 chacun | 256 × 256 |
| `stable_diffusion_v_1_5`, `wukong` | générée | 2 500 chacun | 512 × 512 |
| `Midjourney` | générée | 2 500 | 1024 × 1024 |

Les deux classes montrent les mêmes sujets ImageNet (animaux, objets) : le contenu ne trahit donc pas la classe. Aucun split train/test n'est fourni.

### Deux expériences, avec exactement le même modèle

Les deux sont **équilibrées 50/50**. On garde les 5 828 images réelles, et on sous-échantillonne les images générées à part égale par générateur, avec une graine fixe. Pas de pondération de la perte.

| | Expérience « tous générateurs » | Expérience « générateurs mis de côté » |
|---|---|---|
| Générateurs à l'entraînement | les 7 | ADM, GLIDE, VQDM, SD 1.5, Wukong |
| Images générées par générateur | ~833 | ~1 165 |
| Évaluation | split classique | split classique, plus test sur les 2 500 Midjourney et les 2 500 BigGAN |
| Question posée | quel score quand tous les générateurs sont connus ? | le modèle reconnaît-il des générateurs jamais vus ? |

Pourquoi Midjourney et BigGAN sont mis de côté :
- ce sont les deux tailles extrêmes ;
- ils représentent deux familles différentes : un GAN, et un générateur commercial récent.

### Prétraitement : sans biais de taille

La même chaîne s'applique à toutes les images, dans les deux expériences :

1. **Décodage** en pixels RGB. Les métadonnées disparaissent.
2. **Crop de 128 × 128 px à résolution native**, sans redimensionnement préalable. Le crop est aléatoire à l'entraînement et centré en validation et en test. 128 px est la taille de BigGAN, la plus petite du dataset.
3. **Agrandissement à 224 × 224 px**, avec le **même facteur ×1,75 pour toutes les images**. Aucune classe n'est donc déformée différemment des autres.
4. **Normalisation** avec `keras.applications.resnet50.preprocess_input`.

On ne recompresse rien de façon systématique : le dataset est déjà tout en JPEG qualité 96. Contrepartie acceptée : sur Midjourney (1024 px), le crop ne couvre qu'un petit morceau de l'image.

**Augmentation, ajoutée pour l'entraînement n° 2** (voir [docs/rapport_train1.md](docs/rapport_train1.md)) : à l'entraînement seulement, avant le crop, chaque image, réelle ou générée, reçoit au hasard un flou gaussien (probabilité 0,5, σ entre 0 et 2) et une recompression JPEG (probabilité 0,5, qualité entre 50 et 100). Le premier entraînement avait appris « net = réel » : ~40 % des vraies photos floutées ou recompressées étaient prises pour des images générées.

### Modèle et entraînement

- **Backbone** : ResNet50 pré-entraîné sur ImageNet (`include_top=False`).
- **Tête** : GlobalAveragePooling2D → Dropout(0,3) → Dense(1, sigmoïde). Perte : entropie croisée binaire.
- **Phase 1**, backbone gelé : seule la tête apprend. Adam, lr = 1e-3, 10 epochs au maximum.
- **Phase 2**, dégel du dernier bloc (`conv5`) : Adam, lr = 1e-5, 30 epochs au maximum.
- **Arrêt anticipé sur les deux phases** : arrêt après 4 epochs sans amélioration de la perte de validation, avec restauration des poids de la meilleure epoch. (Entraînement n° 1 : 5 + 10 epochs, arrêt anticipé en phase 2 seulement.)
- **Détails** :
  - BatchNorm en mode inférence (`training=False`) ;
  - batch de 64 ;
  - précision mixte ;
  - graine fixe.
- **Budget** : pas de recherche d'hyperparamètres. Une exécution par expérience, soit **2 entraînements au total**. Les poids sont sauvegardés pour pouvoir réévaluer sans réentraîner.

### Évaluation

- **Split** : 70 / 15 / 15 (train / validation / test), stratifié par classe et par générateur.
- **Métriques** : accuracy, F1, AUC ROC, matrice de confusion, taux de détection par générateur. Seuil de décision : 0,5.
- **Évaluation multi-crops (ajoutée pour l'entraînement n° 2)** : chaque image de test est aussi évaluée sur une grille de 3 × 3 crops de 128 px, à résolution native. La prédiction est la moyenne des 9 probabilités.
  - Pourquoi : un seul crop ne voit que 1,6 % d'une image Midjourney (1024 px) et peut tomber sur une zone vide. La grille en voit 14 %.
  - La procédure est la même pour toutes les images, donc pas de biais de taille.
  - Les résultats sont donnés côte à côte : 1 crop centré et grille 3 × 3.
  - Le test de perturbation et Grad-CAM restent sur 1 crop, pour rester comparables avec l'entraînement n° 1.
- **Test sur générateurs mis de côté** : les 2 500 Midjourney et les 2 500 BigGAN, face aux images réelles du jeu de test (~875). On donne le taux de détection de chaque générateur et l'AUC.
- **Tableau final** :

| Modèle | Évalué sur |
|---|---|
| Expérience « tous générateurs » | son jeu de test |
| Expérience « générateurs mis de côté » | son jeu de test |
| Expérience « générateurs mis de côté » | Midjourney (jamais vu) |
| Expérience « générateurs mis de côté » | BigGAN (jamais vu) |

### Contrôles anti-triche

Ces contrôles prouvent que le modèle n'a pas appris un raccourci. Ils ne demandent aucun entraînement supplémentaire.

- **Scores par générateur** : un générateur détecté à 100 % quand les autres plafonnent à 60 % serait suspect.
- **Test de perturbation**, dans l'esprit du protocole BIAS-ID : on applique la même recompression JPEG ou un léger flou aux images de test **des deux classes**. On vérifie ensuite que les prédictions ne basculent pas toutes vers une classe.

### Explicabilité : Grad-CAM

Sur le modèle de l'expérience « générateurs mis de côté », couche `conv5_block3_out`. Environ 8 exemples :
- 2 images réelles ;
- 2 images générées bien classées ;
- 2 erreurs ;
- 2 images Midjourney ou BigGAN.

---

## 4. Le notebook à écrire

### Règles de rédaction

1. Le notebook s'exécute **tel quel sur Colab ou Kaggle**, de haut en bas, sans manipulation manuelle.
2. En tête, dans cet ordre : **toutes les installations** de packages, puis **tous les imports**, puis **tous les téléchargements** (dataset, poids).
3. Chaque section de code est précédée d'une **cellule markdown avec un titre**.
4. Chaque fonction a une **docstring**.
5. Les lignes longues ou peu évidentes sont **commentées**.

### Gestion des paquets

- **En local**, les paquets s'installent avec **uv** (`uv add <paquet>`), dans l'environnement `.venv` à la racine du projet. Chaque dépendance est ainsi tracée dans `pyproject.toml` et `uv.lock`. Pas de `pip install` direct.
- **Sur Colab ou Kaggle**, l'environnement local n'existe pas. La première cellule du notebook installe donc elle-même ce qui manque, pour que le notebook reste exécutable tel quel. Les mêmes paquets doivent aussi figurer dans `pyproject.toml`.

### Lancer le notebook

Le notebook est [notebooks/real_vs_generated.ipynb](notebooks/real_vs_generated.ipynb).

- **Sur Colab** : activer un GPU (Exécution → Modifier le type d'exécution), puis tout exécuter. Le dataset se télécharge avec `kagglehub`. Sur Colab gratuit (~12 Go de RAM), mettre `CACHE_DECODED_IMAGES = False` dans la section 4.
- **Vitesse** : le goulot d'étranglement est le CPU, pas le GPU. Le notebook garde donc les images décodées en mémoire (~15 Go) et n'applique le flou et le JPEG qu'à une fenêtre autour du crop. Ces deux optimisations donnent exactement les mêmes pixels que la version lente (vérifié pixel par pixel).
- **Sur Kaggle** : ajouter le dataset `cartografia/unbiased-tiny-genimage` au notebook (« Add Input »), activer un GPU, puis tout exécuter.
- **Test rapide** : avec la variable d'environnement `SMOKE_TEST=1` (ou `SMOKE_TEST = True` dans la section 3), le notebook tourne en version miniature : 60 images par dossier, 1 epoch par phase, sans poids pré-entraînés. Ce mode sert uniquement à vérifier que tout s'exécute.
- Les modèles entraînés et leurs historiques sont sauvegardés dans `outputs/`. Si un modèle y existe déjà, il est rechargé au lieu d'être réentraîné.
- En local sous Windows, TensorFlow n'utilise pas le GPU (il faut WSL2). Les vrais entraînements se font donc sur Colab ou Kaggle.

### Sections

1. Installation des packages
2. Imports
3. Téléchargement du dataset
4. Construction des sous-ensembles équilibrés et du split
5. Prétraitement (pipeline `tf.data`)
6. Construction du modèle
7. Expérience « tous générateurs » : entraînement
8. Expérience « générateurs mis de côté » : entraînement
9. Évaluation et tableau comparatif
10. Contrôles anti-triche
11. Grad-CAM
12. Conclusion et limites

---

## 5. Le fil conducteur de l'oral

1. **Le problème** : distinguer une photo d'une image générée, quel que soit le générateur.
2. **Le piège** : les biais JPEG et taille de GenImage, et comment on les évite.
3. **La méthode** : dataset débiaisé, crop natif, ResNet50 en transfer learning.
4. **Les résultats** des deux expériences : ce qu'on perd quand le générateur est inconnu.
5. **Les contrôles anti-triche** : la preuve que le modèle ne triche pas.
6. **Grad-CAM** : où le modèle regarde.
7. **Les limites et les pistes** : plusieurs crops par image en évaluation, autres backbones, jeu de test au contenu varié.

Le détail des slides se fera une fois les résultats obtenus.

---

## 6. Où trouver le détail

| Fichier | Contenu |
|---|---|
| [docs/cadrage.md](docs/cadrage.md) | Le document de cadrage : toutes les décisions, avec un lien vers le ticket qui justifie chacune. |
| [GLOSSARY.md](GLOSSARY.md) | Le vocabulaire du projet. |
| [.scratch/real-vs-generated/map.md](.scratch/real-vs-generated/map.md) | La carte de planification : destination, notes, index des décisions, hors périmètre. |
| [.scratch/real-vs-generated/issues/](.scratch/real-vs-generated/issues/) | Les 10 tickets de décision, chacun avec sa question et sa réponse. |
| [.scratch/real-vs-generated/research/](.scratch/real-vs-generated/research/) | Les notes de recherche détaillées et sourcées. |

## 7. Reste à faire

- [x] Écrire le notebook : [notebooks/real_vs_generated.ipynb](notebooks/real_vs_generated.ipynb) (testé de bout en bout en mode miniature, pas encore entraîné pour de vrai).
- [ ] Lancer les 2 entraînements sur Colab ou Kaggle.
- [ ] Préparer les slides à partir des résultats.
- [ ] Compléter l'équipe et l'échéance.

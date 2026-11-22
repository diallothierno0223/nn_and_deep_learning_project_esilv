# Carte : classification image réelle vs image générée

Label: wayfinder:map

## Destination

Un document de cadrage où tous les choix sont verrouillés (dataset et sous-ensemble, prétraitement, backbones CNN et fine-tuning, protocole d'évaluation). Il doit être assez précis pour coder le notebook TensorFlow/Keras sans rien décider de plus, avec un coût de calcul qui tient sur Kaggle/Colab.

## Notes

- Domaine : projet de cours ESILV « Neural Networks & Deep Learning ». Livrables : un notebook, une présentation PowerPoint, un oral.
- Tâche : classification binaire **image réelle** vs **image générée** (voir [GLOSSARY.md](../../GLOSSARY.md)). Le générateur d'origine n'a pas d'importance pour la décision.
- Contraintes imposées : CNN + transfer learning obligatoires. On peut ajouter d'autres choses par-dessus, mais jamais remplacer le CNN (pas de ViT à la place).
- Framework : TensorFlow / Keras (celui du cours).
- Calcul : GPU gratuits Kaggle et Google Colab uniquement.
- Évaluation déjà actée : split classique sur un mélange de générateurs, plus une évaluation sur 1 à 2 générateurs mis de côté (jamais vus à l'entraînement), pas davantage à cause du coût.
- Prétraitement acté dans le principe : toutes les images à la même taille, seulement les pixels (aucune extension ni métadonnée exploitable).
- Grad-CAM : à faire, en bonus.
- Pas de code ni d'entraînement avant d'avoir atteint la destination : on économise le calcul.
- Rédaction du notebook (préférence permanente) :
  1. Le notebook s'exécute tel quel sur Colab ou Kaggle, de haut en bas, sans manipulation manuelle.
  2. Ordre fixe en tête : d'abord toutes les installations de packages, puis tous les imports, puis tous les téléchargements (dataset, poids…).
  3. Chaque section de code est précédée d'une cellule markdown avec un titre (ex. « Mise en place », « Entraînement »…).
  4. Chaque fonction a une docstring.
  5. Les lignes longues ou peu évidentes sont commentées.
- Choix annoncés par l'utilisateur (2026-11-23), à confirmer dans leurs tickets : dataset `cartografia/unbiased-tiny-genimage` (2,5 Go, déjà débiaisé) ; backbone ResNet50. Le calcul gratuit (Colab/Kaggle) est jugé suffisant.
- Skills à consulter : grilling + domain-modeling pour les tickets de discussion ; research pour les tickets de recherche.
- Équipe et échéance : à compléter plus tard.

## Decisions so far

- [Inspecter un échantillon des datasets candidats](issues/04-inspecter-echantillon.md) : unbiased-tiny-genimage est validé à l'œil (5 828 réelles, 7 × 2 500 générées, tout en JPEG, pas de split fourni) ; un biais de taille entre générateurs reste à traiter.

- [Existe-t-il un dataset débiaisé au contenu plus varié que GenImage ?](issues/09-dataset-plus-varie.md) : non, rien de prêt à l'emploi. On garde unbiased-tiny-genimage pour l'entraînement ; Extended Synthbuster (photos RAISE variées contre FLUX et SD 3.5, 5,2 Go sur Kaggle) est le candidat comme jeu de test complémentaire.

- [Quels backbones CNN Keras conviennent, et combien coûte leur fine-tuning ?](issues/03-backbones-keras.md) : candidats ResNet50, EfficientNet(V2)B0 et ConvNeXtTiny. Une epoch de 100k images prend environ 5 à 10 min sur un T4 (estimation). Kaggle offre 30 h GPU/semaine. Le prétraitement compte plus que le choix du backbone.

- [Quels datasets image réelle vs image générée existent ?](issues/01-datasets-candidats.md) : GenImage (8 générateurs, ~500 Go) via un miroir Kaggle réduit, de préférence la version déjà débiaisée unbiased-tiny-genimage (2,5 Go). Synthbuster peut servir de test hors distribution ; les autres datasets sont trop gros ou difficiles d'accès.

- [Quels biais de format piègent ces datasets, et quel prétraitement les neutralise ?](issues/02-biais-de-format.md) : GenImage oppose des JPEG réels à des PNG générés de taille fixe. Il faut recompresser les images générées en JPEG, aligner les tailles, cropper plutôt que resizer, puis vérifier qu'aucun raccourci ne subsiste.

## Not yet specified

- Grad-CAM : sur quel(s) modèle(s), quelles couches, combien d'exemples montrer, et comment les présenter à l'oral.
- Data augmentation : laquelle est sûre pour cette tâche (certaines transformations peuvent effacer les traces de génération, ou en créer).
- Budget d'hyperparamètres : combien d'essais le calcul Kaggle/Colab permet, et lesquels.
- Structure du notebook : la liste des sections après le bloc installations / imports / téléchargements (les conventions de rédaction sont dans les Notes), et ce qui est ré-entraîné ou chargé depuis des poids sauvegardés.
- Plan des slides et de l'oral : quelle histoire raconter, quels résultats mettre en avant.
- Équipe et échéance.

## Out of scope

- Baseline CNN entraîné from scratch : l'apport du transfer learning se justifie à l'oral sans expérience dédiée.
- Remplacer le CNN par un Vision Transformer ou CLIP : interdit par le sujet.
- Deepfakes, images retouchées ou montages : seules les images entièrement générées comptent.
- Art ou dessins faits à la main : la classe réelle ne contient que des photos.
- Jeu de test complémentaire au contenu varié (Extended Synthbuster, voir [le ticket sur un dataset plus varié](issues/09-dataset-plus-varie.md)) : l'utilisateur préfère s'en tenir à GenImage seul.

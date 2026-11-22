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
- Skills à consulter : grilling + domain-modeling pour les tickets de discussion ; research pour les tickets de recherche.
- Équipe et échéance : à compléter plus tard.

## Decisions so far

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

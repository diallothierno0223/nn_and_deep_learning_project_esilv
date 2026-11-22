# Quels backbones CNN Keras conviennent, et combien coûte leur fine-tuning ?

Type: research
Status: open
Blocked by: —

## Question

Parmi les CNN pré-entraînés disponibles dans `keras.applications` (ResNet, EfficientNet/EfficientNetV2, ConvNeXt, Xception…), lesquels sont pertinents pour détecter des images générées ? Que rapporte la littérature sur la détection d'images générées par des CNN (par exemple les travaux de Wang et al. 2020 avec ResNet-50) ?

Pour chaque backbone retenu, donner :
- le nombre de paramètres ;
- la résolution d'entrée native ;
- une estimation du temps par epoch et de la mémoire GPU pour un fine-tuning sur un GPU Kaggle (P100 / T4) ou Colab (T4), avec un dataset de 10k à 100k images.

Rappeler aussi les quotas et limites actuels de Kaggle et Colab : heures GPU par semaine, durée maximale d'une session, disque.

## Recherche

En cours : les résultats arrivent sur la branche jetable `research/03-backbones-keras`, dans le fichier `research/03-backbones-keras.md` (jamais mergée).

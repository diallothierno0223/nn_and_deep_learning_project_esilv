# Quels backbones CNN Keras conviennent, et combien coûte leur fine-tuning ?

Type: research
Status: resolved
Blocked by: —

## Question

Parmi les CNN pré-entraînés disponibles dans `keras.applications` (ResNet, EfficientNet/EfficientNetV2, ConvNeXt, Xception…), lesquels sont pertinents pour détecter des images générées ? Que rapporte la littérature sur la détection d'images générées par des CNN (par exemple les travaux de Wang et al. 2020 avec ResNet-50) ?

Pour chaque backbone retenu, donner :
- le nombre de paramètres ;
- la résolution d'entrée native ;
- une estimation du temps par epoch et de la mémoire GPU pour un fine-tuning sur un GPU Kaggle (P100 / T4) ou Colab (T4), avec un dataset de 10k à 100k images.

Rappeler aussi les quotas et limites actuels de Kaggle et Colab : heures GPU par semaine, durée maximale d'une session, disque.

## Recherche

Détail et sources sur la branche jetable `research/03-backbones-keras`, dans le fichier `research/03-backbones-keras.md` (jamais mergée).

## Answer

Le backbone compte moins que le prétraitement et le protocole : sur GenImage, ResNet-50 plafonne à ~72 % en cross-générateur, et le débiaisage JPEG/taille rapporte plus de 11 points. La référence reste Wang et al. 2020 (ResNet-50 pré-entraîné ImageNet + augmentation flou/JPEG), mais elle n'a jamais été testée sur des images de diffusion.

Backbones retenus dans `keras.applications` :
- **ResNet50** : référence de la littérature ;
- **EfficientNetB0 / EfficientNetV2B0** : options légères (mais pas plus rapides que ResNet50 sur GPU) ;
- **ConvNeXtTiny** : CNN moderne.
Optionnels, 2 à 3 fois plus chers : Xception, EfficientNetB3, EfficientNetV2S.

Coût estimé (extrapolé des chiffres NVIDIA V100, aucune mesure T4/P100 publiée) : sur un T4 en précision mixte, une epoch de 100k images prend ~5–10 min pour ResNet50, EfficientNetB0 ou ConvNeXtTiny, ~10–20 min pour les plus gros ; une epoch de 10k images prend ~0,5–2 min. Batch de 64–128 sur 16 Go. Les 4 cœurs CPU de Kaggle risquent d'être le goulot : prétraiter les crops une fois et les mettre en cache.

Limites officielles Kaggle : 30 h GPU/semaine, sessions de 12 h, 20 Go sauvegardés dans `/kaggle/working`, un P100 ou deux T4, 29 Go de RAM. Colab gratuit : aucun quota publié, sessions jusqu'à 12 h « selon disponibilité » ; T4 et ~70–110 Go de disque non vérifiés.

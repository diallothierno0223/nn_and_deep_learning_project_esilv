# Quels biais de format piègent ces datasets, et quel prétraitement les neutralise ?

Type: research
Status: resolved
Blocked by: —

## Question

Dans les datasets de détection d'images générées (GenImage en particulier), quels biais non sémantiques distinguent les deux classes ? On pense au format de fichier (JPEG vs PNG), aux traces de compression JPEG, à la résolution et au ratio, aux métadonnées. Quels travaux montrent qu'un classifieur exploite ces raccourcis ?

Le principe déjà acté (même taille pour toutes les images, uniquement les pixels) suffit-il ? Les traces de compression JPEG survivent dans les pixels même après redimensionnement. Quels prétraitements la littérature recommande-t-elle (recompression JPEG des deux classes, crop au lieu de resize, alignement de résolution…) ? Comment vérifier après coup qu'il ne reste plus de raccourci ?

## Recherche

Résultats sur la branche jetable `research/02-biais-de-format`, dans le fichier `research/02-biais-de-format.md` (jamais mergée).

## Answer

Oui, GenImage est piégé, et « même taille + pixels seuls » ne suffit pas. Les images réelles sont des JPEG ImageNet (qualité ~96, tailles variables, ~450 px) ; les images générées sont des PNG non compressés, à taille fixe par générateur. Un ResNet50 apprend « JPEG = réel » et « taille ≈ celle du générateur = généré ». Les traces de compression survivent dans les pixels décodés, et un resize commun crée un nouveau biais (facteurs d'échelle différents par classe). Retirer ces biais fait gagner ~11 points en généralisation cross-générateur.

Prétraitement recommandé : réenregistrer les images générées en JPEG qualité 96 ; ne garder que des générateurs d'une même taille (ex. le trio 512 px SD1.4 / SD1.5 / Wukong) et des images réelles entre 450 et 550 px ; crop au lieu de resize ; augmentation flou + JPEG aléatoire sur les deux classes. Contrôles après coup : protocole BIAS-ID, test PNG vs JPEG sur des photos réelles non compressées, petit classifieur « trivial » sur les seules caractéristiques de format et de taille (suggestion non issue d'un article).

Points non vérifiés : survie mesurée des traces JPEG après resize, biais EXIF ou de ratio, risque de double compression.

Détail et sources : branche `research/02-biais-de-format`, fichier `research/02-biais-de-format.md` (commit 838230b).

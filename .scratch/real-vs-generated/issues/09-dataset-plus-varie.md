# Existe-t-il un dataset débiaisé au contenu plus varié que GenImage ?

Type: research
Status: resolved
Blocked by: —

## Question

`cartografia/unbiased-tiny-genimage` convient, mais son contenu est celui d'ImageNet : surtout des animaux et des objets. Existe-t-il un dataset image réelle vs image générée, multi-générateurs, **sans biais de format ni de taille** (ou facile à débiaiser nous-mêmes), dont le contenu est plus varié : paysages, personnes, scènes urbaines, intérieurs… ?

Pour chaque candidat : contenu des images réelles et générées, générateurs, nombre d'images par classe (équilibre), formats et tailles par classe, taille totale, accès depuis Kaggle/Colab, licence. Comparer avec `unbiased-tiny-genimage` et dire si l'un d'eux vaut le changement, ou s'il peut servir de jeu de test complémentaire.

## Answer

Non : aucun dataset prêt à l'emploi ne combine à la fois un contenu varié, plusieurs générateurs, aucun biais de format ou de taille, une petite taille et une copie Kaggle. On garde `unbiased-tiny-genimage` pour l'entraînement.

Pour le contenu varié, on peut ajouter un **jeu de test complémentaire : Extended Synthbuster** (issu de B-Free).
- **Images réelles** : 1 000 photos RAISE (paysages, intérieurs, personnes, bâtiments, nature).
- **Images générées** : 1 000 FLUX et 1 000 SD 3.5, produites à partir des légendes des photos réelles, donc au contenu apparié. S'y ajoutent 1 000 images LDM, à écarter ou à traiter à part.
- **Accès** : copie Kaggle `ngovanphuong2604/bfree-extended-synthbuster`, 5,2 Go, tout en PNG.
- **Usage** : appliquer aux deux classes le même traitement qu'à l'entraînement (crop/resize puis JPEG qualité 96).

Autres pistes :
- **Synthbuster original** (9 générateurs, 12,4 Go + 22,9 Go de TIFF RAISE) : possible mais lourd.
- **OpenFake** : seule option réaliste pour *entraîner* sur du contenu varié (personnes, actualités), mais il faut le construire et le débiaiser soi-même. Le sous-ensemble Kaggle de 20k images a probablement un biais de taille.
- **Écartés** : WildRF (biais de format), Chameleon (accès sur demande), DRCT-2M (ModelScope uniquement), ELSA D3 (des URLs seulement), Semi-Truths (images retouchées), ImagiNet (144 Go, accès restreint), COCO_AI (biais de taille).

Les copies Kaggle sont des re-uploads de particuliers aux licences parfois fausses : réserver à un usage académique.

Détail et sources : [research/09-dataset-plus-varie.md](../research/09-dataset-plus-varie.md).

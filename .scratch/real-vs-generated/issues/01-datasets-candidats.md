# Quels datasets image réelle vs image générée existent ?

Type: research
Status: resolved
Blocked by: —

## Question

Quels datasets publics et réalistes permettent de classer image réelle vs image générée, avec des images générées venant de **plusieurs générateurs** ? Point de départ : GenImage, mais chercher aussi les alternatives (CIFAKE est écarté, jugé trop basique).

Pour chaque candidat, relever :
- la taille totale (nombre d'images, Go) ;
- les générateurs couverts et le nombre d'images par générateur ;
- la source et la nature des images réelles ;
- la résolution et les formats de fichiers par classe ;
- la licence ;
- la facilité d'accès depuis Kaggle/Colab (dataset Kaggle existant ? sous-ensemble téléchargeable ?).

Indiquer quel sous-ensemble raisonnable tiendrait dans les limites de disque et de temps de Kaggle/Colab.

## Recherche

Rédaction complète (131 lignes, sources citées) : pas encore poussée sur `research/01-datasets-candidats`, l agent ayant été bloqué par la protection d écriture entre worktrees. Copie provisoire dans le scratchpad de la session.

## Answer

**GenImage** (NeurIPS 2023 D&B) est la base : 2,68 M d'images (1,33 M réelles d'ImageNet, 1,35 M générées), environ 500 Go, licence CC BY-NC-SA 4.0. Elle couvre 8 générateurs : Midjourney v5 (1024 px), SD 1.4 et SD 1.5 (512 px), Wukong (512 px), ADM, GLIDE et VQDM (256 px), BigGAN (128 px). Trop gros en entier pour Kaggle/Colab, mais il existe des miroirs Kaggle réduits :
- `cartografia/unbiased-tiny-genimage` (2,53 Go), déjà en JPEG qualité 96 pour les deux classes, donc sans le biais de format ;
- `yangsangtai/tiny-genimage` (8,35 Go) ;
- `tripya/genimage-224-8-generators` (13,7 Go, 10k réelles + 10k générées par générateur, en 224 px).

Alternatives écartées ou secondaires :
- **Synthbuster** (9 générateurs récents, 1 000 images chacun, 12,4 Go) : test seulement, bon jeu hors distribution ;
- **Community Forensics** (≥ 260 Go en version Small) et **OpenFake** : trop gros ;
- **ELSA D3**, **DRCT-2M**, **WildFake**, **Chameleon** : accès difficile, non publié, ou trop centré sur Stable Diffusion.

Recommandation de l'agent :
- entraîner sur un sous-ensemble GenImage débiaisé (unbiased-tiny-genimage, ou tiny-genimage recompressé nous-mêmes) ;
- mettre de côté Midjourney et BigGAN comme générateurs jamais vus ;
- en option, tester sur Synthbuster (DALL·E 2/3, Firefly, SD 2, SDXL).

Non vérifié :
- si tiny-genimage contient aussi des images réelles ;
- le format des images réelles du jeu 224 px ;
- la résolution et le format de Synthbuster ;
- les limites officielles de Kaggle (30 h GPU/semaine, sessions de 12 h, 20 Go de disque, chiffres de sources secondaires) ;
- le disque gratuit de Colab.

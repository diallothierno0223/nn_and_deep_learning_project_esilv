# Inspecter un échantillon des datasets candidats

Type: task
Status: resolved
Blocked by: 01

## Question

Télécharger un petit échantillon de chaque dataset candidat retenu par le ticket « Quels datasets image réelle vs image générée existent ? », puis le regarder avec l'utilisateur :
- quelques images réelles et générées côte à côte ;
- les tailles, formats et résolutions réellement observés par classe.

Le but est de juger si le dataset convient avant de le choisir. C'est une tâche HITL : c'est l'utilisateur qui décide si les images lui conviennent.

## Comments

**Observé le 2026-11-23** sur https://www.kaggle.com/datasets/cartografia/unbiased-tiny-genimage (explorateur de fichiers Kaggle, version 1, 2,53 Go) :
- 8 dossiers : `Nature` (images réelles, 5 828 fichiers `.JPEG` ImageNet), et 7 générateurs à 2 500 fichiers `.jpg` chacun : `ADM`, `BigGAN`, `Midjourney`, `VQDM`, `glide`, `stable_diffusion_v_1_5`, `wukong`. SD 1.4 est absent. Soit 17 500 images générées pour 5 828 réelles (≈ 3 pour 1).
- Pas de split train/test fourni. Un fichier `nature_metadata.csv` donne nom, classe, largeur et hauteur des images réelles.
- Contenu : les deux classes montrent les mêmes sujets ImageNet (poissons, oiseaux, requins, éléphants…), donc le contenu ne trahit pas la classe.
- Format : tout est en JPEG (qualité 96 selon la fiche), le biais de format est traité.
- **Biais de taille restant probable** : les réelles font 450–550 px, mais les générateurs gardent leur taille native (BigGAN 128 px ; ADM, GLIDE, VQDM 256 px ; SD et Wukong 512 px ; Midjourney 1024 px). À vérifier sur fichiers, et à traiter dans le ticket prétraitement (crop, ou ne garder que les générateurs proches de 512 px).

## Answer

L'utilisateur a regardé l'échantillon et juge `unbiased-tiny-genimage` satisfaisant (« ce dataset a l'air bien »). Faits à reporter dans les tickets suivants :
- 5 828 images réelles ; 7 générateurs à 2 500 images chacun ; tout en JPEG ;
- pas de split train/test fourni ;
- déséquilibre d'environ 3 pour 1 ;
- biais de taille probable entre générateurs (de 128 à 1024 px).

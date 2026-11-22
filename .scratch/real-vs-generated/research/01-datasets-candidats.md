# Datasets candidats : image réelle vs image générée (multi-générateurs)

Ticket : `.scratch/real-vs-generated/issues/01-datasets-candidats.md`
Recherche faite le 2026-11-23. Chaque affirmation est suivie de sa source. Ce qui n'a pas pu être vérifié est marqué **[non vérifié]**.

## TL;DR

- **GenImage reste le meilleur point de départ** : 8 générateurs (1 GAN + 7 diffusion), réels ImageNet, découpage par générateur déjà prêt pour le test « générateur jamais vu ». Il est trop gros en entier (~500 Go), mais il existe sur Kaggle des **sous-ensembles prêts à attacher** (tiny-genimage 8,35 Go, Unbiased Tiny GenImage 2,53 Go, GenImage 224 13,7 Go) et des **miroirs complets par générateur** (24 à 107 Go chacun).
- **Biais connu et critique** : dans GenImage, les réels sont des JPEG (qualité ~96) de taille variable, les faux des PNG à taille fixe. Un CNN peut apprendre « JPEG = réel » au lieu de « généré ». Il faut corriger ça (sous-ensemble « unbiased » ou compression JPEG des faux), sinon le score est trompeur.
- **Meilleur complément pour le test hors distribution** : **Synthbuster** (9 générateurs récents dont DALL·E 3, Firefly, Midjourney v5, SDXL ; 1 000 images chacun ; 12,4 Go ; réels RAISE-1k = vraies photos d'appareil). Il est fait pour le test seulement, et il existe des copies Kaggle.
- Les alternatives plus récentes (OpenFake, Community Forensics, ELSA D3, DRCT-2M, WildFake, Chameleon) sont soit énormes, soit à accès restreint, soit pas publiées. Elles servent au mieux de source pour un petit jeu de test supplémentaire.

## Tableau récapitulatif

| Dataset | Taille | Générateurs | Réels | Licence | Accès Kaggle/Colab |
|---|---|---|---|---|---|
| GenImage (NeurIPS 2023 D&B) | 2 681 167 images, ~500 Go | 8 : Midjourney v5, SD 1.4, SD 1.5, ADM, GLIDE, Wukong, VQDM, BigGAN | ImageNet (1,33 M) | CC BY-NC-SA 4.0 + conditions | Miroirs Kaggle par générateur + sous-ensembles « tiny » |
| Synthbuster | 9 000 faux, 12,4 Go (+ RAISE-1k à part) | 9 : DALL·E 2, DALL·E 3, Firefly, Midjourney v5, SD 1.3/1.4/2/XL, GLIDE | RAISE-1k (photos brutes d'appareil) | CC BY-NC-SA 4.0 | Zenodo (zip direct) + copies Kaggle |
| Community Forensics (CVPR 2025) | ~2,7 M faux, ~1,08 To ; version Small ~260-278 Go | 4 803 modèles | LAION, ImageNet, COCO, FFHQ, etc. | CC BY 4.0 (Small : CC BY-NC-SA 4.0) | Hugging Face, trop gros |
| OpenFake | ~963 k faux + réels LAION ; ~2,86 M lignes | 18 familles (SD, Flux, DALL·E 3, MJ 6/7, Imagen, GPT Image 1, Grok…) | LAION-400M filtré, Pexels, ImageNet, Reddit | CC BY-NC 4.0 | Hugging Face (Parquet), streaming possible |
| ELSA D3 | 2,3 M lignes × 4 générateurs | DeepFloyd IF, SD 1.4, SD 2.1, SDXL | LAION-400M (URL) | non trouvé | Hugging Face |
| DRCT-2M | ~2 M faux | 16 variantes Stable Diffusion | MSCOCO | non trouvé | ModelScope uniquement |
| WildFake | 3,69 M images | ~20 (GAN, diffusion, autres) | COCO, FFHQ, ImageNet, LAION… | non trouvé | Pas publié (« plan to release ») |
| Chameleon (AIDE) | ~26 000 images | MJ, DALL·E 3, SD + LoRA (sans détail) | Unsplash | accès sur EULA | Test seulement, accès sur demande |

## 1. GenImage (point de départ)

**Taille.** 2 681 167 images : 1 331 167 réelles (1 281 167 train + 50 000 test) et 1 350 000 générées. Source : article, §2.1, https://arxiv.org/abs/2306.08571 (PDF https://arxiv.org/pdf/2306.08571). Volume total annoncé « ~500GB » par https://www.unbiased-genimage.org. Publié à NeurIPS 2023, track Datasets and Benchmarks : https://neurips.cc/virtual/2023/poster/73644.

**Générateurs et images par générateur.** Midjourney (v5), Stable Diffusion V1.4, Stable Diffusion V1.5, ADM, GLIDE, Wukong, VQDM, BigGAN. 1 350 images générées par classe ImageNet (1 300 train, 50 test). Chaque générateur produit 162 images train et 6 test par classe, sauf SD V1.5 avec 166 train et 8 test. Cela donne environ 162 000 faux train et 6 000 faux test par générateur (calcul à partir de l'article, §2.1). Chaque générateur forme un « sous-ensemble » avec ses propres réels, et les réels ne sont pas partagés entre sous-ensembles (même source). Le dépôt organise les données en `<générateur>/train|val/ai|nature` : https://github.com/GenImage-Dataset/GenImage.

**Images réelles.** Toutes les images d'ImageNet (1 000 classes) : https://arxiv.org/abs/2306.08571 §2.1 et le README GitHub (« the nature folder contains the collected images from ImageNet »). Ce sont des photos du web, pas forcément prises par l'utilisateur au téléphone, mais ce sont bien des photos réelles.

**Résolution et formats.**
- Générés : Midjourney 1024×1024 ; SD 1.4/1.5 et Wukong 512×512 ; GLIDE et VQDM 256×256 ; BigGAN 128×128 (article §2.2). ADM 256×256 selon Grommelt et al. (https://arxiv.org/html/2403.17608, §3.2).
- Format : générés en **PNG non compressé**, réels ImageNet en **JPEG**, « majoritairement qualité 96 », taille variable (~450×450 en général). Source : Grommelt et al., « Fake or JPEG? Revealing Common Biases in Generated Image Detection Datasets », https://arxiv.org/html/2403.17608, §3.1, §3.2 et §4.2.
- Conséquence : un détecteur peut apprendre le format au lieu du contenu. En imposant la même qualité JPEG 96 aux faux et une taille 450-550 px aux réels, la précision moyenne inter-générateurs d'un ResNet-50 passe de 71,68 % à 82,74 % (même source, tableau 4). Ce point concerne directement le ticket 02 (biais de format).

**Généralisation entre générateurs.** Entraîné sur SD V1.4, un ResNet-50 atteint ~99,9 % sur SD 1.4/1.5 et Wukong mais tombe vers 50-55 % sur Midjourney, VQDM et BigGAN (article, tableau 2). C'est donc un bon terrain pour l'évaluation sur générateurs exclus.

**Licence.** Fichier `License` du dépôt : CC BY-NC-SA 4.0 avec des conditions en plus, usage non commercial seulement (recherche, enseignement). Source : https://github.com/GenImage-Dataset/GenImage/blob/main/License. Les images ImageNet restent soumises aux conditions d'ImageNet.

**Accès officiel.** Baidu Yunpan et Google Drive (lien publié le 2024-03-15) dans le README GitHub. Le site Unbiased GenImage propose aussi un script de téléchargement depuis Harvard Dataverse (~500 Go) et un `metadata.csv` (taille, qualité JPEG, classe) pour construire soi-même des sous-ensembles débiaisés : https://www.unbiased-genimage.org.

**Accès depuis Kaggle (vérifié via l'API publique Kaggle `api/v1/datasets/list`, le 2026-11-23).**

| Dataset Kaggle | Taille | Contenu | Licence affichée |
|---|---|---|---|
| `yangsangtai/tiny-genimage` | 8,35 Go | 5 000 images par générateur (train + val), sauf SD 1.4, au format d'origine (PNG pour `ai/`) | CC BY-NC-SA 4.0 |
| `cartografia/unbiased-tiny-genimage` | 2,53 Go | tiny-genimage débiaisé selon Grommelt et al. : faux recompressés en JPEG q96, réels q96 et 450-550 px, complétés depuis ImageNet | CC BY-NC-SA 4.0 |
| `tripya/genimage-224-8-generators` | 13,69 Go | 8 générateurs × (10 000 réels + 10 000 faux), tout en 224×224 | CC0 (affichée par l'uploader, douteux vu la licence d'origine) |
| `vtphatt2/genimage-*` (miroir complet) | validation 25,4 Go ; BigGAN 24,3 ; GLIDE 32,3 ; VQDM 38,2 ; ADM 38,9 ; SD 1.4 96,5 ; SD 1.5 98,6 ; Wukong 97,3 ; Midjourney 107,4 + 107,4 + 12,7 | Miroir non officiel, un dataset par générateur (https://github.com/vtphatt2/GenImage-mirror) | CC BY-NC-SA 4.0 |

Un équivalent Hugging Face de tiny-genimage existe : `TheKernel01/Tiny-GenImage`, 35 000 lignes (28 000 train / 7 000 val), Parquet, 8,36 Go, CC BY-NC-SA 4.0 (https://huggingface.co/datasets/TheKernel01/Tiny-GenImage).

Limites de cette vérification :
- **[non vérifié]** Le nombre exact de réels dans `tiny-genimage` : la description dit « 5000 images (including training set and verification set) for each model », sans dire si les réels sont inclus dans ces 5 000. Je n'ai lu que la première page de la liste de fichiers.
- **[non vérifié]** Le format des réels dans `tripya/genimage-224-8-generators` : les faux sont en `.png`, mais je n'ai pas vu de fichier réel. Si les réels ont été décodés depuis le JPEG puis réenregistrés en PNG 224, le biais de taille disparaît, mais les traces de compression JPEG restent dans les pixels.
- Les tailles des miroirs `vtphatt2` (~679 Go au total) dépassent les ~500 Go annoncés. Ce sont les tailles décompressées indiquées par Kaggle.

## 2. Synthbuster (meilleur jeu de test hors distribution)

- **Contenu.** 1 000 images générées par modèle pour 9 modèles : DALL·E 2, DALL·E 3, Adobe Firefly, Midjourney v5, Stable Diffusion 1.3, 1.4, 2, XL, et GLIDE. Elles sont générées à partir de prompts décrivant les photos de RAISE-1k (`prompts.csv` fourni). Aucune compression JPEG ni rééchantillonnage. Source : https://zenodo.org/records/10066460 et https://www.veraai.eu/posts/dataset-synthbuster-towards-detection-of-diffusion-model-generated-images.
- **Réels.** RAISE-1k, des photos brutes d'appareil photo. Elles ne sont **pas incluses** et sont à télécharger séparément (http://loki.disi.unitn.it/RAISE/download.html). Mêmes sources que ci-dessus.
- **Taille.** `synthbuster.zip` fait 12,4 Go. Licence : CC BY-NC-SA 4.0 (Zenodo).
- **Résolution et format.** **[non vérifié]** Non indiqués sur Zenodo. Il faudra inspecter l'archive (ticket 04). Les images RAISE sont des photos haute résolution, donc il faudra les recadrer ou les redimensionner.
- **Kaggle.** Plusieurs copies de 12,39 Go (`maryjoy12/synthbuster`, `vhininh/synthbuster`, `erengencturk/synthbuster`…) et une partie réels `ngovanphuong2604/synthbuster-real-raise-part` (22,6 Go). Ce sont des copies non officielles, et je n'ai pas vérifié leur contenu.
- **Intérêt.** Ces générateurs (DALL·E 3, Firefly, SDXL) n'existent pas dans GenImage, et les réels sont de vraies photos d'appareil. Seuls Midjourney v5, SD 1.4 et GLIDE se recoupent avec GenImage : pour un test vraiment « jamais vu », il faut garder DALL·E 2/3, Firefly, SD 2 et SDXL.

## 3. Community Forensics (CVPR 2025)

- ~2,7 M images générées (2 760 270 lignes), **4 803 générateurs**, découpées en Systematic (1 919 493), Manual (774 023), Commercial (14 918) et PublicEval (51 836, avec réels COCO/FFHQ). Source : https://huggingface.co/datasets/OwensLab/CommunityForensics.
- Réels : LAION et ImageNet (~40 % chacun), CelebA, COCO, FFHQ, VISION, Forchheim, etc. Taille ~1,08 To, et jusqu'à 2,2 To d'espace pendant le téléchargement. Format Parquet, exemples en PNG 512×512. Licence CC BY 4.0, usage recherche (même page).
- Version **Small** : ~278 k faux + ~278 k réels, 260-278 Go selon la page (chiffres incohérents sur la carte), CC BY-NC-SA 4.0 (https://huggingface.co/datasets/OwensLab/CommunityForensics-Small).
- Verdict : trop gros pour Kaggle/Colab, même en Small. On peut au mieux streamer quelques milliers d'images de l'éval pour un test.

## 4. OpenFake

- ~963 342 images synthétiques de 18 familles de générateurs récents : SD 1.5/2.1/XL/3.5, Flux dev/schnell/1.1-pro, DALL·E 3, Midjourney 6/7, Imagen 3/4, GPT Image 1, Grok 2, Ideogram 3, HiDream, Recraft v3, Chroma. Les comptes vont de 1 000 à 186 666 par famille. Source : https://arxiv.org/html/2509.09495, tableau 5.
- Réels : ~3 M images filtrées de LAION-400M (visages, actualité). Seul un sous-ensemble équilibré est hébergé, le reste est donné par URL. Réels < 512×512 exclus, faux ~1 MP en ratios variés. Le test a des générateurs exclus de l'entraînement (même source).
- Hugging Face : ~2,86 M lignes (config `core` 2,45 M), Parquet, licence CC BY-NC 4.0 (https://huggingface.co/datasets/ComplexDataLab/OpenFake).
- **[non vérifié]** La taille en Go et le format des fichiers images.
- Verdict : très réaliste et récent, mais trop gros en entier. Le split `test` (~91 k) ou la config `reddit` (36,1 k) pourraient servir de test hors distribution en streaming.

## 5. ELSA D3

- 2,31 M lignes train + 4,8 k validation. Chaque ligne = une image réelle (URL LAION) + 4 générées par DeepFloyd IF, SD 1.4, SD 2.1 et SDXL. Source : https://huggingface.co/datasets/elsaEU/ELSA_D3 et l'article https://arxiv.org/html/2407.20337.
- Résolutions 256×256 (IF), 512×512, 640×480 et 640×360 (SD). Les réels LAION sont à ~91 % en JPEG (article).
- **[non vérifié]** La licence et la taille en Go. Les réels sont donnés **par URL**, avec des liens morts probables. Seulement 4 générateurs, tous de la famille diffusion.

## 6. DRCT-2M

- ~2 M images générées de 256 à 1024 px, 16 modèles de la famille Stable Diffusion (10 T2I + 3 ControlNet + 3 inpainting), prompts et réels MSCOCO. Il existe aussi DRCT-2M-Wild (136 k images de 8 plateformes). Source : article OpenReview https://openreview.net/pdf/6a65ad38d0c82d1a5b968eef583f28efb1c0a6bd.pdf (introduction) et https://arxiv.org/pdf/2507.17240 pour le découpage 10/3/3.
- Distribué seulement sur ModelScope (https://modelscope.cn/datasets/BokingChen/DRCT-2M/files, lien donné par https://github.com/beibuwandeluori/DRCT).
- **[non vérifié]** La licence, la taille en Go et le nombre par générateur. Les variantes inpainting ne sont pas des images « entièrement générées », il faudrait les exclure.

## 7. WildFake

- 3 694 313 images : 1 013 446 réelles et 2 680 867 générées (le tableau 1 dit 2 557 278, incohérence dans l'article). Générateurs GAN (BigGAN, StyleGAN, GigaGAN…), diffusion (DALL·E, ADM, Imagen, Midjourney V5, SD…) et autres (VQGAN, Muse…). Réels COCO, FFHQ, ImageNet, LSUN, LAION-5B, etc. Source : https://arxiv.org/html/2402.11843.
- Les auteurs disent qu'ils « prévoient de publier » le dataset. Je n'ai trouvé aucun lien officiel. Il existe un dataset Kaggle `shxrlenee/aigc-detection-dataset` (12,65 Go) remonté par la recherche « wildfake », mais **[non vérifié]** son lien avec WildFake.

## 8. Chameleon (article AIDE)

- 11 170 images générées + 14 863 photos réelles (Unsplash), de 720p à 4K, prises sur ArtStation, Civitai et Liblib (Midjourney, DALL·E 3, SD + LoRA), sans détail par générateur. Test seulement, accès sur EULA et validation manuelle. Source : https://arxiv.org/html/2406.19435.
- Verdict : très réaliste mais difficile d'accès, et aucune étiquette de générateur ne permet le protocole « générateur exclu ».

## Contraintes Kaggle / Colab

- Colab gratuit : 12 h maximum par session, déconnexion en cas d'inactivité, GPU non garantis, et les limites ne sont pas publiées (https://research.google.com/colaboratory/faq.html). Le disque gratuit n'est pas documenté dans la FAQ **[non vérifié]**. Il faut tout retélécharger à chaque session.
- Kaggle : la page officielle https://www.kaggle.com/docs/notebooks n'a pas pu être lue (rendu JavaScript). Les sources secondaires donnent ~30 h GPU par semaine, des sessions de 12 h et 20 Go persistés dans `/kaggle/working` (https://www3.nd.edu/~dchiang/teaching/nlp/2023/kaggle.html, https://developer.nvidia.com/blog/how-kaggle-makes-gpus-accessible-to-5-million-data-scientists/). **[non vérifié]** À contrôler dans les réglages du compte.
- Point clé : sur Kaggle, un dataset Kaggle s'attache comme entrée en lecture seule, **sans le copier sur le disque de travail**. C'est pour ça que les miroirs Kaggle de GenImage et Synthbuster sont plus pratiques que Google Drive, Baidu ou Zenodo depuis Colab. **[non vérifié]** La limite de volume total d'entrées attachées à un notebook.

## Recommandation : sous-ensemble raisonnable

Il faut un jeu qui tienne en quelques heures d'entraînement sur un GPU T4/P100 et en quelques Go :

1. **Entraînement et validation** : un GenImage réduit et **débiaisé**.
   - Option la plus simple : `cartografia/unbiased-tiny-genimage` (2,53 Go, déjà en JPEG q96 des deux côtés).
   - Option plus grande : `yangsangtai/tiny-genimage` (8,35 Go) en appliquant nous-mêmes la recette de Grommelt et al. (faux recompressés en JPEG q96, réels filtrés 450-550 px, crop puis resize 224). On peut aussi prendre `tripya/genimage-224-8-generators` (13,7 Go, 20 000 images par générateur), en vérifiant d'abord le format des réels.
   - Ordre de grandeur : 5 000 à 10 000 images par générateur × 5-6 générateurs = 50 000 à 120 000 images en 224×224. Une époque de transfer learning (EfficientNet ou ResNet-50 en Keras) sur T4 prend quelques minutes à quelques dizaines de minutes : ça tient dans une session.
2. **Générateurs exclus (1-2)** : retirer complètement un ou deux générateurs de GenImage de l'entraînement, par exemple **Midjourney** (diffusion commerciale, haute résolution) et **BigGAN** (GAN). Ce sont ceux qui généralisent le moins d'après le tableau 2 de l'article, donc le test sera informatif.
3. **Test externe facultatif** : Synthbuster (12,4 Go, attachable sur Kaggle) restreint à DALL·E 2/3, Firefly, SD 2 et SDXL, avec un échantillon de RAISE-1k comme réels. C'est un vrai test hors distribution, avec d'autres générateurs et d'autres réels.
4. **À éviter** pour ce projet : le GenImage complet (~500 Go), Community Forensics, ELSA D3, DRCT-2M et OpenFake en entier, trop lourds ou trop difficiles d'accès. WildFake et Chameleon ne sont pas accessibles simplement.

## Points non vérifiés (résumé)

- Composition exacte (réels inclus ou non) de `yangsangtai/tiny-genimage` et format des réels dans `tripya/genimage-224-8-generators`.
- Résolution et format des images Synthbuster, et contenu réel des copies Kaggle.
- Licence et taille de ELSA D3 et DRCT-2M ; taille en Go d'OpenFake ; disponibilité de WildFake.
- Limites officielles Kaggle (page de doc non lisible) et disque Colab gratuit.
- Les licences « CC0 », « Apache 2.0 » ou « MIT » affichées sur certains datasets Kaggle dérivés de GenImage contredisent la licence d'origine CC BY-NC-SA 4.0 : c'est la licence d'origine qui s'applique.

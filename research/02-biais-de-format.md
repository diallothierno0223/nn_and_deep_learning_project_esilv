# Biais de format dans les datasets réel vs généré, et prétraitements qui les neutralisent

Réponse au ticket `.scratch/real-vs-generated/issues/02-biais-de-format.md`.
Recherche faite le 2026-11-23 à partir des articles (arXiv) et des dépôts officiels. Chaque affirmation renvoie à sa source. Ce qui n'a pas pu être vérifié est signalé par **[non vérifié]**.

## TL;DR

- Dans GenImage, les deux classes diffèrent par des propriétés **sans rapport avec la génération**. Les images réelles viennent d'ImageNet, en JPEG, la plupart en qualité 96. Les images générées sont en PNG, donc sans compression. Chaque générateur a aussi une taille fixe (128, 256, 512 ou 1024 px), alors que les images réelles ont des tailles variées, autour de 450×450. ([Grommelt et al., 2024](https://arxiv.org/abs/2403.17608))
- Les classifieurs **exploitent ces raccourcis**. Un ResNet50 entraîné sur GenImage/Midjourney classe 80,45 % des photos réelles PNG (FFHQ) correctement, et 100 % de ces mêmes photos une fois passées en JPEG qualité 60. Il a donc appris l'équation « JPEG = réel ». ([Grommelt et al.](https://arxiv.org/html/2403.17608))
- **« Même taille + pixels seuls » ne suffit pas.** Ne lire que les pixels élimine l'extension et les métadonnées, mais les traces de compression sont dans les pixels. Redimensionner à une taille commune ajoute un nouveau biais : le facteur de redimensionnement diffère d'une classe à l'autre, donc les traces de rééchantillonnage aussi. ([Rajan et al., 2024](https://arxiv.org/html/2410.11835) ; [Guillaro et al., B-Free](https://arxiv.org/html/2412.17671))
- Prétraitement recommandé :
  1. **Aligner la compression.** Recompresser les images générées en JPEG à la qualité des réelles (q = 96 pour GenImage), ou compresser les deux classes de la même façon.
  2. **Aligner les tailles.** Garder les réelles dont la taille est proche de celle du générateur.
  3. **Recadrer (crop) plutôt que redimensionner.**
  4. **Augmenter avec du flou et du JPEG aléatoires, appliqués aux deux classes.**
- **Vérification après coup.** Faire passer le même test JPEG ou resize aux images réelles et aux générées, puis mesurer de combien le score se décale vers une classe ([BIAS-ID](https://arxiv.org/html/2605.31153)). Compléter par un test sur des photos réelles non compressées (PNG/RAW) et par une baseline « triviale » qui ne voit que le format ou la taille.

## 1. Biais non sémantiques identifiés dans GenImage

| Biais | Images réelles (ImageNet) | Images générées | Source |
|---|---|---|---|
| Format / compression | JPEG, « the majority of images compressed using a quality factor of 96 » | PNG, non compressées | [Grommelt et al., Annexe A et Table 1](https://arxiv.org/html/2403.17608) |
| Taille | Variable, la plupart autour de 450×450 | Fixe par générateur : BigGAN 128², GLIDE/ADM/VQDM 256², SD1.4/SD1.5/Wukong 512², Midjourney 1024² | [Grommelt et al., §3.2](https://arxiv.org/html/2403.17608) ; tailles confirmées dans [GenImage, §2.2](https://arxiv.org/html/2306.08571) (sauf ADM, absent du papier GenImage) |
| Ratio d'aspect | Variable (déduit des tailles variables d'ImageNet) | Carré | Déduit des deux sources ci-dessus ; **[non vérifié]** : aucun papier lu ne mesure isolément l'effet du ratio |
| Métadonnées EXIF | — | — | **[non vérifié]** : aucune source primaire trouvée sur un biais EXIF dans GenImage. Le projet n'utilise que les pixels, donc la question ne se pose pas. |

Le papier GenImage lui-même ne parle pas de biais de format. Il teste seulement la robustesse à JPEG (q = 65 et 30), au flou et à la basse résolution, sur le test de SD V1.4, et note que « JPEG compression has a significant impact on these backbone models ». ([GenImage, §3.3](https://arxiv.org/html/2306.08571))

Ce n'est pas propre à GenImage :
- Les fausses images sont souvent en PNG et les vraies en JPEG. Ce problème de format est connu en criminalistique d'image depuis au moins 2013 : « Format issues have been the Achilles' heel of forensic detectors since at least 2013 ». ([B-Free, Introduction](https://arxiv.org/html/2412.17671))
- Sur LSUN, la plupart des images réelles « are compressed with the same quality factor ». ([Cozzolino et al., Raising the Bar, §4.2](https://arxiv.org/html/2312.00195))
- Dans un autre jeu, toutes les images réelles (LAION) avaient été redimensionnées, et pas les fausses. ([B-Free, Introduction](https://arxiv.org/html/2412.17671))

## 2. Preuves que les classifieurs exploitent ces raccourcis

**JPEG** ([Grommelt et al., §3.1, Table 2](https://arxiv.org/html/2403.17608))
- Prenons un ResNet50 entraîné sur GenImage brut. Si on compresse ses images générées de test, même en JPEG 95, le rappel chute alors que la précision reste proche de 1 : les fausses images compressées sont classées « réelles ».
- Contre-test sur des photos réelles non compressées (FFHQ, PNG, 1024²), avec le détecteur entraîné sur Midjourney. Sa précision monte avec la compression : PNG 80,45 %, JPEG95 94,84 %, JPEG90 98,93 %, JPEG80 99,95 %, JPEG60 100 %. La compression ne peut pas avoir « détruit des artefacts de génération » ici, puisque ces images sont réelles. Le modèle détecte donc bien la compression.

**Taille** ([Grommelt et al., §3.2](https://arxiv.org/html/2403.17608))
- Le détecteur BigGAN (128²) se trompe sur les images réelles dont un côté mesure entre 100 et 150 px. Même chose pour ADM (256²) près de sa taille. Le détecteur Midjourney est moins bon sur les grandes images réelles.
- Aucun effet net n'apparaît pour Stable Diffusion.

**Rééchantillonnage** ([Rajan et al., §3.3, §5.2, Annexe A.2.2](https://arxiv.org/html/2410.11835))
- Les réelles sont plus grandes que les fausses. Un « random resized crop » réduit donc surtout les réelles et agrandit surtout les fausses. Le modèle apprend alors que « réduction = réel ».
- Les auteurs le montrent sur le détecteur de Corvi : son score sur les fausses chute fortement quand on les réduit, parce qu'il « has only seen downsizing artifacts on real images ».
- Ojha-LDM « ends up associating downsampling with real images ».

**Gain mesuré après débiaisage** ([Grommelt et al., §4](https://arxiv.org/html/2403.17608))
- Précision moyenne en généralisation inter-générateurs :
  - ResNet50 : 71,68 % → 82,74 % (+11,06 points)
  - Swin-T : 74,09 % → 85,83 % (+11,74 points)
- Robustesse au JPEG : sur un test en JPEG95, la précision passe de 53,91 % à 67,17 %.

**Autres travaux qui confirment l'importance du format et du prétraitement**
- Sans ré-encodage commun des images, l'AUROC sous JPEG-50 peut *dépasser* celle sur images propres, signe d'un biais de format. ([Zhou & Wang, 2026, résumé](https://arxiv.org/abs/2606.20488)) Préprint.
- Passer d'un « resize-to-512 » à la résolution native inverse les conclusions par générateur, jusqu'à 0,38 d'AUROC. ([même source](https://arxiv.org/abs/2606.20488)) Préprint.
- Plusieurs détecteurs de l'état de l'art sont « strongly affected by common, processing-related biases ». Par exemple, AIDE et SPAI baissent fortement leur score « généré » quand on compresse en JPEG. ([BIAS-ID, NeurIPS 2026 D&B](https://arxiv.org/html/2605.31153))

## 3. « Même taille + pixels seuls » suffit-il ? Non

- **Pixels seuls.** On élimine l'extension, l'EXIF et les tables de quantification. Mais une image JPEG décodée garde ses artefacts de compression dans les pixels. Grommelt et al. ont mis ce biais en évidence avec des détecteurs CNN qui ne lisent que des pixels décodés.
  - Vérifié : les détecteurs de GenImage n'utilisent que les pixels.
  - **[non vérifié]** : le pipeline exact de la baseline n'est pas documenté. Le papier mentionne qu'un ResNet50 « resizes all the input data to 224×224 pixels » dans une discussion sur la taille, sans le présenter comme la configuration d'entraînement. Il ne teste pas explicitement si le redimensionnement efface les traces JPEG. ([Grommelt et al.](https://arxiv.org/html/2403.17608))
- **Même taille par redimensionnement.** Le biais de taille devient un biais de rééchantillonnage.
  - Dans GenImage, ramener tout le monde à 224² revient à réduire une image SD (512²) d'un facteur 0,44 et une image réelle (~450²) d'un facteur ≈ 0,5, avec des facteurs variables. Pour Midjourney (1024²) ou BigGAN (128², agrandie), les deux classes portent des signatures d'interpolation très différentes.
  - Ce mécanisme est démontré par [Rajan et al.](https://arxiv.org/html/2410.11835) (§1 : « if they have to be resized to a fixed size, real and fake images could have different amount of resizing artifacts »).
  - Le redimensionnement « wash[es] out the subtle low-level forensic clues ». ([B-Free](https://arxiv.org/html/2412.17671))
- **Survie des traces JPEG après redimensionnement.** Aucune source primaire lue ne la mesure directement : **[non vérifié]**. Les arguments ci-dessus suffisent quand même à conclure que redimensionner ne neutralise pas le biais, et peut en ajouter un.

## 4. Prétraitements recommandés par la littérature

### 4.1 Aligner la compression (le plus important pour GenImage)
- **Recompresser les générées en JPEG qualité 96**, celle de la majorité des réelles ImageNet. C'est le « JPEG constraint » de [Grommelt et al., §4.1](https://arxiv.org/html/2403.17608), dont le code est dans [`jpeg_augment.py`](https://github.com/gendetection/UnbiasedGenImage) ([site du projet](https://www.unbiased-genimage.org)).
- Variante utilisée dans [B-Free, §3.1](https://arxiv.org/html/2412.17671) pour construire « GenImage unbiased » : compresser les fausses « at a JPEG quality close to those used for the real class ».
- Si les données le permettent, enregistrer les fausses dans le même format que les réelles. [Rajan et al., Annexe A.1.2](https://arxiv.org/html/2410.11835) : « we also save our fake images in the JPG format… to make sure that the detector does not focus on compression artifacts ».
- Le plus simple à appliquer : **recompresser les deux classes au même facteur de qualité** (par exemple q = 96, ou un tirage aléatoire identique pour les deux classes).
  - **[non vérifié]** : aucun papier lu ne compare directement cette option à « ne recompresser que les fausses ».
  - Limite : recompresser une réelle déjà en JPEG produit une double compression, qu'un CNN peut en théorie distinguer d'une compression simple. Le risque est plus faible si le facteur de qualité est le même (96 sur 96). **[non vérifié]** dans le contexte GenImage.

### 4.2 Aligner tailles et ratios
- Grommelt et al. ([§4.2](https://arxiv.org/html/2403.17608) ; [site](https://www.unbiased-genimage.org)) procèdent ainsi :
  - Ne garder que les générateurs en 512² : SD1.4, SD1.5, Wukong.
  - Ne garder que les réelles dont la taille est entre 450 et 550 px.
  - À l'entraînement : crop central 450, puis resize 224.
  - Au test : resize 512, crop 450, puis resize.
  - Le jeu d'entraînement passe d'environ 320 000 à 75 000 images.
- Pour le projet : si on entraîne sur un seul sous-ensemble (par ex. SD1.4), filtrer les réelles autour de 512 px est peu coûteux et retire l'essentiel du biais de taille.

### 4.3 Crop plutôt que resize
- B-Free : « we avoid resizing the image and rely on large crops of 504×504 pixels ». ([§4](https://arxiv.org/html/2412.17671))
- CNNDetection (Wang et al., CVPR 2020) évalue sur un crop 224×224, et propose aussi une évaluation sans crop ([README](https://github.com/PeterWang512/CNNDetection)).
- Pour un backbone Keras pré-entraîné en 224² : faire un **random crop 224 à l'entraînement et un center crop au test**, sans redimensionner. Si un resize est inévitable (images plus petites que 224, comme BigGAN 128²), appliquer **le même facteur aux deux classes**, ou exclure ces sources. [Rajan et al.](https://arxiv.org/html/2410.11835) appliquent « identical data augmentations (e.g., crop) » à chaque paire réelle/fausse.

### 4.4 Augmentations appliquées aux deux classes
- CNNDetection : flou gaussien avec `--blur_prob 0.5`, `--blur_sig 0.0,3.0`, et JPEG avec `--jpg_prob 0.5`, `--jpg_qual 30,100` (modèles « Blur+JPEG(0.5) » et « (0.1) »). ([README](https://github.com/PeterWang512/CNNDetection) ; [papier](https://arxiv.org/abs/1912.11035))
- GenImage attribue la robustesse de CNNSpot à cette augmentation. ([GenImage, §3.3](https://arxiv.org/html/2306.08571))
- B-Free garde une augmentation standard (flou + JPEG) dans toutes ses variantes et ajoute « scaling, cut-out, noise addition, and jittering ». ([Table 2, §4.2](https://arxiv.org/html/2412.17671))
- Remarque : l'augmentation seule ne compense pas un jeu biaisé. Cozzolino et al. notent que « data augmentation only marginally reduces the effects of the dataset used ». ([§4.2](https://arxiv.org/html/2312.00195))

### 4.5 Solutions plus lourdes (hors périmètre probable du projet)
- **Données alignées.** On fabrique les fausses à partir des réelles, ce qui donne la même taille, le même ratio et le même contenu. Rajan et al. reconstruisent les réelles avec l'autoencodeur du LDM ([arXiv 2410.11835](https://arxiv.org/abs/2410.11835)). B-Free utilise l'inpainting et l'auto-conditionnement de Stable Diffusion ([arXiv 2412.17671](https://arxiv.org/abs/2412.17671)).
- **Réelles non compressées** pour l'évaluation : RAISE, utilisé par Synthbuster, B-Free et BIAS-ID. Dans Synthbuster, « both real and generated images are saved in raw format » ([B-Free, §3.1](https://arxiv.org/html/2412.17671)).

## 5. Vérifier qu'il ne reste pas de raccourci

1. **Test de décalage par transformation (protocole BIAS-ID).** ([Ricker et al.](https://arxiv.org/html/2605.31153), code : [github.com/jonasricker/biasid](https://github.com/jonasricker/biasid))
   - Appliquer *la même* transformation aux réelles et aux générées du test : JPEG de q = 100 à 10, resize ×0,5 à ×2, éventuellement passage en niveaux de gris.
   - Mesurer, pour chaque classe, le décalage moyen du score (MSS = score transformé − score original).
   - Un détecteur non biaisé a des décalages faibles, ou qui se compensent entre les classes. Un détecteur biaisé pousse *systématiquement* les deux classes vers « réel » (ou vers « généré »).
   - Les auteurs distinguent biais et manque de robustesse : « performance alone cannot always fully capture how a detector is affected by a transformation ».
2. **Contre-test sur photos réelles non compressées** ([Grommelt et al., Table 2](https://arxiv.org/html/2403.17608)).
   - Prendre des photos réelles en PNG (FFHQ ou RAISE) et les évaluer en PNG puis en JPEG q = 95, 90, 80, 60.
   - Si la précision sur ces réelles grimpe avec la compression, le modèle a appris « JPEG = réel ».
3. **Analyse par taille d'origine** ([Grommelt et al., §3.2](https://arxiv.org/html/2403.17608)) : regrouper les réelles du test par taille d'origine et vérifier que la précision ne chute pas près de la taille du générateur.
4. **Baseline triviale.** Entraîner un petit classifieur, par exemple une régression logistique, sur des features *non sémantiques* seulement : taille d'origine, ratio, facteur de qualité JPEG estimé, format. Puis faire de même sur les images après prétraitement, par exemple avec un CNN minuscule sur le résidu haute fréquence ou sur la carte des blocs 8×8.
   - Avant prétraitement, la baseline devrait atteindre presque 100 %. Après, elle devrait tomber près de 50 %.
   - **[non vérifié]** : ce test est une recommandation de bon sens dérivée des papiers ci-dessus. Je ne l'ai pas trouvé sous cette forme exacte dans une source primaire. Le préprint [Zhou & Wang](https://arxiv.org/abs/2606.20488) recommande une baseline naïve feature-kNN comme contrôle, ce qui va dans le même sens.
5. **Généralisation inter-générateurs.** Entraîner sur un générateur et tester sur les autres. Grommelt et al. montrent que le débiaisage change fortement cette matrice ([§4](https://arxiv.org/html/2403.17608)). Une précision quasi parfaite en intra-générateur qui s'effondre ailleurs est un signal d'alerte.

## 6. Ce qui en découle pour le projet (proposition)

1. Choisir un sous-ensemble de générateurs de même taille. Le trio en 512² (SD1.4, SD1.5, Wukong) est le cas étudié par Grommelt et al.
2. Filtrer les réelles ImageNet entre 450 et 550 px.
3. Recompresser **toutes** les générées en JPEG q = 96, avec le même encodeur (PIL) que celui utilisé pour l'augmentation.
4. Entrée réseau : random crop 224 à l'entraînement, center crop 224 au test, **sans resize**. En option, un resize avec le même facteur pour les deux classes.
5. Augmentation flou + JPEG aléatoire (q = 30 à 100, p = 0,1 à 0,5), appliquée aux deux classes.
6. Après entraînement : tests 1, 2 et 4 de la section 5 au minimum. Les tests 1 et 2 ne demandent qu'un petit lot d'images réelles PNG et quelques lignes de code.

Le ticket `06-pretraitement` décidera. Les points 1 à 3 reprennent le pipeline publié d'Unbiased GenImage ([dépôt](https://github.com/gendetection/UnbiasedGenImage)). Les points 4 à 6 sont une synthèse de B-Free, CNNDetection et BIAS-ID.

## Points non vérifiés (récapitulatif)

- Survie mesurée des traces JPEG après redimensionnement : pas de source primaire trouvée.
- Biais EXIF ou métadonnées dans GenImage : pas de source trouvée (sans objet ici, puisqu'on n'utilise que les pixels).
- Effet isolé du ratio d'aspect : non mesuré dans les sources lues.
- Recompresser les deux classes plutôt que les seules générées : pas de comparaison directe trouvée. Risque de double compression non quantifié.
- Le papier de Grommelt et al. est un préprint arXiv : aucune publication en conférence n'est indiquée sur la page arXiv. BIAS-ID est indiqué accepté à NeurIPS 2026 (D&B). Zhou & Wang (2026) n'a été lu qu'au niveau du résumé.

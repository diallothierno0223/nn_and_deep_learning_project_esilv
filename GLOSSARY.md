# Glossaire

**Image réelle** : photo d'une scène qui existe, prise par une personne avec un appareil photo ou un téléphone. Jamais produite par un réseau de neurones.
_À éviter_ : « image humaine » (ambigu : un dessin est fait par un humain mais n'est pas une image réelle).

**Image générée** : image entièrement produite par un modèle d'IA générative (Stable Diffusion, Midjourney, GAN…), quel que soit le modèle. Le générateur d'origine n'a pas d'importance pour la tâche.
_À éviter_ : « fake » (évoque les deepfakes, images retouchées ou montages, hors sujet ici).

**Générateur** : le modèle d'IA qui a produit une image générée (ex. Stable Diffusion v1.4, Midjourney v5).

**Tâche** : classification binaire d'une image en image réelle ou image générée.

**Générateur mis de côté** : générateur dont aucune image n'est vue à l'entraînement, utilisé uniquement pour tester si le modèle reconnaît les images générées par un générateur inconnu. Ici : Midjourney et BigGAN.
_À éviter_ : « held-out », « scaffold ».

**Expérience « tous générateurs »** : entraînement et test sur un mélange équilibré de tous les générateurs.

**Expérience « générateurs mis de côté »** : même modèle, entraîné sans les générateurs mis de côté, puis testé à la fois sur un split classique et sur les générateurs mis de côté.

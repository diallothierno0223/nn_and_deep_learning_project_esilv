# Augmentation, Grad-CAM, hyperparamètres, notebook et oral : quels choix ?

Type: grilling
Status: resolved
Blocked by: 06, 07, 08

## Question

Trancher, en un seul tour, ce qu'il restait dans le brouillard de la carte : l'augmentation de données, Grad-CAM, le budget d'hyperparamètres, la structure du notebook et le fil conducteur de l'oral.

## Answer

Validé par l'utilisateur le 2026-11-23 (« oui à tout »).

- **Augmentation de données** : rien en dehors du crop aléatoire déjà prévu. Le flou et la compression JPEG aléatoires restent une piste « pour aller plus loin » : ils pourraient effacer les traces de génération sur des crops de 128 px.
- **Grad-CAM** :
  - modèle de l'expérience « générateurs mis de côté », couche `conv5_block3_out` ;
  - environ 8 exemples : 2 images réelles, 2 générées bien classées, 2 erreurs, 2 images Midjourney ou BigGAN.
- **Hyperparamètres** : pas de recherche, on garde les valeurs fixées. Une exécution par expérience, soit 2 entraînements au total.
- **Structure du notebook** :
  1. installations ;
  2. imports ;
  3. téléchargement du dataset ;
  4. sous-ensembles et split ;
  5. prétraitement ;
  6. modèle ;
  7. expérience « tous générateurs » ;
  8. expérience « générateurs mis de côté » ;
  9. évaluation et tableau comparatif ;
  10. contrôles anti-raccourci ;
  11. Grad-CAM ;
  12. conclusion.

  Chaque section commence par un titre markdown. Les poids de chaque modèle sont sauvegardés pour pouvoir réévaluer sans réentraîner.
- **Fil conducteur de l'oral** :
  1. le problème ;
  2. le piège des biais JPEG et taille, et comment on l'évite ;
  3. la méthode ;
  4. les résultats des deux expériences ;
  5. les contrôles anti-raccourci ;
  6. Grad-CAM ;
  7. les limites.

  Le détail des slides se fera une fois les résultats obtenus.

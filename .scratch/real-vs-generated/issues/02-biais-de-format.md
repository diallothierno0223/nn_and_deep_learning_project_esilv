# Quels biais de format piègent ces datasets, et quel prétraitement les neutralise ?

Type: research
Status: open
Blocked by: —

## Question

Dans les datasets de détection d'images générées (GenImage en particulier), quels biais non sémantiques distinguent les deux classes ? On pense au format de fichier (JPEG vs PNG), aux traces de compression JPEG, à la résolution et au ratio, aux métadonnées. Quels travaux montrent qu'un classifieur exploite ces raccourcis ?

Le principe déjà acté (même taille pour toutes les images, uniquement les pixels) suffit-il ? Les traces de compression JPEG survivent dans les pixels même après redimensionnement. Quels prétraitements la littérature recommande-t-elle (recompression JPEG des deux classes, crop au lieu de resize, alignement de résolution…) ? Comment vérifier après coup qu'il ne reste plus de raccourci ?

## Recherche

En cours : les résultats arrivent sur la branche jetable `research/02-biais-de-format`, dans le fichier `research/02-biais-de-format.md` (jamais mergée).

# Parcours francais : commerce-payments (protocole Commerce Payments)

Lecture commentee du depot base/commerce-payments : un protocole permissionless qui reproduit on-chain le flux "authorize and capture" du commerce traditionnel, avec un systeme modulaire de collecteurs de tokens (ERC-3009, Permit2, pre-approbation, spend permissions) et un modele de frais borne cryptographiquement par le payeur.

Sommaire :

Chapitre 1 Presentation du protocole Commerce Payments. Chapitre 2 PaymentInfo et le hash qui identifie un paiement. Chapitre 3 TokenStore, l escrow par operateur deploye a la demande. Chapitre 4 TokenCollector, l abstraction qui rend l autorisation modulaire. Chapitre 5 authorize, placer une reserve en escrow. Chapitre 6 capture, distribuer les fonds retenus. Chapitre 7 charge, autoriser et capturer en une seule transaction. Chapitre 8 void et reclaim, les deux chemins d annulation. Chapitre 9 refund, rendre des fonds deja captures. Chapitre 10 Le modele de frais et ses bornes cryptographiques. Chapitre 11 ERC-3009 et Permit2, collecter par signature. Chapitre 12 PreApproval et SpendPermission, les deux autres voies de collecte. Chapitre 13 Limites et perimetre de ce parcours.

Ce parcours est une lecture pedagogique du code source et de la documentation du depot, sans installation ni execution du projet.

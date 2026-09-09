# Chapitre 1 -- Presentation du protocole Commerce Payments

Commerce Payments est un protocole permissionless qui reproduit on-chain le flux "authorize and capture" utilise par les processeurs de paiement traditionnels (Stripe, cartes bancaires). Au lieu de transferer les fonds en une seule fois, le protocole separe deux etapes : l autorisation, qui place une reserve sur les fonds du payeur, et la capture, qui distribue effectivement le paiement au marchand.

Ce decouplage resout un probleme concret du commerce : au moment ou une commande est passee, le marchand ne sait pas toujours si elle sera honoree (stock disponible, fraude, annulation). Une autorisation reserve les fonds sans les deplacer definitivement, laissant au marchand une fenetre pour confirmer la commande avant de capturer le paiement -- ou pour l annuler sans jamais avoir touche aux fonds du payeur.

Trois roles distincts interagissent dans ce protocole. Le payer autorise une depense, generalement via une signature hors-chaine, sans avoir besoin d envoyer lui-meme de transaction. L operateur est l entite qui pilote le flux de paiement : c est lui qui appelle authorize, capture, void ou refund sur le contrat. Le protocole ne lui fait cependant pas une confiance aveugle -- chaque action reste bornee par les parametres cryptographiquement approuves par le payeur au depart (montant maximum, fenetres de temps, plafonds de frais). Le receiver, enfin, est l adresse qui recoit le paiement une fois capture.

Le contrat central, AuthCaptureEscrow, ne connait qu un seul type d objet : un PaymentInfo, une structure immuable qui identifie de maniere unique un paiement donne et fixe toutes les bornes que l operateur devra respecter. Ce parcours suit ce contrat chapitre par chapitre, ainsi que le systeme modulaire de collecteurs de tokens qui lui permet de rester agnostique de la methode d autorisation utilisee par le payeur (signature ERC-3009, Permit2, allowance classique, ou spend permission de smart wallet).

Fichier central : `src/AuthCaptureEscrow.sol`.

[Chapitre suivant : PaymentInfo et le hash qui identifie un paiement](02-paymentinfo-hash.md)

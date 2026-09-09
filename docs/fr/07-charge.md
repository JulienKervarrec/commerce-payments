# Chapitre 7 -- charge, autoriser et capturer en une seule transaction

Tous les paiements ne beneficient pas du flux en deux etapes : un achat numerique instantane n a par exemple aucune raison de passer par une phase d autorisation separee suivie d une capture ulterieure. charge() existe pour ce cas, en combinant authorize et capture dans un seul appel, avec un seul evenement PaymentCharged plutot que la paire PaymentAuthorized puis PaymentCaptured.

La sequence interne suit exactement le meme schema de validation que les deux fonctions separees -- _validatePayment, puis _validateFee, puis verification de l unicite via hasCollectedPayment -- mais l etat final ecrit directement refundableAmount au montant charge, avec capturableAmount laisse a zero : le paiement passe immediatement au statut "capture", sans jamais transiter par un etat intermediaire "autorise". Cela signifie qu un paiement charge ne peut jamais etre voide (void) ni reclame (reclaim) par le payeur -- ces deux operations n existent que pour les montants encore capturables. Seul un refund reste possible, dans la fenetre refundExpiry.

L ordre des operations internes merite attention : l etat est ecrit et l evenement emis avant que _collectTokens ne soit appelee, respectant le meme pattern checks-effects-interactions que le reste du contrat, puis _distributeTokens envoie effectivement les fonds au receiver et au feeReceiver. Entre la collecte et la distribution, les tokens transitent physiquement par le TokenStore de l operateur -- charge ne raccourcit donc pas le chemin des fonds, elle raccourcit seulement le nombre d appels necessaires cote operateur pour obtenir le meme resultat final qu un authorize immediatement suivi d un capture au montant maximal.

Fichier central : `src/AuthCaptureEscrow.sol` (fonction charge).

[Chapitre suivant : void et reclaim, les deux chemins d annulation](08-void-reclaim.md)

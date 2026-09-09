# Chapitre 5 -- authorize, placer une reserve en escrow

authorize() est le point d entree qui place des fonds en attente de capture. Seul l operateur designe dans paymentInfo.operator peut l appeler (modifier onlySender), et le montant demande ne peut ni etre nul ni depasser paymentInfo.maxAmount ni les autres contraintes verifiees par _validatePayment (fenetres de temps, plafonds de frais coherents).

La fonction verifie ensuite que ce paiement precis -- identifie par son hash -- n a jamais ete collecte auparavant (hasCollectedPayment). Cette verification transforme chaque PaymentInfo en objet a usage unique : une fois qu authorize ou charge a ete appele avec succes pour un hash donne, aucun nouvel appel a authorize ou charge sur ce meme hash ne peut plus reussir, meme si le montant initialement autorise a ensuite ete entierement voide ou reclame. Pour reautoriser un paiement, il faut changer un champ du PaymentInfo -- typiquement le salt -- ce qui produit un nouveau hash et donc un nouvel objet de paiement.

L etat est mis a jour avant tout transfert externe (pattern checks-effects-interactions) : hasCollectedPayment passe a true et capturableAmount est fixe au montant demande, refundableAmount restant a zero puisqu aucune capture n a encore eu lieu. L evenement PaymentAuthorized est emis avec le PaymentInfo complet, ce qui permet a des indexeurs off-chain de reconstruire l historique complet d un paiement sans avoir besoin de connaitre le PaymentInfo a l avance -- seul le hash suffit ensuite pour interroger l etat.

Le transfert effectif des fonds est delegue a _collectTokens, qui invoque le TokenCollector choisi par l operateur avec le type attendu CollectorType.Payment. C est cette etape qui, en pratique, tire les tokens du payeur vers le TokenStore de l operateur -- authorize elle-meme ne fait que comptabiliser et declencher ce transfert, sans jamais manipuler directement les tokens ERC-20.

Fichier central : `src/AuthCaptureEscrow.sol` (fonction authorize).

[Chapitre suivant : capture, distribuer les fonds retenus](06-capture.md)

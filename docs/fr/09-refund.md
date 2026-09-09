# Chapitre 9 -- refund, rendre des fonds deja captures

Une fois des fonds captures, void et reclaim ne s appliquent plus -- seul refund permet de les restituer au payeur, et uniquement avant refundExpiry, la derniere des trois echeances du PaymentInfo. Passe ce delai, un paiement capture devient definitif du point de vue du protocole ; toute compensation ulterieure devrait alors passer par un mecanisme hors-protocole.

refund() est reservee a l operateur et bornee par le montant refundableAmount reellement disponible pour ce hash -- impossible de rembourser plus que ce qui a effectivement ete capture et pas deja rembourse (RefundExceedsCapture sinon). Le montant rembourse est soustrait de refundableAmount avant tout transfert, empechant un meme montant d etre rembourse deux fois meme en cas de reentrance.

La particularite de refund par rapport a void ou reclaim est qu elle appelle _collectTokens exactement comme authorize ou charge, avec cette fois CollectorType.Refund plutot que Payment -- les fonds a rembourser ne proviennent pas necessairement du TokenStore lui-meme, ils peuvent etre retires directement du compte de l operateur (via OperatorRefundCollector, chapitre 12) ou d une source de liquidite externe geree par un collecteur personnalise. Une fois les fonds collectes dans le TokenStore, _sendTokens les redirige immediatement vers paymentInfo.payer -- le remboursement transite donc par l escrow meme s il ne provient pas des fonds originellement retenus.

Cette conception decouple deliberement l origine des fonds de remboursement du flux de capture initial : un operateur peut par exemple choisir de rembourser depuis sa propre tresorerie plutot que d attendre une reconciliation avec un fournisseur tiers, tout en gardant le meme contrat d escrow comme point de passage verifiable pour chaque remboursement.

Fichier central : `src/AuthCaptureEscrow.sol` (fonction refund).

[Chapitre suivant : le modele de frais et ses bornes cryptographiques](10-frais.md)

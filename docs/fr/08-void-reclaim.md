# Chapitre 8 -- void et reclaim, les deux chemins d annulation

Un montant autorise mais jamais capture doit toujours pouvoir revenir au payeur -- c est une des garanties centrales du protocole. Deux fonctions y menent, void et reclaim, qui different uniquement par qui peut les appeler et quand.

void() est reservee a l operateur (onlySender(paymentInfo.operator)) et n a aucune contrainte de delai : l operateur peut annuler une autorisation a tout moment, par exemple des qu il sait qu une commande ne sera pas honoree (rupture de stock, fraude detectee). La fonction verifie simplement que capturableAmount est non nul (ZeroAuthorization sinon), le remet a zero, emet PaymentVoided, puis renvoie l integralite du montant au payer via _sendTokens.

reclaim() est reservee au payer lui-meme (onlySender(paymentInfo.payer)), et seulement apres authorizationExpiry -- avant cette echeance, le payeur ne peut pas reclamer unilateralement des fonds que l operateur pourrait encore vouloir capturer legitimement. C est le filet de securite du payeur contre un operateur inactif ou disparu : si l operateur ne capture ni ne voide a temps, le payeur reprend le controle de ses fonds des que la fenetre d autorisation se ferme, sans avoir besoin de la moindre cooperation de l operateur.

Les deux fonctions partagent la meme mecanique finale -- remise a zero de capturableAmount et appel a _sendTokens vers paymentInfo.payer -- mais restent des chemins strictement distincts dans le code, chacun avec sa propre garde d acces et sa propre condition temporelle. Cette separation reflete une asymetrie de confiance deliberee : l operateur a un pouvoir d annulation immediat parce qu il pilote le flux, tandis que le payeur n a qu un pouvoir d annulation differe, qui ne s active que si l operateur n a pas agi dans les temps.

Fichier central : `src/AuthCaptureEscrow.sol` (fonctions void et reclaim).

[Chapitre suivant : refund, rendre des fonds deja captures](09-refund.md)

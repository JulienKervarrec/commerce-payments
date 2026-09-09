# Chapitre 10 -- Le modele de frais et ses bornes cryptographiques

Le protocole integre un mecanisme de frais natif, pense pour des marketplaces ou des plateformes qui prelevent une commission sur chaque transaction sans avoir besoin d un contrat separe. Chaque PaymentInfo porte deux bornes en points de base, minFeeBps et maxFeeBps, ainsi qu un feeReceiver optionnel -- ces trois valeurs sont fixees au moment ou le payeur signe son autorisation, et deviennent donc des garanties cryptographiques que l operateur ne peut pas depasser plus tard.

_validateFee, appelee a chaque capture et charge, calcule minFee et maxFee en multipliant le montant de l operation par les bps correspondants puis en divisant par 10 000 (_MAX_FEE_BPS). Le feeAmount propose par l operateur doit tomber dans cet intervalle inclusif, sinon FeeAmountOutOfRange fait revert avec les trois valeurs pour faciliter le diagnostic. Un feeReceiver a zero n est autorise que si feeAmount est lui-meme nul (ZeroFeeReceiver sinon) -- impossible de prelever des frais sans destination valide.

Le champ feeReceiver de PaymentInfo introduit une nuance importante : s il est non-nul, il fige definitivement l adresse qui recevra les frais, et tout appel avec une adresse differente revert (InvalidFeeReceiver). S il est laisse a zero au moment de la signature, en revanche, l operateur regagne la liberte de choisir n importe quelle adresse de destination a chaque capture -- un choix delibere qui permet a un payeur d approuver un paiement sans connaitre a l avance l identite exacte du destinataire des frais, tout en gardant le controle total du pourcentage preleve.

_distributeTokens applique enfin le partage : la part de frais est envoyee en premier au feeReceiver si non nulle, puis le solde (amount moins feeAmount) est envoye au receiver principal. Les deux transferts passent par le meme chemin _sendTokens depuis le TokenStore de l operateur, garantissant que la somme des deux parts egale toujours exactement le montant capture ou charge.

Fichier central : `src/AuthCaptureEscrow.sol` (fonctions _validateFee et _distributeTokens).

[Chapitre suivant : ERC-3009 et Permit2, collecter par signature](11-collecteurs-signature.md)

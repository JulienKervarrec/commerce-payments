# Chapitre 2 -- PaymentInfo et le hash qui identifie un paiement

Tout paiement dans ce protocole est entierement decrit par une structure PaymentInfo, definie dans AuthCaptureEscrow.sol. Elle regroupe l operateur, le payeur, le receiver, le token, un maxAmount (uint120, le plafond que l operateur ne pourra jamais depasser), trois timestamps d expiration (preApprovalExpiry, authorizationExpiry, refundExpiry), une fourchette de frais (minFeeBps, maxFeeBps), un feeReceiver optionnel, et un salt pour garantir l unicite du hash meme entre deux paiements identiques par ailleurs.

Ces trois timestamps forment une chaine de contraintes strictement ordonnee, verifiee a chaque operation par _validatePayment : preApprovalExpiry <= authorizationExpiry <= refundExpiry. Le premier borne la fenetre pendant laquelle le payeur peut encore autoriser le paiement de depart (authorize ou charge). Le second borne la fenetre pendant laquelle l operateur peut encore capturer des fonds deja places en escrow. Le troisieme borne la fenetre pendant laquelle un paiement capture peut encore etre rembourse. Toute violation de cet ordre fait revert avec InvalidExpiries des la premiere operation.

getHash(paymentInfo) calcule l identifiant unique d un paiement : keccak256 du PaymentInfo encode avec un typehash constant (PAYMENT_INFO_TYPEHASH), puis ce hash intermediaire est lui-meme rehache avec block.chainid et address(this). Cette double couche garantit qu un meme PaymentInfo produit des hashes differents sur deux chaines differentes ou avec deux deploiements differents du contrat -- une meme signature ne peut donc jamais etre rejouee au mauvais endroit.

Toute la logique du contrat s articule autour de ce hash comme cle primaire : paymentState[paymentInfoHash] est l unique mapping de stockage du contrat, associant a chaque hash un PaymentState (hasCollectedPayment, capturableAmount, refundableAmount). Le PaymentInfo complet, lui, n est jamais stocke on-chain -- il doit etre fourni en calldata a chaque appel, et c est au hash de garantir que l operateur ne peut pas le modifier en cours de route sans invalider toutes les autorisations deja collectees.

Fichier central : `src/AuthCaptureEscrow.sol` (struct PaymentInfo, fonction getHash).

[Chapitre suivant : TokenStore, l escrow par operateur deploye a la demande](03-tokenstore.md)

# Chapitre 3 -- TokenStore, l escrow par operateur deploye a la demande

Les fonds en escrow ne restent jamais dans le contrat AuthCaptureEscrow lui-meme : chaque operateur possede son propre contrat TokenStore, un escrow minimaliste dont l unique fonction sendTokens ne peut etre appelee que par le singleton AuthCaptureEscrow (verifie via l immutable authCaptureEscrow fixe au constructeur). Cette segmentation isole les fonds d un operateur de ceux de tous les autres : un bug ou une compromission touchant un operateur ne met pas en danger les escrows des autres.

Le TokenStore d un operateur n est pas deploye a l avance -- son adresse est simplement predictible. getTokenStore(operator) calcule une adresse deterministe via LibClone.predictDeterministicAddress, un clone minimal (EIP-1167) de tokenStoreImplementation avec pour salt l adresse de l operateur elle-meme (bytes32(bytes20(operator))). N importe qui peut donc calculer a l avance l adresse du TokenStore d un operateur, meme avant qu il n existe on-chain, et y envoyer des tokens en toute confiance.

Le deploiement effectif est paresseux et se produit dans _sendTokens : le contrat tente d abord un appel direct sur l adresse predite ; si cet appel echoue parce que le TokenStore n existe pas encore (tokenStore.code.length == 0), le contrat le deploie a la volee via LibClone.cloneDeterministic avec le meme salt, emet TokenStoreCreated, puis reessaie l appel. Ce mecanisme evite un deploiement explicite et un cout de gas initial pour chaque nouvel operateur -- le premier retrait declenche simplement une etape de deploiement supplementaire, invisible pour l appelant.

Un detail de robustesse : _sendTokens ne se contente pas de verifier que l appel a reussi, il verifie aussi que la donnee retournee decode bien en true (abi.decode(returnData, (bool))). Si l appel echoue pour une autre raison qu une absence de code (un revert explicite dans TokenStore, par exemple), les donnees d erreur sont propagees telles quelles via un bloc assembly qui relance le revert d origine, preservant le message d erreur exact.

Fichiers centraux : `src/TokenStore.sol`, `src/AuthCaptureEscrow.sol` (fonctions getTokenStore et _sendTokens).

[Chapitre suivant : TokenCollector, l abstraction qui rend l autorisation modulaire](04-tokencollector.md)

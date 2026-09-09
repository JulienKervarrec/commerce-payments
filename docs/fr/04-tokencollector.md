# Chapitre 4 -- TokenCollector, l abstraction qui rend l autorisation modulaire

AuthCaptureEscrow ne sait pas, et n a pas besoin de savoir, comment un payeur a autorise la depense de ses tokens. Cette responsabilite est entierement deleguee a un contrat TokenCollector externe, choisi par l operateur a chaque appel d authorize, charge ou refund. Le contrat abstrait TokenCollector definit le contrat d interface commun : une fonction collectTokens, appelable uniquement par le singleton AuthCaptureEscrow (verifie via onlySender-like check sur msg.sender), qui delegue immediatement a une implementation interne _collectTokens propre a chaque collecteur concret.

Chaque collecteur declare aussi un collectorType constant, Payment ou Refund, verifie par _collectTokens dans AuthCaptureEscrow avant tout appel -- un collecteur de type Refund ne peut jamais etre utilise pour une operation authorize ou charge, et inversement. Cette verification empeche un operateur malveillant de detourner un collecteur concu pour rembourser afin de l utiliser comme mecanisme de prelevement initial, ou vice versa.

Apres l appel au collecteur, AuthCaptureEscrow verifie lui-meme que le solde du TokenStore a bien augmente exactement du montant attendu (mesure du solde avant et apres l appel, comparaison stricte). Cette verification independante du resultat rend le contrat resilient a un collecteur bogue ou malicieux qui pretendrait avoir transfere des fonds sans le faire reellement : si le solde ne correspond pas, TokenCollectionFailed fait revert la transaction entiere, quel que soit ce que le collecteur a raconte.

Le protocole fournit cinq collecteurs concrets, que les chapitres suivants detaillent un par un : ERC3009PaymentCollector et Permit2PaymentCollector s appuient sur des signatures hors-chaine (avec support des wallets contractuels non deployes via ERC-6492), PreApprovalPaymentCollector s appuie sur un appel prealable du payeur plus une allowance ERC-20 classique, SpendPermissionPaymentCollector s appuie sur le systeme de spend permissions des smart wallets Base Account, et OperatorRefundCollector, le seul de type Refund, s appuie sur une allowance donnee par l operateur lui-meme.

Fichier central : `src/collectors/TokenCollector.sol`.

[Chapitre suivant : authorize, placer une reserve en escrow](05-authorize.md)

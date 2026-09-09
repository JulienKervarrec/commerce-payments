# Chapitre 12 -- PreApproval et SpendPermission, les deux autres voies de collecte

PreApprovalPaymentCollector suit un schema en deux temps distinct des collecteurs par signature : le payeur doit d abord appeler lui-meme preApprove(paymentInfo) on-chain (une transaction classique, pas une signature hors-chaine), qui enregistre isPreApproved[hash] = true apres avoir verifie que l appelant est bien le payer attendu et que preApprovalExpiry n est pas depasse. Au moment de la collecte effective, le collecteur verifie simplement ce flag puis transfere les tokens via une allowance ERC-20 standard prealablement accordee par le payeur a ce collecteur. Ce mode convient a des integrations ou une signature EIP-712 hors-chaine n est pas pratique -- un payeur interagissant directement avec une interface on-chain, par exemple.

SpendPermissionPaymentCollector, lui, s appuie sur le systeme de spend permissions des smart wallets Base Account (le meme mecanisme documente dans le parcours account-sdk de cette bibliotheque). Il construit une SpendPermission complete a partir des champs du PaymentInfo -- account devient le payer, spender devient le collecteur lui-meme, allowance devient maxAmount, et le salt integre a nouveau le hash payer-agnostique du paiement. Si une signature est fournie dans collectorData, le collecteur l utilise pour approuver la permission au vol via approveWithSignature avant de la depenser ; sinon, il suppose que la permission a deja ete approuvee au prealable par un autre chemin.

Ce collecteur integre egalement un support optionnel de MagicSpend : si collectorData contient une WithdrawRequest encodee en plus de la signature, la depense passe par spendWithWithdraw plutot que par le simple spend, permettant de retirer des fonds d un paymaster MagicSpend au moment meme de la depense -- utile lorsque le smart wallet du payeur ne detient pas encore les fonds necessaires sur la chaine consideree.

OperatorRefundCollector, presente au chapitre 9, complete ce tableau comme seul collecteur de type Refund parmi les cinq : il transfere simplement des tokens depuis le compte de l operateur lui-meme vers le TokenStore, via une allowance ERC-20 que l operateur a prealablement accordee a ce collecteur precis.

Fichiers centraux : `src/collectors/PreApprovalPaymentCollector.sol`, `src/collectors/SpendPermissionPaymentCollector.sol`, `src/collectors/OperatorRefundCollector.sol`.

[Chapitre suivant : limites et perimetre de ce parcours](13-limites-perimetre.md)

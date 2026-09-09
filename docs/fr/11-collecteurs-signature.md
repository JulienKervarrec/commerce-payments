# Chapitre 11 -- ERC-3009 et Permit2, collecter par signature

Deux des cinq collecteurs s appuient sur une signature hors-chaine du payeur plutot que sur une allowance ERC-20 classique, evitant ainsi une transaction d approbation prealable separee. ERC3009PaymentCollector s appuie sur la norme ERC-3009 (receiveWithAuthorization), directement implementee par des tokens comme USDC : la signature autorise un transfert vers ce collecteur pour un montant plafonne a maxAmount, avec pour nonce le hash du paiement calcule sans le champ payer (_getHashPayerAgnostic) -- une meme signature reste ainsi valable independamment de la valeur exacte inscrite dans le champ payer au moment ou elle a ete produite hors-chaine.

Le collecteur ERC-3009 recoit toujours l integralite de maxAmount dans son propre solde, puis retourne l excedent au payeur si le montant reellement demande est inferieur (cas d une capture partielle), avant de transferer le montant net vers le TokenStore. Permit2PaymentCollector suit une logique tres proche mais s appuie sur le protocole Permit2 d Uniswap plutot que sur une capacite native du token : il fonctionne donc avec n importe quel ERC-20, y compris ceux qui n implementent pas ERC-3009, au prix d une dependance externe au singleton Permit2 deploye sur la chaine.

Les deux collecteurs heritent d ERC6492SignatureHandler, qui gere un cas particulier important : un payeur peut etre un smart wallet pas encore deploye au moment de la signature (contre-factuel). Le suffixe magique ERC-6492 (0x6492...6492) permet d encoder, en plus de la signature elle-meme, l adresse et les donnees d un appel de deploiement a executer avant la verification -- _handleERC6492Signature detecte ce suffixe, decode l appel de deploiement, et le declenche via un Multicall3 public plutot que directement, pour eviter qu un contrat malicieux ne puisse abuser de l identite de l escrow comme appelant.

Fichiers centraux : `src/collectors/ERC3009PaymentCollector.sol`, `src/collectors/Permit2PaymentCollector.sol`, `src/collectors/ERC6492SignatureHandler.sol`.

[Chapitre suivant : PreApproval et SpendPermission, les deux autres voies de collecte](12-collecteurs-approbation.md)

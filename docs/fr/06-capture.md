# Chapitre 6 -- capture, distribuer les fonds retenus

capture() transforme des fonds precedemment autorises en un paiement effectif au receiver. L operation peut etre appelee plusieurs fois pour un meme paiement, jusqu a epuisement du montant capturable cumule -- un cas d usage frequent en commerce, ou une commande peut etre expediee et facturee en plusieurs lots (capture partielle a l expedition de chaque colis, par exemple).

Deux verifications temporelles et de solde encadrent l operation. D abord, le timestamp courant doit rester strictement avant authorizationExpiry -- passe ce delai, plus aucune capture n est possible et seul un reclaim par le payeur peut recuperer les fonds restants (chapitre 8). Ensuite, le montant demande ne peut pas depasser capturableAmount actuellement en escrow pour ce hash ; sinon la fonction revert avec InsufficientAuthorization, qui inclut le montant disponible et le montant demande pour faciliter le debogage cote operateur.

La mise a jour d etat est symetrique et transparente : le montant capture est soustrait de capturableAmount et ajoute a refundableAmount. Ce glissement d un compteur vers l autre materialise exactement le changement de statut du paiement -- des fonds qui pouvaient encore etre annules (void) redeviennent des fonds effectivement payes, qui pourront desormais seulement etre rembourses (refund), jamais annules directement.

_validateFee est appelee ici comme dans charge, avant tout changement d etat, pour verifier que le feeAmount propose par l operateur reste dans les bornes minFeeBps/maxFeeBps approuvees par le payeur au moment de la signature initiale -- un operateur ne peut donc jamais capturer avec des frais plus eleves que ce que le payeur a explicitement autorise, meme si l operateur controle entierement le moment et le montant de la capture.

Fichier central : `src/AuthCaptureEscrow.sol` (fonction capture).

[Chapitre suivant : charge, autoriser et capturer en une seule transaction](07-charge.md)

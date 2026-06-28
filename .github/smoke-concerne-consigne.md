# Consigne — Champ `concerne` dans `smoketest.json` (repo Plateforme-CPTS)

> À coller dans une session Claude sur le repo **Evalita03/Plateforme-CPTS**.
> Elle est **autonome** : elle ne suppose aucune connaissance du cockpit Strategy.
> But côté cockpit : classer chaque anomalie par **nature** pour la router vers la
> bonne expertise (correction produit / sécurité → Iris / RGPD → Cléo).

---

```
CONSIGNE — Ajouter le champ "concerne" à chaque anomalie de smoketest.json

CONTEXTE
Tu produis déjà le fichier smoketest.json (rapport des smoke tests), qui contient
un tableau "anomalies". On veut désormais classer chaque anomalie par NATURE, pour
pouvoir la confier plus tard à la bonne expertise : correction produit, sécurité,
ou conformité RGPD. Ce classement doit se déduire de l'anomalie elle-même.

CE QU'IL FAUT FAIRE
Sur CHAQUE objet du tableau "anomalies", ajoute un champ "concerne" dont la valeur
est EXACTEMENT l'une de ces trois chaînes (en minuscules, sans accent) :

- "fonctionnel"  Le produit est cassé ou trop fermé : une fonction ne marche pas,
                 un accès LÉGITIME est refusé à tort, un calcul ou un chiffre est
                 incohérent, une page renvoie une erreur. Rien n'est sur-exposé.
                 (C'est la valeur par défaut.)

- "securite"     Un contrôle d'accès laisse passer ce qu'il ne devrait pas : un
                 rôle peut atteindre une route, une action ou une fonction qu'il ne
                 devrait PAS pouvoir atteindre (sur-autorisation, action non permise).
                 Risque = accès non autorisé. SANS donnée personnelle en jeu.

- "rgpd"         L'anomalie expose des DONNÉES PERSONNELLES ou de SANTÉ au mauvais
                 destinataire : données d'un membre/patient visibles par le mauvais
                 rôle, données d'une CPTS visibles par une autre, infos sensibles
                 accessibles sans habilitation.

RÈGLE DE DÉCISION (applique-la dans cet ordre)
1. Des données personnelles ou de santé sont exposées au mauvais destinataire ? -> "rgpd"
2. Sinon, un rôle peut accéder/faire quelque chose qu'il ne devrait pas (sans
   donnée perso) ? -> "securite"
3. Sinon -> "fonctionnel"

EXEMPLES
- "coordinateur bloqué à tort sur /membres" (accès légitime refusé)        -> "fonctionnel"
- "le total du circuit de l'argent ne correspond pas à la somme des lignes" -> "fonctionnel"
- "un secrétaire peut ouvrir /finances/cloture, réservé au coordinateur"    -> "securite"
- "un membre simple peut supprimer un autre membre"                         -> "securite"
- "un coordinateur voit la liste des membres d'une AUTRE CPTS"              -> "rgpd"
- "un rôle non habilité accède aux coordonnées/infos de santé d'un patient" -> "rgpd"

CONTRAINTES (impératives)
- "concerne" est OBLIGATOIRE sur toutes les anomalies, y compris les anciennes,
  dès la prochaine passe. En cas de doute réel, mets "fonctionnel".
- Emploie exactement ces valeurs, minuscules et sans accent : "securite" (pas
  "sécurité"), "rgpd", "fonctionnel".
- N'ajoute QUE ce champ. Ne change, ne renomme et ne supprime aucun autre champ
  du fichier : tout le reste de smoketest.json doit rester identique.
- Le fichier doit rester un JSON strictement valide.

EXEMPLE D'UNE ANOMALIE APRÈS AJOUT
{
  "id": "SMK-2026-06-27-01",
  "severite": "important",
  "titre": "Accès refusé à tort /membres",
  "module": "vie_associative",
  "ecran": "/membres",
  "role": "coordinateur",
  "statut": "ouvert",
  "detecte_le": "2026-06-27",
  "resume": "coordinateur devrait accéder à vie_associative mais est bloqué.",
  "concerne": "fonctionnel"
}
```

---

## Côté cockpit (repo Strategy) — pour mémoire

- Tant que la ligne « dont X sécurité / Y données » n'est pas activée sous le feu de
  Smoke, le champ `concerne` est **simplement ignoré** : l'ajouter ne casse rien.
- Quand l'agent plateforme commence à étiqueter, activer cette ligne discrète sous le
  feu (elle compte les anomalies `securite` → Iris et `rgpd` → Cléo).
- Le jour où **Iris** et **Cléo** sont créés, Smoke leur transmet ces points
  (même principe que Mako alimenté par Orion). L'historique sera déjà qualifié.

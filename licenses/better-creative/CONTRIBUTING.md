# Contributing to Better Creative

Thanks for wanting to help. Better Creative is **proprietary software** and its
source code is **not published**. There is no public contribution workflow:
this guide is for the people THEFricadelle has given access to the source, and it
explains exactly what that access allows.

Read [LICENSE](LICENSE) for the binding terms. This file is a plain-language
guide, not a substitute for it.

## Access to the source

Access to the source code is given only by THEFricadelle, or with their written
authorization, to people invited to contribute (Section 5.1 of the LICENSE).
It lets you read the code and prepare contributions. It does not let you:

- **share the source** with anyone who does not hold the same access;
- **use it for anything other than contributing**, including reusing it,
  verbatim or adapted, inside another project;
- **publish any build** made from it — no .jar, no source
  archives, nothing on mod-hosting sites, modpack platforms, Discord, or anywhere else;
- **keep using it** once the access is withdrawn, which can happen at any time;
- **claim authorship**, or remove or alter any copyright, authorship, or license
  notice — SPDX headers included.

Using AI-assisted tooling to write your contribution is fine — the §3(h)
restriction targets using the codebase as training data, not your editor.

## Contributor terms (important)

By submitting a pull request, patch, or code suggestion, you agree that:

1. You grant THEFricadelle a perpetual, worldwide, irrevocable, royalty-free,
   sublicensable and transferable license to use, modify, relicense, and
   distribute your contribution as part of Better Creative, under this or any
   other license.
2. You are the author of the contribution, you have the right to submit it, and
   it contains no third-party code you are not entitled to submit.
3. You keep the copyright on your own contribution. Submitting it gives you
   **no ownership, no co-authorship, and no right to redistribute, publish,
   mirror, relicense, or fork Better Creative**. Your permissions stay exactly
   those of anyone else under §§2 and 3 — having a PR merged never enlarges
   them.
4. Where the law allows it, you waive your moral rights in the contribution as
   against the author; where it does not, you agree not to assert them in a way
   that would block the license above. In return, your contribution will never
   be misattributed to someone else.
5. These terms apply identically whether or not you are a member of the team or
   organization hosting the repository. Membership, maintainer status, and write
   access grant no right over Better Creative and no authority to permit
   anything the LICENSE reserves to THEFricadelle (§1).

## What you get in return

Section 5.3 of the LICENSE gives every contributor two things:

- **Credit** in [CONTRIBUTORS.md](CONTRIBUTORS.md), under the name or handle you
  chose and naming what you actually contributed, never withdrawn later for any
  reason. Ask via the issue tracker for a different name, no contact address, or
  no listing at all.
- **The modpack permission**, confirmed explicitly: you may ship
  Better Creative in a modpack you publish — Official Channel reference,
  unmodified official file, notices preserved. Having had access to the source
  never costs you this. It covers the official file only, never a build of
  your own.

That credit recognizes your work. It does not make you a co-owner or
co-maintainer, and it grants no right to redistribute Better Creative.

If you do not agree with these terms, do not submit a pull request — open an
issue describing the problem instead. That is just as useful.

## Contributions that are welcome

| Type | Welcome | Notes |
|------|---------|-------|
| Bug fixes | ✅ Yes | The best kind of PR. Include reproduction steps. |
| Crash fixes | ✅ Yes | Attach the crash report or stack trace. |
| Compatibility fixes | ✅ Yes | JEI, EMI, ModernFix, or any mod that also touches the creative screen. |
| Performance improvements | ✅ Yes | Explain the measurement, not just the theory. |
| Typos, localization fixes | ✅ Yes | Small and easy to merge. Both `en_us.json` and `fr_fr.json`. |
| New translations | ✅ Yes | A new `assets/bettercreative/lang/<locale>.json`, complete. |
| Documentation corrections | ✅ Yes | README, guides, comments. |
| New features | ⚠️ Ask first | Open an issue before writing code — I may already have a design or a reason to refuse it. |
| Refactors / restyling | ⚠️ Ask first | Large diffs with no behavior change are usually rejected. |
| Dependency or build changes | ⚠️ Ask first | Affects distribution and the release pipeline. |

## How to submit a pull request

1. **Open an issue first** for anything beyond a small fix — it avoids wasted
   work on both sides.
2. **Branch** from `dev` in the official repository.
3. **Name the branch** `fix/short-description` or `feat/short-description`.
4. **Write the code** following the conventions below.
5. **Build and test**:
   ```bash
   ./gradlew testAll    # unit tests, game tests on a dedicated server, live client run
   ```
   Everything must pass. A PR that does not build will not be reviewed.
   `testAll` runs the unit tests, the game tests on a dedicated server, and a
   real client that drives the creative screen and writes
   `run/clientsmoke/smoke-report.txt`. That last run opens a game window: leave
   it alone until it closes, a click inside it can break a check. Requires JDK
   21; the first build downloads and decompiles Minecraft, expect several
   minutes.
6. **Commit** with a conventional message: `fix: keep hidden tabs out of the search aggregate`.
7. **Open the pull request against `dev`**, describing what it
   fixes and how you verified it.

## Code conventions

- **Language**: all code, identifiers, comments, and log messages in **English**.
- **Naming**: `PascalCase` classes, `camelCase` methods and fields, `UPPER_SNAKE_CASE` constants.
- **User-facing strings** go through the lang files, never hardcoded, and land
  in **both** `en_us.json` and `fr_fr.json` in the same change.
- **Mixin injector methods** must be prefixed `bettercreative$` so they
  cannot collide with another mod's injectors in the same target class.
- **Prefer `@ModifyExpressionValue` over `@Redirect`** — far better multi-mod
  compatibility.
- **Ordering logic stays in `TabOrderResolver`**, a pure function of (game list,
  config), so it can be reasoned about without launching the game.
- **Comments**: minimal and in English — explain *why*, not *what*.
- **SPDX headers**: keep the existing header on every source file. New files
  must carry the same header.
- **No version bumps**: never change `mod_version` (`gradle.properties`). Releases are handled
  by the author.
- **No new dependencies** without asking first.
- **Scope**: one logical change per pull request.

`RULES.md` lists the project-specific traps in full — most importantly: never
mutate a `CreativeModeTab` or the registry (they are shared singletons), never
identify a tab by index, never let the resolver return an empty list, and never
drop config entries belonging to a mod that is not currently installed.

## Reporting a bug

Include, at minimum:

- Better Creative version, Minecraft version, NeoForge version.
- Whether JEI, EMI, ModernFix or another creative-screen mod is installed, and
  their versions.
- Steps to reproduce.
- The relevant log excerpt or crash report (use a paste service for long logs).
- Your `config/better-creative/client.toml` if the problem is
  about ordering, pinning, or hiding.

If the report concerns the **search scope** option, say whether you tested in a
real modpack or in the dev client — ModernFix can skip the hook that a vanilla
run exercises fine, and that difference has already produced two false
"it works" results.

## Contact

For redistribution requests, modpack inclusion beyond Section 3(b), commercial distribution, or anything else not
covered here, use the official issue tracker:

  https://github.com/THEFricadelle/mc-mods-issues/issues/new?template=permission-request.yml

**Author: THEFricadelle**

---

# Contribuer à Better Creative (Version Française)

Merci de vouloir aider. Better Creative est un **logiciel propriétaire** et son
code source **n'est pas publié**. Il n'y a pas de contribution publique : ce
guide s'adresse aux personnes à qui THEFricadelle a donné accès au code, et il
explique précisément ce que cet accès permet.

Lisez [LICENSE](LICENSE) pour les conditions contraignantes. Ce fichier est un
guide en langage clair, pas un substitut.

## Accès au code source

L'accès au code source n'est donné que par THEFricadelle, ou avec son autorisation
écrite, aux personnes invitées à contribuer (Section 5.1 de la LICENSE). Il vous
permet de lire le code et de préparer des contributions. Il ne vous permet pas :

- **de partager le code** avec quiconque ne dispose pas du même accès ;
- **de l'utiliser à autre chose que contribuer**, y compris de le réutiliser,
  tel quel ou adapté, dans un autre projet ;
- **de publier un build** qui en est issu — aucun .jar, aucune
  archive source, rien sur sites d'hébergement de mods, plateformes de modpacks, Discord ou ailleurs ;
- **de continuer à l'utiliser** une fois l'accès retiré, ce qui peut arriver à
  tout moment ;
- **de revendiquer la paternité**, ni de supprimer ou altérer une mention de
  copyright, de paternité ou de licence — en-têtes SPDX compris.

Utiliser des outils assistés par IA pour rédiger votre contribution ne pose
aucun problème — la restriction du §3(h) vise l'usage du code comme données
d'entraînement, pas votre éditeur.

## Conditions applicables aux contributeurs (important)

En soumettant une pull request, un patch ou une suggestion de code, vous
acceptez que :

1. Vous accordez à THEFricadelle une licence perpétuelle, mondiale, irrévocable,
   gratuite, sous-licenciable et transférable pour utiliser, modifier,
   relicencier et distribuer votre contribution au sein de Better Creative,
   sous cette licence ou toute autre.
2. Vous êtes l'auteur de la contribution, vous avez le droit de la soumettre, et
   elle ne contient aucun code tiers que vous n'auriez pas le droit de soumettre.
3. Vous conservez le copyright sur votre propre contribution. La soumettre ne
   vous donne **aucun droit de propriété, aucune co-paternité, et aucun droit de
   redistribuer, publier, miroiter, relicencier ou forker Better Creative**.
   Vos autorisations restent exactement celles de n'importe qui d'autre au titre
   des §§2 et 3 — une PR fusionnée ne les élargit jamais.
4. Dans la limite permise par la loi, vous renoncez à vos droits moraux sur la
   contribution à l'égard de l'auteur ; à défaut, vous vous engagez à ne pas les
   invoquer d'une manière qui ferait obstacle à la licence ci-dessus. En
   contrepartie, votre contribution ne sera jamais attribuée à un tiers.
5. Ces conditions s'appliquent à l'identique, que vous soyez ou non membre de
   l'équipe ou de l'organisation qui héberge le dépôt. L'appartenance, le statut
   de mainteneur et l'accès en écriture ne donnent aucun droit sur
   Better Creative ni le pouvoir d'autoriser ce que la LICENSE réserve à
   THEFricadelle (§1).

## Ce que vous obtenez en retour

La Section 5.3 de la LICENSE accorde deux choses à tout
contributeur :

- **Le crédit** dans [CONTRIBUTORS.md](CONTRIBUTORS.md), sous le nom ou le
  pseudonyme que vous avez choisi et en nommant ce que vous avez réellement
  apporté, jamais retiré ultérieurement. Demandez via le tracker d'issues un
  autre nom, aucune adresse de contact, ou aucune mention du tout.
- **La permission modpack**, confirmée explicitement : vous pouvez diffuser
  Better Creative dans un modpack que vous publiez — canal officiel référencé,
  fichier officiel non modifié, mentions préservées. Avoir eu accès au code ne
  vous en prive jamais. Elle ne couvre que le fichier officiel, jamais un build
  de votre cru.

Ce crédit reconnaît votre travail. Il ne fait pas de vous un copropriétaire ni
un co-mainteneur, et ne donne aucun droit de redistribuer Better Creative.

Si vous n'acceptez pas ces conditions, ne soumettez pas de pull request —
ouvrez plutôt une issue décrivant le problème. C'est tout aussi utile.

## Contributions bienvenues

| Type | Bienvenue | Notes |
|------|-----------|-------|
| Corrections de bugs | ✅ Oui | Le meilleur type de PR. Incluez les étapes de reproduction. |
| Corrections de crash | ✅ Oui | Joignez le rapport de crash ou la stack trace. |
| Corrections de compatibilité | ✅ Oui | JEI, EMI, ModernFix, ou tout mod touchant aussi l'écran créatif. |
| Améliorations de performance | ✅ Oui | Expliquez la mesure, pas seulement la théorie. |
| Fautes de frappe, localisation | ✅ Oui | Petit et facile à fusionner. `en_us.json` **et** `fr_fr.json`. |
| Nouvelles traductions | ✅ Oui | Un nouveau `assets/bettercreative/lang/<locale>.json`, complet. |
| Corrections de documentation | ✅ Oui | README, guides, commentaires. |
| Nouvelles fonctionnalités | ⚠️ Demandez avant | Ouvrez une issue avant de coder — j'ai peut-être déjà une conception ou une raison de refuser. |
| Refactorisations / restylage | ⚠️ Demandez avant | Les gros diffs sans changement de comportement sont généralement refusés. |
| Changements de dépendances / build | ⚠️ Demandez avant | Impacte la distribution et le pipeline de release. |

## Comment soumettre une pull request

1. **Ouvrez d'abord une issue** pour tout ce qui dépasse une petite correction —
   cela évite du travail perdu des deux côtés.
2. **Créez une branche** depuis `dev` dans le dépôt officiel.
3. **Nommez la branche** `fix/description-courte` ou `feat/description-courte`.
4. **Écrivez le code** en suivant les conventions ci-dessous.
5. **Compilez et testez** :
   ```bash
   ./gradlew testAll    # unit tests, game tests on a dedicated server, live client run
   ```
   Tout doit passer. Une PR qui ne compile pas ne sera pas relue.
   `testAll` lance les tests unitaires, les game tests sur un serveur dédié, et
   un vrai client qui pilote l'écran créatif et écrit
   `run/clientsmoke/smoke-report.txt`. Ce dernier ouvre une fenêtre de jeu :
   n'y touchez pas avant sa fermeture, un clic dedans peut fausser un contrôle.
   JDK 21 requis ; le premier build télécharge et décompile Minecraft, comptez
   plusieurs minutes.
6. **Committez** avec un message conventionnel : `fix: keep hidden tabs out of the search aggregate`.
7. **Ouvrez la pull request vers `dev`**, en décrivant ce qu'elle
   corrige et comment vous l'avez vérifié.

## Conventions de code

- **Langue** : tout le code, les identifiants, les commentaires et les messages
  de log en **anglais**.
- **Nommage** : `PascalCase` classes, `camelCase` methods and fields, `UPPER_SNAKE_CASE` constants.
- **Les chaînes visibles par l'utilisateur** passent par les fichiers de langue,
  jamais en dur, et atterrissent dans **les deux** fichiers `en_us.json` et
  `fr_fr.json` dans le même changement.
- **Les méthodes d'injection mixin** doivent être préfixées
  `bettercreative$` pour ne pas entrer en collision avec les injecteurs
  d'un autre mod dans la même classe cible.
- **Préférez `@ModifyExpressionValue` à `@Redirect`** — bien meilleure
  compatibilité multi-mods.
- **La logique de tri reste dans `TabOrderResolver`**, fonction pure de (liste
  du jeu, config), pour pouvoir être raisonnée sans lancer le jeu.
- **Commentaires** : minimalistes et en anglais — expliquez le *pourquoi*, pas
  le *quoi*.
- **En-têtes SPDX** : conservez l'en-tête existant sur chaque fichier source.
  Les nouveaux fichiers doivent porter le même en-tête.
- **Aucun changement de version** : ne modifiez jamais `mod_version` (`gradle.properties`).
  Les releases sont gérées par l'auteur.
- **Aucune nouvelle dépendance** sans demander au préalable.
- **Périmètre** : un seul changement logique par pull request.

`RULES.md` liste l'intégralité des pièges propres au projet — surtout : ne
jamais muter un `CreativeModeTab` ni le registre (ce sont des singletons
partagés), ne jamais identifier un onglet par son index, ne jamais laisser le
resolver renvoyer une liste vide, et ne jamais supprimer les entrées de
configuration appartenant à un mod qui n'est pas installé actuellement.

## Signaler un bug

Incluez au minimum :

- La version de Better Creative, de Minecraft et de NeoForge.
- Si JEI, EMI, ModernFix ou un autre mod modifiant l'écran créatif est
  installé, et leurs versions.
- Les étapes de reproduction.
- L'extrait de log pertinent ou le rapport de crash (utilisez un service de
  paste pour les logs volumineux).
- Votre `config/better-creative/client.toml` si le problème
  concerne le tri, l'épinglage ou le masquage.

Si le rapport concerne l'option **portée de la recherche**, précisez si vous
avez testé dans un vrai modpack ou dans le client de développement — ModernFix
peut sauter le hook qu'une exécution vanilla exerce sans problème, et cette
différence a déjà produit deux faux « ça marche ».

## Contact

Pour toute demande de redistribution, d'autorisation modpack au-delà de ce que la LICENSE permet déjà, ou tout autre
point non couvert ici, passez par le tracker d'issues officiel :

  https://github.com/THEFricadelle/mc-mods-issues/issues/new?template=permission-request.yml

**Author: THEFricadelle**

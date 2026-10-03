# NOTICE — Better Creative

**Copyright (C) 2026 THEFricadelle. All rights reserved.**
SPDX-License-Identifier: `LicenseRef-Better-Creative-ARR`

Better Creative is **proprietary software**. Its source code is not published,
and the project is **not open-source**.

This file is a plain-language summary for convenience. The binding terms are in
[LICENSE](LICENSE); if the two ever disagree, the LICENSE wins.

## At a glance

| Action | Allowed? |
|--------|----------|
| Download the official build from CurseForge / Modrinth | ✅ Yes |
| Run it on your client, in singleplayer or on any server you join | ✅ Yes |
| Use it while playing on a monetized server (donations, ranks, shop) | ✅ Yes — as long as the mod itself isn't sold or paywalled |
| Keep the official file in a server's `mods/` folder, where it does nothing | ✅ Yes — §2.1, handy when a pack's server files mirror its client files |
| Report bugs, open issues | ✅ Yes |
| Be credited for a merged contribution, by name and by what you contributed | ✅ Yes — [CONTRIBUTORS.md](CONTRIBUTORS.md), §5.3(a) |
| Redistribute it, relicense it, or publish a fork **because you contributed** | ❌ No — §5.2, a merged PR enlarges nothing |
| Include it in a CurseForge / Modrinth modpack that **references** the official unmodified file | ✅ Yes, no need to ask |
| Send the official file to players joining **your own** server (launcher / host auto-sync) | ✅ Yes — see §2.2 |
| Share your presets file, your config file, or a pack policy file | ✅ Yes — your settings are yours, they are not part of the Mod |
| Mention it factually: "my pack includes Better Creative", tutorials, reviews | ✅ Yes |
| Bundle the .jar in an exported / offline modpack | ❌ Written permission required |
| Offer it as a "one-click install" product in a hosting panel catalogue | ❌ Written permission required |
| Re-upload or mirror it anywhere (mod-hosting sites, modpack platforms, forums, Discord, file lockers) | ❌ No |
| Modify it and distribute the result | ❌ No |
| Decompile or reverse-engineer it, beyond what the law always allows | ❌ No — §4 |
| Share the source code, if you were given access to contribute | ❌ No — §5.1 |
| Publish a build made from the source code | ❌ No |
| Reuse its code in another mod, plugin, or project | ❌ No |
| Sell it, rent it, or bundle it with a paid product | ❌ No |
| Claim you wrote it, or remove the copyright notices | ❌ No |
| Use the name, mod id, or logo for another project, or to imply endorsement | ❌ No |
| Use the code to train or fine-tune an AI / machine-learning model | ❌ No |
| Redistribute it, or authorize someone else to, because you are a team or organization member | ❌ No — §1, membership grants no permission |

## About revocation

The right to use the Mod is revocable — but **not arbitrarily**.
Revocation is **individual** (it takes effect only against a specific person or
entity, upon written notice), **prospective** (it never makes past compliant use
unlawful), and it does **not** silently kill a compliant modpack, nor other
users' ability to run an Official Build they lawfully obtained. It is a tool
against abuse, not a kill switch over the ecosystem. See
§2.3 of the LICENSE.

The modpack permission can be withdrawn from one specific pack maintainer, by
separate written notice to that maintainer. It never applies to pack versions
already published: players who installed them are not affected.

## Source code

The source code is not published. People given access to contribute may read
it and prepare contributions, nothing more, and that access can be withdrawn
(§5.1). A copy of the source made while an earlier version of the license made
it public stays governed by that earlier version, grants no right over later
releases, and is never an official channel.

## Ownership and maintenance

Better Creative is maintained by Team-Arcadia. Copyright in the
mod is held by **THEFricadelle** alone, the only party able to
grant, withhold, or withdraw any permission under the LICENSE.

Hosting under an organization or team account transfers nothing: being a member,
maintainer, or administrator gives no right to redistribute the
mod, publish a build of it, or authorize a third party to do
either. Team members who contribute do so as contributors, on the same terms as
anyone else (§5.2). See §1 of the [LICENSE](LICENSE).

## Third-party components

Better Creative builds against, but does not include or redistribute, the
following:

| Component | Role | Licensing |
|-----------|------|-----------|
| [NeoForge](https://neoforged.net/) | Mod loader / framework, config system (`ModConfigSpec`) | Provided by the end user's installation, under its own license |
| [SpongePowered Mixin](https://github.com/SpongePowered/Mixin) | Bytecode injection into the creative screen | Shipped with NeoForge, under its own license |
| [MixinExtras](https://github.com/LlamaLad7/MixinExtras) | `@ModifyExpressionValue` injector | Shipped with NeoForge, under its own license |
| [Gson](https://github.com/google/gson) | Preset and policy file parsing | Part of the Minecraft runtime, under its own license |
| Minecraft (Mojang) | Host game | Under Mojang's terms |

These are compile-time or runtime dependencies resolved on the user's side. No
third-party code is bundled into the Better Creative jar.

Minecraft is a trademark of Mojang Synergies AB. This mod is unofficial and is
not affiliated with or endorsed by Mojang Synergies AB or Microsoft.

## Your configuration is yours

The files under `config/better-creative/` — the client config, the presets file, and a
pack policy file a modpack ships there — are their owners' own data. Nothing in
the LICENSE restricts what you do with them: publish a preset, share it with
your pack's players, or paste it in an issue. They contain no part of the Mod's
code.

## Requesting permission

Anything marked ❌ above can still be granted case by case. Ask — the answer is
often yes for reasonable requests. Use the official issue tracker:

  https://github.com/THEFricadelle/mc-mods-issues/issues/new?template=permission-request.yml

Permission must be **written** to be valid. Silence is not consent: no reply, or
no objection to a use, never counts as permission. A permission granted in one
case applies to that case only.

**Author: THEFricadelle**

---

# NOTICE — Better Creative (Version Française)

**Copyright (C) 2026 THEFricadelle. Tous droits réservés.**
SPDX-License-Identifier: `LicenseRef-Better-Creative-ARR`

Better Creative est un **logiciel propriétaire**. Son code source n'est pas
publié, et le projet n'est **pas open-source**.

Ce fichier est un résumé en langage clair, fourni par commodité. Les conditions
contraignantes se trouvent dans [LICENSE](LICENSE) ; en cas de divergence, la
LICENSE prévaut.

## En un coup d'œil

| Action | Autorisé ? |
|--------|-----------|
| Télécharger le build officiel depuis CurseForge / Modrinth | ✅ Oui |
| L'exécuter sur votre client, en solo ou sur n'importe quel serveur rejoint | ✅ Oui |
| L'utiliser en jouant sur un serveur monétisé (dons, grades, boutique) | ✅ Oui — tant que le mod lui-même n'est ni vendu ni derrière un paywall |
| Laisser le fichier officiel dans le dossier `mods/` d'un serveur, où il ne fait rien | ✅ Oui — §2.1, pratique quand les fichiers serveur d'un pack reprennent ceux du client |
| Signaler des bugs, ouvrir des issues | ✅ Oui |
| Être crédité pour une contribution fusionnée, par nom et par ce que vous avez apporté | ✅ Oui — [CONTRIBUTORS.md](CONTRIBUTORS.md), §5.3(a) |
| Le redistribuer, le relicencier ou publier un fork **au motif que vous avez contribué** | ❌ Non — §5.2, une PR fusionnée n'élargit rien |
| L'inclure dans un modpack CurseForge / Modrinth qui **référence** le fichier officiel non modifié | ✅ Oui, sans demander |
| Transmettre le fichier officiel aux joueurs rejoignant **votre propre** serveur (auto-sync launcher / hébergeur) | ✅ Oui — voir §2.2 |
| Partager votre fichier de présets, de configuration ou un fichier de politique de pack | ✅ Oui — vos réglages vous appartiennent, ils ne font pas partie du mod |
| Le mentionner factuellement : « mon pack inclut Better Creative », tutoriels, tests | ✅ Oui |
| Empaqueter le .jar dans un modpack exporté / hors-ligne | ❌ Autorisation écrite requise |
| Le proposer en « installation en un clic » dans le catalogue d'un hébergeur | ❌ Autorisation écrite requise |
| Le ré-uploader ou le mirrorer ailleurs (sites d'hébergement de mods, plateformes de modpacks, forums, Discord, hébergeurs de fichiers) | ❌ Non |
| Le modifier et en distribuer le résultat | ❌ Non |
| Le décompiler ou le rétro-concevoir, au-delà de ce que la loi permet toujours | ❌ Non — §4 |
| Partager le code source, si l'on vous y a donné accès pour contribuer | ❌ Non — §5.1 |
| Publier un build issu du code source | ❌ Non |
| Réutiliser son code dans un autre mod, plugin ou projet | ❌ Non |
| Le vendre, le louer, ou le lier à un produit payant | ❌ Non |
| Prétendre l'avoir écrit, ou retirer les mentions de copyright | ❌ Non |
| Utiliser le nom, le mod id ou le logo pour un autre projet, ou pour suggérer une caution | ❌ Non |
| Utiliser le code pour entraîner ou affiner un modèle d'IA / d'apprentissage automatique | ❌ Non |
| Le redistribuer, ou autoriser quelqu'un à le faire, au motif que vous êtes membre de l'équipe ou de l'organisation | ❌ Non — §1, l'appartenance ne donne aucune autorisation |

## À propos de la révocation

Le droit d'utiliser le mod est révocable — mais **pas
arbitrairement**. La révocation est **individuelle** (elle ne prend effet qu'à
l'encontre d'une personne ou entité déterminée, sur notification écrite), **non
rétroactive** (elle ne rend jamais illicite un usage passé conforme), et elle ne
tue **pas** silencieusement un modpack conforme, ni la possibilité pour
les autres utilisateurs d'exécuter un build officiel obtenu licitement. C'est un
outil contre l'abus, pas un interrupteur sur l'écosystème. Voir
§2.3 de la LICENSE.

La permission modpack peut être retirée à un mainteneur de pack déterminé, par
une notification écrite distincte qui lui est adressée. Elle ne s'applique
jamais aux versions du pack déjà publiées : les joueurs qui les ont installées
ne sont pas concernés.

## Code source

Le code source n'est pas publié. Les personnes à qui l'on donne accès pour
contribuer peuvent le lire et préparer des contributions, rien de plus, et cet
accès peut être retiré (§5.1). Une copie du code faite à l'époque où une version
antérieure de la licence le rendait public reste régie par cette version
antérieure, ne donne aucun droit sur les versions suivantes, et n'est jamais un
canal officiel.

## Propriété et maintenance

Better Creative est maintenu par Team-Arcadia. Les droits d'auteur sur le
mod sont détenus par **THEFricadelle** seul, unique partie en mesure
d'accorder, de refuser ou de retirer une autorisation au titre de la LICENSE.

L'hébergement sous une organisation ou un compte d'équipe ne transfère rien :
être membre, mainteneur ou administrateur ne donne aucun droit de redistribuer
le mod, d'en publier un build, ni d'autoriser un tiers à le faire.
Les membres de l'équipe qui contribuent le font en tant que contributeurs, aux
mêmes conditions que n'importe qui d'autre (§5.2). Voir §1 de la
[LICENSE](LICENSE).

## Composants tiers

Better Creative compile contre les composants suivants, sans les inclure ni les
redistribuer :

| Composant | Rôle | Licence |
|-----------|------|---------|
| [NeoForge](https://neoforged.net/) | Mod loader / framework, système de config (`ModConfigSpec`) | Fourni par l'installation de l'utilisateur final, sous sa propre licence |
| [SpongePowered Mixin](https://github.com/SpongePowered/Mixin) | Injection bytecode dans l'écran créatif | Fourni avec NeoForge, sous sa propre licence |
| [MixinExtras](https://github.com/LlamaLad7/MixinExtras) | Injecteur `@ModifyExpressionValue` | Fourni avec NeoForge, sous sa propre licence |
| [Gson](https://github.com/google/gson) | Lecture des fichiers de présets et de politique | Partie du runtime Minecraft, sous sa propre licence |
| Minecraft (Mojang) | Jeu hôte | Sous les conditions de Mojang |

Ce sont des dépendances de compilation ou d'exécution résolues côté
utilisateur. Aucun code tiers n'est empaqueté dans le jar Better Creative.

Minecraft est une marque de Mojang Synergies AB. Ce mod est non officiel et
n'est ni affilié à Mojang Synergies AB ou Microsoft, ni approuvé par eux.

## Votre configuration vous appartient

Les fichiers de `config/better-creative/` — la configuration client, le fichier de
présets, et un fichier de politique qu'un modpack y livre — appartiennent à
leurs auteurs. Rien dans la LICENSE ne restreint ce que vous en faites :
publiez un préset, partagez-le avec les joueurs de votre pack, ou collez-le dans
une issue. Ils ne contiennent aucune partie du code du mod.

## Demander une autorisation

Tout ce qui est marqué ❌ ci-dessus peut malgré tout être accordé au cas par cas.
Demandez — la réponse est souvent oui pour les demandes raisonnables. Passez par
le tracker d'issues officiel :

  https://github.com/THEFricadelle/mc-mods-issues/issues/new?template=permission-request.yml

L'autorisation doit être **écrite** pour être valable. Le silence ne vaut pas
accord : l'absence de réponse, ou l'absence d'objection à un usage, ne constitue
jamais une autorisation. Une autorisation accordée dans un cas ne vaut que pour
ce cas.

**Author: THEFricadelle**

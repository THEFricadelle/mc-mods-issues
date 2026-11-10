# Migration guide

Upgrading an existing CustomPerm install. English first, French below.

**Français :** [Guide de migration](#guide-de-migration)

---

## 2.0.1 to 2.5.0: read permissions per area

`customperm.admin` no longer reads every area on its own. It opens `/customperm`, the Dashboard, Help,
`status`, `modcheck` and the admin alerts. Reading an area needs its new view node:

| Area | Read | Write (reads too) |
|---|---|---|
| Commands | `customperm.view.commands` | `customperm.manage.commands` |
| Aliases | `customperm.view.aliases` | `customperm.manage.aliases` |
| Rate limits | `customperm.view.ratelimits` | `customperm.manage.ratelimits` |
| Grades, Players page | `customperm.view.grades` | `customperm.manage.grades` |
| LuckPerms editor | `customperm.view.luckperms` | `customperm.manage.luckperms` |
| Logs | `customperm.view.logs` | `customperm.manage.logs` |
| Config (`contexts`, `names`) | `customperm.view.config` | `customperm.manage.config` |

**Who is affected.** Admins holding `customperm.*` (or `*`), the console and the host of a singleplayer or LAN
world keep everything. An admin holding `customperm.admin` alone, or with some `customperm.manage.<area>`
nodes, now reads only the areas of those manage nodes. A track moderator (`customperm.track.<track>`) still
promotes and demotes by command, but needs `customperm.view.grades` for the Players page and `track list`.

**To keep the former behaviour**, grant `customperm.view.*` next to `customperm.admin`:

```
# Internal backend (no LuckPerms), for a grade holding customperm.admin
customperm grade addperm <grade> customperm.view.*

# With LuckPerms
lp group <group> permission set customperm.view.* true
```

Your files keep their meaning. On the first start `settings.json` gains an `api` block (writes off, no mod
allowed), and every file is saved again in its normalised form, as each start already did. The server log
explains the view nodes at every start.

## 2.0.1 to 2.5.0: going back to 2.0.1

Keep your own copy of `config/arcadia/customperm/` before upgrading: the `backup/` folder holds the files as the
new release already loaded them. To go back, put the 2.0.1 jar back. It loads the files this release wrote
unchanged: the `customperm.view.*` nodes stay in the grades and are ignored, since 2.0.1 lets
`customperm.admin` read every area. 2.0.1 drops the `api` block when it saves `settings.json`, so after a later
upgrade the mods allowed to write through the API must be allowed again.

## 2.0.1 to 2.5.0: negative expiries

An `expires` value below zero in `grades.json` (a prefix or suffix entry, or a value in any `...Expiries`
map) used to read as permanent, like `0`. It now reads as already expired: the entry grants and shows nothing,
and the expiry sweep removes it within a second, as it does any entry whose time has passed. Each such value is
named in the server log at load (`grades.json: grades.vip.permissionExpiries.vip.node -9 is an impossible
expiry, read as already expired`). Commands and the interface never wrote a negative expiry, so only a hand
edit is concerned. To keep such an entry for good, set its `expires` to `0` (prefixes, suffixes) or delete its
line from the `...Expiries` map before upgrading, or add the entry again after.

## 1.0.x to 2.0.0

2.0.0 changes who may administer the mod, how permission nodes resolve, and how the in-game interface is
delivered. Nothing in your configuration files is rewritten, but two changes take effect the moment the
server starts, so read [What breaks on the first start](#what-breaks-on-the-first-start) before upgrading a
live server.

CustomPerm tells you about this itself: the configuration is stamped with a format version, and a
configuration written before 2.0.0 makes the server log the list of changes once, raise an admin alert, and
tell every operator who joins that the new permissions are needed.

### Before you upgrade

- Stop the server. Keep a copy of `config/arcadia/customperm/` and of your world.
- Note which players administer CustomPerm. You are about to grant them a permission.
- Check the exposed commands, aliases and grades you rely on: `/customperm status`, `/customperm scan`,
  `/customperm grade list`.
- CustomPerm 2.0.0 needs NeoForge 1.21.1 and, if you use it, LuckPerms 5.4.150 or later.

### What breaks on the first start

**1. Administering CustomPerm now needs a granted permission.**

Being an operator is no longer enough, permission level 4 included. `/customperm` and the admin interface
need op level 2 **and** `customperm.admin`; every change also needs the `customperm.manage.<area>` node of
its area. Straight after the upgrade nobody holds these nodes, so `/customperm` is hidden from everyone in
game. Grant them from the server console:

```
# Internal backend (no LuckPerms)
customperm grade create admins
customperm grade addperm admins customperm.*
customperm grade assign <name> admins

# With LuckPerms
lp user <name> permission set customperm.* true
```

The console always has access, and so does the host of a singleplayer or LAN world, which has no console.
Grant `customperm.admin` alone to a moderator who should read without changing anything, and add single
areas as needed: `customperm.manage.commands`, `.aliases`, `.ratelimits`, `.grades`, `.logs`, `.config`
(reload), `.luckperms`. After 2.0.1, `customperm.admin` alone reads no area: such a moderator also needs
`customperm.view.*` (see [2.0.1 to 2.5.0](#201-to-250-read-permissions-per-area)).

If you used the interface write nodes of the development builds, `customperm.gui.<area>.edit`, replace them
with the matching `customperm.manage.<area>`.

**2. Internal grades resolve the most specific node first.**

The old rule was "a DENY anywhere wins". The new one follows LuckPerms: the exact node beats `a.b.*`, which
beats `a.*`, which beats `*`, across every grade a player holds, and a DENY only wins against an ALLOW of the
same precision.

Grades that deny a wildcard while allowing something under it change meaning. For example, a grade with
`customperm.*` denied and `customperm.command.gamemode` allowed used to refuse `/gamemode`; it now grants it.
Review with:

```
customperm grade list
customperm test <player> <node>
```

`/customperm test` says whether the value is an explicit ALLOW, an explicit DENY, or not set.

**3. An explicit DENY now applies to operators.**

A DENY in a grade an operator holds, or in LuckPerms, now refuses the command, the alias or the admin access
to that operator too. Before, operators passed every CustomPerm check. This is what makes an accidental `/op`
harmless, and it can also refuse something to a staff member who used to pass regardless.

### What changes without breaking anything

- **The in-game interface needs CustomPerm on the admin's client.** It is drawn natively; the TesseraUI
  dependency and its panel are gone, and so is the `gui_sync` channel. Players without the mod connect
  normally and lose nothing but the interface.
- **Network protocol 2.** Clients that still run 1.0.x connect, but `/customperm gui` tells them the
  interface needs the current CustomPerm.
- **`/customperm gui` is a server command.** It opens the interface on the player who runs it.
- **`/customperm grade assign|unassign` take a player name**, not a selector. Offline players are resolved
  from the players who joined this server before, and a name several accounts used is refused rather than
  guessed.
- **New `settings.json` fields**, all with defaults that keep the previous behaviour: `gateAllCommands`,
  `defaultGrade`, `playerCommandLog`, `maskPlayerCommandArguments`, `maskedCommands`, `logRetentionDays`,
  `configVersion`.
- **The activity log** records admin changes from the first start, in `<world>/customperm/logs/`, kept 30
  days. Recording player commands is off until you turn it on.
- **New commands:** `command preserve`, `command gateall`, `alias movestep`, `alias setstep`,
  `grade adddeny`, `grade removedeny`, `grade setdefault`, `grade cleardefault`, `log admin|players`,
  `log record`, `log mask`.

### Already automatic

- The configuration directory moved from `config/customperm/` to `config/arcadia/customperm/` in an earlier
  version, and is still moved on start if the old one is found.
- Missing files, `{}` files, unknown fields and `null` collections are normalised into safe empty structures,
  so a 1.0.x configuration loads as it is.
- Rate limit rules without a `persistence` field default to `world_save`.

### After the upgrade

1. From the console, grant the admin nodes, then check in game that `/customperm status` answers.
2. Re-read the grades that deny a wildcard (point 2 above).
3. Optional, to protect against an accidental `/op`: a default grade plus gating every command, see
   "Restricting operators" in the README.
4. Optional: turn on the player command log if you want it, and tell your players.

### Going back to 1.0.x

Put the 1.0.5 jar back and restore the configuration you copied. A configuration written by 2.0.0 still loads
in 1.0.x: the fields it does not know are ignored. What you lose is what 2.0.0 stored in them, and grades
written with the new resolution in mind must be reviewed again, since 1.0.x applies "a DENY anywhere wins".

---

# Guide de migration

Mise à jour d'une installation CustomPerm existante.

## 2.0.1 vers 2.5.0 : permissions de lecture par domaine

`customperm.admin` ne lit plus tous les domaines à lui seul. Il ouvre `/customperm`, le Tableau de bord,
l'Aide, `status`, `modcheck` et les alertes admin. Lire un domaine demande son nouveau nœud de lecture :

| Domaine | Lecture | Écriture (lit aussi) |
|---|---|---|
| Commandes | `customperm.view.commands` | `customperm.manage.commands` |
| Alias | `customperm.view.aliases` | `customperm.manage.aliases` |
| Limites | `customperm.view.ratelimits` | `customperm.manage.ratelimits` |
| Grades, page Joueurs | `customperm.view.grades` | `customperm.manage.grades` |
| Éditeur LuckPerms | `customperm.view.luckperms` | `customperm.manage.luckperms` |
| Journaux | `customperm.view.logs` | `customperm.manage.logs` |
| Configuration (`contexts`, `names`) | `customperm.view.config` | `customperm.manage.config` |

**Qui est concerné.** Les admins qui détiennent `customperm.*` (ou `*`), la console et l'hôte d'un monde solo
ou LAN gardent tout. Un admin qui détient `customperm.admin` seul, ou avec quelques nœuds
`customperm.manage.<domaine>`, ne lit plus que les domaines de ces nœuds de gestion. Un modérateur de track
(`customperm.track.<track>`) promeut et rétrograde toujours par commande, mais a besoin de
`customperm.view.grades` pour la page Joueurs et `track list`.

**Pour garder l'ancien comportement**, accordez `customperm.view.*` à côté de `customperm.admin` :

```
# Backend interne (sans LuckPerms), pour un grade qui détient customperm.admin
customperm grade addperm <grade> customperm.view.*

# Avec LuckPerms
lp group <groupe> permission set customperm.view.* true
```

Vos fichiers gardent leur sens. Au premier démarrage, `settings.json` gagne un bloc `api` (écritures
désactivées, aucun mod autorisé), et chaque fichier est réenregistré sous sa forme normalisée, comme à chaque
démarrage déjà. Le log du serveur explique les nœuds de lecture à chaque démarrage.

## 2.0.1 vers 2.5.0 : revenir en 2.0.1

Gardez votre propre copie de `config/arcadia/customperm/` avant la mise à jour : le dossier `backup/` contient
les fichiers tels que la nouvelle version les a déjà chargés. Pour revenir en arrière, remettez le jar 2.0.1. Il
charge sans changement les fichiers écrits par cette version : les nœuds `customperm.view.*` restent dans les
grades et sont ignorés, puisque la 2.0.1 laisse `customperm.admin` lire tous les domaines. La 2.0.1 retire le
bloc `api` quand elle réenregistre `settings.json` : après une nouvelle mise à jour, les mods autorisés à écrire
par l'API doivent être autorisés de nouveau.

## 2.0.1 vers 2.5.0 : expirations négatives

Une valeur `expires` inférieure à zéro dans `grades.json` (une entrée de préfixe ou de suffixe, ou une valeur
d'une table `...Expiries`) valait jusqu'ici permanent, comme `0`. Elle vaut désormais déjà expirée : l'entrée
n'accorde et n'affiche rien, et le balayage des expirations la retire en moins d'une seconde, comme toute
entrée dont le temps est passé. Chaque valeur de ce type est nommée dans le log du serveur au chargement.
Les commandes et l'interface n'ont jamais écrit d'expiration négative : seule une modification à la main est
concernée. Pour garder une telle entrée pour de bon, mettez son `expires` à `0` (préfixes, suffixes) ou
supprimez sa ligne de la table `...Expiries` avant la mise à jour, ou ajoutez l'entrée de nouveau après.

## 1.0.x vers 2.0.0

La 2.0.0 change qui peut administrer le mod, la façon dont les nœuds de permission sont résolus, et la façon
dont l'interface en jeu est fournie. Aucun fichier de configuration n'est réécrit, mais deux changements
s'appliquent dès le démarrage du serveur : lisez [Ce qui casse au premier démarrage](#ce-qui-casse-au-premier-démarrage)
avant de mettre à jour un serveur en production.

CustomPerm vous prévient de lui-même : la configuration porte une version de format, et une configuration
écrite avant la 2.0.0 fait écrire la liste des changements dans le log au démarrage, lève une alerte admin, et
prévient chaque opérateur qui se connecte que les nouvelles permissions sont nécessaires.

### Avant la mise à jour

- Arrêtez le serveur. Gardez une copie de `config/arcadia/customperm/` et de votre monde.
- Notez quels joueurs administrent CustomPerm. Vous allez leur accorder une permission.
- Vérifiez les commandes exposées, alias et grades dont vous dépendez : `/customperm status`,
  `/customperm scan`, `/customperm grade list`.
- CustomPerm 2.0.0 demande NeoForge 1.21.1 et, si vous l'utilisez, LuckPerms 5.4.150 ou plus récent.

### Ce qui casse au premier démarrage

**1. Administrer CustomPerm demande désormais une permission accordée.**

Être opérateur ne suffit plus, niveau 4 compris. `/customperm` et l'interface d'administration demandent
op level 2 **et** `customperm.admin` ; chaque modification demande en plus le nœud `customperm.manage.<domaine>`
de son domaine. Juste après la mise à jour, personne ne détient ces nœuds : `/customperm` est donc masqué pour
tout le monde en jeu. Accordez-les depuis la console du serveur :

```
# Backend interne (sans LuckPerms)
customperm grade create admins
customperm grade addperm admins customperm.*
customperm grade assign <pseudo> admins

# Avec LuckPerms
lp user <pseudo> permission set customperm.* true
```

La console garde toujours l'accès, ainsi que l'hôte d'un monde solo ou LAN, qui n'a pas de console. Accordez
`customperm.admin` seul à un modérateur qui doit lire sans rien modifier, et ajoutez les domaines un par un :
`customperm.manage.commands`, `.aliases`, `.ratelimits`, `.grades`, `.logs`, `.config` (rechargement),
`.luckperms`. Après la 2.0.1, `customperm.admin` seul ne lit aucun domaine : ce modérateur a aussi besoin de
`customperm.view.*` (voir [2.0.1 vers 2.5.0](#201-vers-250--permissions-de-lecture-par-domaine)).

Si vous utilisiez les nœuds d'écriture des versions de développement, `customperm.gui.<domaine>.edit`,
remplacez-les par les `customperm.manage.<domaine>` correspondants.

**2. Les grades internes résolvent d'abord le nœud le plus spécifique.**

L'ancienne règle était « un DENY n'importe où l'emporte ». La nouvelle suit LuckPerms : le nœud exact passe
avant `a.b.*`, qui passe avant `a.*`, qui passe avant `*`, sur l'ensemble des grades d'un joueur, et un DENY ne
gagne que face à un ALLOW de même précision.

Les grades qui refusent un wildcard tout en autorisant quelque chose en dessous changent de sens. Par exemple,
un grade avec `customperm.*` refusé et `customperm.command.gamemode` autorisé refusait `/gamemode` ; il
l'accorde maintenant. À revoir avec :

```
customperm grade list
customperm test <pseudo> <nœud>
```

`/customperm test` indique si la valeur est un ALLOW explicite, un DENY explicite, ou non définie.

**3. Un DENY explicite s'applique désormais aux opérateurs.**

Un DENY dans un grade que détient un opérateur, ou dans LuckPerms, lui refuse désormais la commande, l'alias ou
l'accès d'administration. Avant, les opérateurs passaient tous les contrôles de CustomPerm. C'est ce qui rend un
`/op` accidentel inoffensif, et cela peut aussi refuser quelque chose à un membre du staff qui passait jusque-là.

### Ce qui change sans rien casser

- **L'interface en jeu demande CustomPerm sur le client de l'admin.** Elle est dessinée nativement ; la
  dépendance TesseraUI et son panneau ont disparu, ainsi que le canal `gui_sync`. Les joueurs sans le mod se
  connectent normalement et ne perdent que l'interface.
- **Protocole réseau 2.** Les clients encore en 1.0.x se connectent, mais `/customperm gui` leur explique que
  l'interface demande la version actuelle de CustomPerm.
- **`/customperm gui` est une commande serveur.** Elle ouvre l'interface sur le joueur qui la lance.
- **`/customperm grade assign|unassign` prennent un pseudo**, pas un sélecteur. Les joueurs hors ligne sont
  résolus parmi ceux déjà venus sur le serveur, et un pseudo utilisé par plusieurs comptes est refusé plutôt que
  deviné.
- **Nouveaux champs de `settings.json`**, tous avec des valeurs par défaut qui conservent le comportement
  précédent : `gateAllCommands`, `defaultGrade`, `playerCommandLog`, `maskPlayerCommandArguments`,
  `maskedCommands`, `logRetentionDays`, `configVersion`.
- **Le journal d'activité** enregistre les modifications d'administration dès le premier démarrage, dans
  `<monde>/customperm/logs/`, gardées 30 jours. L'enregistrement des commandes des joueurs reste désactivé tant
  que vous ne l'activez pas.
- **Nouvelles commandes :** `command preserve`, `command gateall`, `alias movestep`, `alias setstep`,
  `grade adddeny`, `grade removedeny`, `grade setdefault`, `grade cleardefault`, `log admin|players`,
  `log record`, `log mask`.

### Déjà automatique

- Le dossier de configuration est passé de `config/customperm/` à `config/arcadia/customperm/` dans une version
  antérieure, et il est toujours déplacé au démarrage si l'ancien est trouvé.
- Les fichiers manquants, les fichiers `{}`, les champs inconnus et les collections `null` sont normalisés en
  structures vides sûres : une configuration 1.0.x se charge telle quelle.
- Les règles de limite sans champ `persistence` prennent `world_save` par défaut.

### Après la mise à jour

1. Depuis la console, accordez les nœuds d'administration, puis vérifiez en jeu que `/customperm status` répond.
2. Relisez les grades qui refusent un wildcard (point 2 ci-dessus).
3. Facultatif, pour se protéger d'un `/op` accidentel : un grade par défaut et le contrôle de toutes les
   commandes, voir « Restreindre les opérateurs » dans le README.
4. Facultatif : activez le journal des commandes des joueurs si vous le souhaitez, et prévenez vos joueurs.

### Revenir en 1.0.x

Remettez le jar 1.0.5 et restaurez la configuration copiée. Une configuration écrite par la 2.0.0 se charge
encore en 1.0.x : les champs qu'elle ne connaît pas sont ignorés. Ce que vous perdez, c'est ce que la 2.0.0 y
stockait, et les grades écrits en pensant à la nouvelle résolution doivent être revus, puisque la 1.0.x
applique « un DENY n'importe où l'emporte ».

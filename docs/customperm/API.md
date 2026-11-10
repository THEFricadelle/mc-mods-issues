# CustomPerm API

Author: THEFricadelle

[English](#english) | [Français](#français)

---

## English

CustomPerm exposes a public API to other mods in the package `io.github.thefricadelle.customperm.api`, inside the mod jar.
It answers permission checks from the active backend, reads the internal grades, changes them for the mods an
admin allowed, and posts events when permissions change.

Only `io.github.thefricadelle.customperm.api` and `io.github.thefricadelle.customperm.api.event` are promised. Everything else in the
mod, `io.github.thefricadelle.customperm.api.impl` included, is internal and may change in any release.

For a plain "does this player have this node" check, NeoForge's `PermissionAPI` already works with CustomPerm
and needs no dependency at all. Use this API for what `PermissionAPI` does not cover: explicit values (`ALLOW`,
`DENY`, `UNSET`), offline checks, grades, writes and change events.

### Versions

The API has its own version, independent of the mod version:

- `CustomPermApi.API_MAJOR` grows on a breaking change; `CustomPermApi.API_MINOR` grows on an addition.
- Every public type carries `@since` with the API version that introduced it. The current API is 1.0.
- Interfaces you never implement are `@ApiStatus.NonExtendable`: new methods can be added without breaking you.

Check the version at run time when you rely on an addition:

```java
if (CustomPermApi.API_MAJOR == 1 && CustomPermApi.API_MINOR >= 0) { ... }
```

### Depending on CustomPerm

There is no separate API jar: compile against the released mod jar, `compileOnly`, and let the server install
CustomPerm. The license allows compiling against the unmodified official jar and calling
`io.github.thefricadelle.customperm.api`; it does not allow bundling, modifying or re-uploading the mod (see `LICENSE`).

With the Modrinth Maven repository:

```groovy
repositories {
    exclusiveContent {
        forRepository { maven { name = 'Modrinth'; url = 'https://api.modrinth.com/maven' } }
        filter { includeGroup 'maven.modrinth' }
    }
}

dependencies {
    // A CustomPerm release that contains io.github.thefricadelle.customperm.api (see CHANGELOG.md).
    compileOnly 'maven.modrinth:customperm:<version>'
}
```

With CurseForge, through the CurseMaven repository (project id and file id from the file's page):

```groovy
repositories {
    exclusiveContent {
        forRepository { maven { name = 'CurseMaven'; url = 'https://cursemaven.com' } }
        filter { includeGroup 'curse.maven' }
    }
}

dependencies {
    compileOnly('curse.maven:customperm-<projectId>:<fileId>') { transitive = false }
}
```

Before a release is on Modrinth or CurseForge, compile against a copy of the jar in your project instead:

```groovy
dependencies {
    compileOnly files('libs/customperm-<version>.jar')
}
```

To run CustomPerm in your own dev client, add the same coordinates to your runtime configuration (for example
`localRuntime` with ModDevGradle), never to a configuration that ends up in your jar.

Declare it optional in `neoforge.mods.toml`:

```toml
[[dependencies.yourmod]]
modId = "customperm"
type = "optional"
versionRange = "[<first version with the API>,)"
ordering = "NONE"
side = "BOTH"
```

### Soft dependency

Keep every reference to `io.github.thefricadelle.customperm.api` inside a class that is loaded only when CustomPerm is
present. The JVM resolves a class's references when the class is loaded, so a server without CustomPerm must
never load it.

```java
public final class Permissions {
    private static final boolean CUSTOMPERM = ModList.get().isLoaded("customperm");

    public static boolean canFly(ServerPlayer player) {
        return CUSTOMPERM ? CustomPermBridge.canFly(player) : player.hasPermissions(2);
    }
}

// CustomPermBridge.java, a class of its own, loaded only from the branch above.
import io.github.thefricadelle.customperm.api.CustomPermApi;

final class CustomPermBridge {
    static boolean canFly(ServerPlayer player) {
        return CustomPermApi.get().hasPermission(player, "yourmod.fly");
    }
}
```

### Threading and lifecycle

- `CustomPermApi.get()` never returns null once CustomPerm is loaded. `isServerReady()` says whether a server
  runs.
- Synchronous reads run on the server thread and throw `IllegalStateException` on any other thread.
- `checkOffline` and every write return a `CompletableFuture` that always completes on the server thread, so
  `thenAccept` can touch the world safely. They may be called from any thread. Called on the server thread, a
  write's future is already complete when returned, except in a cluster: there it completes once the cluster
  database answered, with the final outcome (`OK`, or `REFUSED_CLUSTER` with the change undone), never before.
- Never `join()` an incomplete future on the server thread: it waits for the very thread that would complete it.
- Without a running server, a write completes at once with `NOT_READY`, and `checkOffline` with `UNSET`.

### Reads

Every read answers from the active backend, exactly as a CustomPerm command check does: LuckPerms when it is in
use, otherwise the internal grades (`grades.json`).

| Method | Answer |
| --- | --- |
| `backend()` | `INTERNAL`, `LUCKPERMS`, `INTERNAL_FALLBACK` (LuckPerms unavailable, grades stand in) or `DENY` (LuckPerms unavailable, nothing granted) |
| `check(player, node)` | `ALLOW`, `DENY` or `UNSET`, without the operator fallback |
| `hasPermission(player, node)` | the effective value: `check`, and on `UNSET` whether the player is operator (level 2) |
| `checkOffline(uuid, node, context)` | a future of `check` for a player who may be offline, in a context (`ApiContext.GLOBAL` for the entries that apply everywhere); under LuckPerms the user is loaded from its storage |
| `grades(uuid)` | the grades held directly, everywhere and in contexts, with their expiry; under LuckPerms the parent groups of a loaded user, flagged `fromLuckPerms` |
| `grade(name)`, `gradeNames()`, `defaultGrade()` | the internal grades; empty under LuckPerms, whose groups belong to the LuckPerms API |
| `meta(player, key)`, `prefix(player)`, `suffix(player)` | as the active backend resolves them, LuckPerms meta stacking included |
| `contexts(player)` | world, game mode and the server's static contexts, as CustomPerm sees the player |

### Writes

Writes go through an `ApiWriter` obtained for your mod id:

```java
import java.time.Duration;
import io.github.thefricadelle.customperm.api.*; // ApiWriter, ApiContext, ApiResult, ApiHolder, ApiTristate...

ApiWriter writer = CustomPermApi.get().writer("yourmod"); // IllegalArgumentException for an id no loaded mod has
writer.assignGrade(player.getUUID(), "vip", Duration.ofDays(30), ApiContext.GLOBAL)
      .thenAccept(result -> {
          if (!result.success()) LOGGER.warn("CustomPerm refused: {} ({})", result.message(), result.status());
      });
```

Version 1.0 covers:

| Method | Does what this command does |
| --- | --- |
| `assignGrade`, `unassignGrade` | `grade assign` (optional duration and context) / `grade unassign` (optional context) |
| `setNode`, `unsetNode` on `ApiHolder.player(uuid)` or `ApiHolder.grade(name)` | `user addperm` / `grade addperm`, `adddeny`, and their removals |
| `setMeta`, `unsetMeta` | `grade meta set` / `user meta set` and `unset` |
| `setPrefix`, `removePrefix`, `setSuffix`, `removeSuffix` | the prefix and suffix commands, on a grade or a player |
| `promote`, `demote` | `promote` / `demote` on a track |
| `createGrade`, `deleteGrade` | `grade create` / `grade delete` |

A `null` or zero `Duration` is permanent; a `null` context is `ApiContext.GLOBAL`. A write runs the same code as
the command: the same validation, the lockout guard where the command has it, the save, the cluster publication,
and the refresh of players' command trees and names. It is recorded in the activity log with the actor
`mod:<modid>` and the source `api`, so admins see which mods change permissions; the first change each mod
makes after a start is also logged at INFO.

`writer.onBehalfOf(adminPlayer)` makes the change for that admin: it needs the same node as the command
(`customperm.manage.grades`, or the track node for a promotion), the lockout guard refuses a change that would
take away the admin's own access, and the log names the admin with the mod.

#### Enabling writes

Writes are off until the server admin allows them, in `settings.json`:

```json
"api": { "writes": true, "writers": ["yourmod"] }
```

- `writes` (default `false`): whether any mod may write.
- `writers` (default empty): the mod ids allowed while `writes` is on. An empty list allows none: explicit is
  safer.
- The same from the game: `/customperm api`, `/customperm api writes <on|off>`,
  `/customperm api writers <add|remove> <modid>` (needs `customperm.manage.config`).
- The same in the admin interface (`/customperm gui api`, the Mod API page): the switch, the list, the mods
  refused and the mods that wrote this session; **Allow** on a refused mod lists it in one click.

A write refused by these settings completes with `REFUSED_DISABLED` and raises an admin alert naming your mod
and the setting to change. The alert is raised once per mod, not per attempt, and is resolved when the admin
allows the mod. Tell your users which setting to change.

These settings are not a security boundary: any mod in the same JVM can reach CustomPerm's internal classes.
They give attribution and a switch the admin controls.

#### Results

`ApiResult(status, message, warnings)`: `success()` is true for `OK` and `NO_CHANGE`, `changed()` only for `OK`.
`message` is the sentence an admin would read for the same change in chat.

| Status | Meaning |
| --- | --- |
| `OK` | the change applied |
| `NO_CHANGE` | the entry was already as asked |
| `INVALID` | an invalid node, name, key, value, priority, duration or context |
| `NOT_FOUND` | the grade or track named does not exist |
| `REFUSED_LOCKOUT` | the lockout guard undid the change (with `onBehalfOf`) |
| `REFUSED_CLUSTER` | the cluster refused the change, which was undone; retrying later may apply |
| `REFUSED_LUCKPERMS` | LuckPerms manages permissions: use the LuckPerms API there |
| `REFUSED_DISABLED` | `api.writes` is off or your mod is not in `api.writers` |
| `REFUSED_PERMISSION` | the admin given to `onBehalfOf` may not make this change |
| `BACKEND_UNAVAILABLE` | LuckPerms is unavailable and the server denies everything until it is back |
| `NOT_READY` | no server runs |

#### Under LuckPerms

With LuckPerms as the active backend, every write completes with `REFUSED_LUCKPERMS`, whatever the settings say:
the internal grades do not decide anything there. Write through the LuckPerms API instead. When LuckPerms is
unavailable, CustomPerm follows `luckPermsFallbackMode`: writes apply on the internal fallback, and complete with
`BACKEND_UNAVAILABLE` on the deny one.

### Events

Events are plain NeoForge events on `NeoForge.EVENT_BUS`, posted on the server thread once a change is final:
applied, saved, published to the cluster and pushed to the players. A change refused or undone (lockout guard,
cluster refusal) posts nothing, and a no-op posts nothing. Events cannot be cancelled. A listener that throws is
logged and never undoes the change. Listen to the concrete classes below.

```java
NeoForge.EVENT_BUS.addListener(PermissionsChangedEvent.class, event -> {
    if (event.all()) cache.clear(); else event.players().forEach(cache::remove);
});
```

A listener added with `addListener` stays for the life of the game. To be able to remove it, register an object
whose methods carry `@SubscribeEvent`, then `NeoForge.EVENT_BUS.register(listener)` and later
`NeoForge.EVENT_BUS.unregister(listener)`. The event classes are in `io.github.thefricadelle.customperm.api.event`.

| Event | Posted when |
| --- | --- |
| `GradeAssignedEvent(player, grade, expiresAt, context)` | a player was given a grade, or its expiry changed |
| `GradeUnassignedEvent(player, grade, context)` | a player lost a grade: unassigned, moved on a track, or expired |
| `NodeChangedEvent(holder, node, allow, removed, expiresAt, context)` | an ALLOW or DENY entry of a player or grade was added, changed or removed |
| `GradeCreatedEvent(grade)`, `GradeDeletedEvent(grade)` | a grade was created or deleted |
| `ConfigReloadedEvent` | `/customperm reload` read the files again |
| `BackendChangedEvent(previous, current)` | LuckPerms became unavailable, or a reload found it again |
| `PermissionsChangedEvent(players, all)` | what some players may do changed: once per change, after its detailed events, and alone where no detail exists |

Every event carries `source()`: `COMMAND`, `INTERFACE`, `API`, `EXPIRY`, `CLUSTER`, `LUCKPERMS`, `RELOAD`, or
`OTHER` for a change that finished in the background.

What posts only `PermissionsChangedEvent`: meta, prefixes, inheritance, weights, the default grade and static
contexts; a change made in LuckPerms (`/lp`, its web editor, its sync), for the player LuckPerms recalculated;
and a change received from another server of the cluster, for the players or grades it replaced. For detail
under LuckPerms, subscribe to the LuckPerms event bus. Events are local to a server: a change made on cluster
member A posts its detailed events on A and the coarse one on B.

---

## Français

CustomPerm expose une API publique aux autres mods dans le package `io.github.thefricadelle.customperm.api`, dans le jar du
mod. Elle répond aux tests de permission depuis le backend actif, lit les grades internes, les modifie pour les
mods qu'un admin a autorisés, et publie des événements quand les permissions changent.

Seuls `io.github.thefricadelle.customperm.api` et `io.github.thefricadelle.customperm.api.event` sont garantis. Tout le reste du mod,
`io.github.thefricadelle.customperm.api.impl` compris, est interne et peut changer à chaque version.

Pour un simple « ce joueur a-t-il ce node », le `PermissionAPI` de NeoForge fonctionne déjà avec CustomPerm sans
aucune dépendance. Cette API sert à ce qu'il ne couvre pas : valeurs explicites (`ALLOW`, `DENY`, `UNSET`), tests
hors ligne, grades, écritures et événements.

### Versions

L'API a sa propre version, indépendante de celle du mod :

- `CustomPermApi.API_MAJOR` augmente à chaque changement incompatible ; `CustomPermApi.API_MINOR` à chaque ajout.
- Chaque type public porte `@since` avec la version de l'API qui l'a introduit. L'API actuelle est la 1.0.
- Les interfaces que vous n'implémentez jamais sont `@ApiStatus.NonExtendable` : de nouvelles méthodes peuvent y
  être ajoutées sans vous casser.

### Dépendre de CustomPerm

Il n'y a pas de jar d'API séparé : compilez contre le jar publié du mod, en `compileOnly`, et laissez le serveur
installer CustomPerm. La licence autorise la compilation contre le jar officiel non modifié et l'appel de
`io.github.thefricadelle.customperm.api` ; elle n'autorise ni à l'embarquer, ni à le modifier, ni à le republier (voir
`LICENSE`).

Les coordonnées Maven (Modrinth `maven.modrinth:customperm:<version>`, ou CurseMaven
`curse.maven:customperm-<projectId>:<fileId>`), la déclaration optionnelle dans `neoforge.mods.toml` et le
modèle de dépendance optionnelle sont ceux de la partie anglaise ci-dessus. Avant qu'une version soit publiée,
compilez contre une copie du jar dans votre projet : `compileOnly files('libs/customperm-<version>.jar')`. Gardez
toute référence à `io.github.thefricadelle.customperm.api` dans une classe à part (son propre fichier, qui importe
le package api), chargée seulement quand `ModList.get().isLoaded("customperm")` est vrai.

### Threads et cycle de vie

- `CustomPermApi.get()` ne renvoie jamais null une fois CustomPerm chargé ; `isServerReady()` dit si un serveur
  tourne.
- Les lectures synchrones se font sur le thread serveur et lèvent `IllegalStateException` ailleurs.
- `checkOffline` et chaque écriture renvoient un `CompletableFuture` toujours complété sur le thread serveur. Ils
  peuvent être appelés depuis n'importe quel thread ; appelée sur le thread serveur, une écriture est déjà
  terminée, sauf en cluster : elle se termine alors quand la base du cluster a répondu, avec le résultat final
  (`OK`, ou `REFUSED_CLUSTER` avec le changement annulé), jamais avant.
- Ne faites jamais `join()` sur un future non terminé depuis le thread serveur.
- Sans serveur, une écriture se termine aussitôt avec `NOT_READY`, et `checkOffline` avec `UNSET`.

### Lectures

Chaque lecture répond depuis le backend actif, comme un test de commande CustomPerm : LuckPerms quand il est
utilisé, sinon les grades internes. `check` donne la valeur explicite, `hasPermission` la valeur effective (avec
le repli opérateur niveau 2), `checkOffline` un joueur éventuellement hors ligne dans un contexte, `grades` les
grades détenus (sous LuckPerms, les groupes d'un utilisateur chargé, marqués `fromLuckPerms`), `grade`,
`gradeNames` et `defaultGrade` les grades internes (vides sous LuckPerms), `meta`, `prefix`, `suffix` et
`contexts` ce que le backend actif résout.

### Écritures

Les écritures passent par un `ApiWriter` obtenu pour l'identifiant de votre mod (`writer("votremod")`). La
version 1.0 couvre : assigner et retirer un grade (durée et contexte optionnels), ajouter et retirer un node
ALLOW ou DENY sur un joueur ou un grade, poser et retirer une meta, monter et descendre sur une track, créer et
supprimer un grade, poser et retirer un préfixe ou un suffixe sur un grade ou un joueur. Une écriture exécute le
même code que la commande : mêmes validations, garde anti-verrouillage là où la commande l'a, sauvegarde,
publication cluster, renvoi de l'arbre de commandes et des noms. Elle est inscrite au journal d'activité avec
l'acteur `mod:<modid>` et la source `api` ; le premier changement de chaque mod après un démarrage est aussi
journalisé en INFO. `onBehalfOf(admin)` fait le changement pour cet admin, avec ses droits et la garde
anti-verrouillage.

#### Activer les écritures

Les écritures sont désactivées tant que l'admin du serveur ne les autorise pas, dans `settings.json` :

```json
"api": { "writes": true, "writers": ["votremod"] }
```

- `writes` (`false` par défaut) : si un mod peut écrire.
- `writers` (vide par défaut) : les mods autorisés quand `writes` est actif. Une liste vide n'en autorise aucun.
- En jeu : `/customperm api`, `/customperm api writes <on|off>`, `/customperm api writers <add|remove> <modid>`
  (exige `customperm.manage.config`).
- Dans l'interface d'administration (`/customperm gui api`, page Mod API) : l'interrupteur, la liste, les mods
  refusés et ceux qui ont écrit pendant la session ; **Allow** sur un mod refusé l'ajoute en un clic.

Une écriture refusée par ces réglages se termine avec `REFUSED_DISABLED` et lève une alerte admin qui nomme le
mod et le réglage à changer, une fois par mod et non à chaque tentative ; elle est résolue quand l'admin autorise
le mod. Ces réglages ne sont pas une frontière de sécurité : ils donnent l'attribution et un interrupteur à
l'admin.

Les statuts de `ApiResult` sont décrits dans le tableau anglais ci-dessus. Sous LuckPerms (backend actif), toute
écriture se termine avec `REFUSED_LUCKPERMS` : utilisez l'API de LuckPerms. LuckPerms indisponible : les
écritures s'appliquent en repli interne et se terminent avec `BACKEND_UNAVAILABLE` en repli deny.

### Événements

Des événements NeoForge sur `NeoForge.EVENT_BUS`, publiés sur le thread serveur une fois le changement
définitif (appliqué, sauvegardé, publié au cluster, envoyé aux joueurs). Un changement refusé ou annulé (garde
anti-verrouillage, refus du cluster) ou sans effet ne publie rien. Non annulables ; un écouteur qui lève une
exception est journalisé et n'annule jamais le changement. Écoutez les classes concrètes : `GradeAssignedEvent`,
`GradeUnassignedEvent`, `NodeChangedEvent`, `GradeCreatedEvent`, `GradeDeletedEvent`, `ConfigReloadedEvent`,
`BackendChangedEvent` et `PermissionsChangedEvent`, le signal global « vos caches sont périmés », publié une fois
par changement après ses événements détaillés, et seul quand aucun détail n'existe (meta, préfixes, héritage,
changements faits dans LuckPerms ou reçus d'un autre serveur du cluster). Chaque événement porte sa `source()`.
Les classes sont dans `io.github.thefricadelle.customperm.api.event`. Un écouteur ajouté par `addListener` reste
actif tant que le jeu tourne ; pour pouvoir le retirer, enregistrez un objet dont les méthodes portent
`@SubscribeEvent` avec `NeoForge.EVENT_BUS.register(...)`, puis `unregister(...)`.

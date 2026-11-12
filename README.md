# Ce mot de passe s'autodétruira dans 1 heure

Un talk pratique sur OpenBao et les credentials dynamiques PostgreSQL : comment remplacer les secrets statiques par des accès temporaires, générés à la demande et automatiquement révoqués.

## 📋 Résumé

Qui n'a jamais retrouvé un compte PostgreSQL d'un collègue parti depuis plusieurs mois, ou un mot de passe qui n'a pas changé depuis la mise en service de la base ? Les comptes nominatifs, c'est propre sur le papier, mais en pratique il faut les créer, les maintenir et penser à les supprimer base par base... et on finit toujours par en oublier.

Et si on arrêtait tout simplement de gérer les comptes et les mots de passe PostgreSQL à la main ? Avec OpenBao, l'utilisateur s'authentifie avec son compte d'entreprise et demande un accès à une base. OpenBao crée alors à la volée un utilisateur PostgreSQL avec les bons droits et lui transmet les identifiants. Une heure plus tard, ce compte disparaît tout seul. Même le compte qu'OpenBao utilise pour faire tout ça a un mot de passe que personne ne connaît.

Je vous montrerai comment mettre en place cette solution avec PostgreSQL et OpenBao, mais surtout comment la rendre utilisable au quotidien : pour un dev, se connecter à une base depuis DBeaver ou IntelliJ, c'est juste un double-clic. La différence, c'est qu'il n'a plus de mot de passe à retenir, et les équipes infra, plus de comptes à nettoyer.

## 🎤 Informations pratiques

| Critère | Valeur |
|---|---|
| **Niveau** | Intermédiaire |
| **Durée** | 45 minutes |
| **Format** | Talk + démos + retour d'expérience |
| **Langue** | Français |

## 🎯 Objectifs

- Comprendre le fonctionnement du Database Secrets Engine d'OpenBao pour PostgreSQL.
- Configurer les rôles, les permissions et les policies pour générer des comptes temporaires.
- Comprendre les TTL, les leases, le renouvellement et la révocation des accès.
- Découvrir la rotation des identifiants du compte PostgreSQL utilisé par OpenBao.
- Faciliter les connexions depuis DBeaver ou IntelliJ sans gérer manuellement les mots de passe PostgreSQL.

## ✅ Pré-requis

- Connaître les bases de PostgreSQL (utilisateurs, rôles et permissions).
- Avoir quelques notions de gestion des secrets et d'authentification.
- Connaître OpenBao est un plus, mais ce n'est pas indispensable.

Aucune machine n'est nécessaire : il s'agit d'une présentation avec des démonstrations, pas d'un atelier. Le talk suppose une instance OpenBao déjà opérationnelle, sans imposer Kubernetes ou un autre environnement de déploiement.

## 🗂️ Structure du repo

Structure **prévisionnelle**, qui évoluera au fil de la préparation :

```text
.
├── slides/
│   ├── slides.md             # Présentation Slidev
│   └── public/               # Images et schémas
├── demo/
│   ├── openbao/
│   │   ├── policies/         # Policies d'accès
│   │   └── scripts/          # Configuration et démonstrations
│   ├── postgresql/
│   │   └── init/             # Initialisation et rôles SQL
│   └── clients/
│       ├── linux-macos/      # Script d'authentification shell
│       └── windows/          # Script d'authentification batch
├── docs/                    # Architecture et runbook
├── LICENSE                  # CC BY 4.0 (contenus pédagogiques)
├── LICENSE-CODE             # Apache-2.0 (code et configurations)
└── README.md
```

Les fichiers et répertoires ci-dessus ne sont pas tous disponibles pour le moment.

## 🖥️ Slides en ligne

- **Slides** : en cours de préparation avec [Slidev](https://sli.dev/).
- **Sources** : https://github.com/mbatard/talk-openbao-dynamic-credentials-postgresql

Les liens de consultation et les instructions de lancement seront ajoutés lors de la publication des slides.

## 🔧 Stack utilisée

| Outil | Usage |
|---|---|
| [OpenBao](https://openbao.org/) | Authentification, policies, Database Secrets Engine et leases |
| [PostgreSQL](https://www.postgresql.org/) | Base de données et comptes temporaires |
| [DBeaver](https://dbeaver.io/) | Client SQL pour les démonstrations |
| [IntelliJ IDEA](https://www.jetbrains.com/idea/) | IDE et client SQL |
| CLI `bao` | Authentification et gestion des accès OpenBao |
| Shell / batch | Génération manuelle du fichier token sur les postes |
| [Slidev](https://sli.dev/) | Support de présentation |

Les versions exactes seront précisées une fois l'environnement de démonstration finalisé.

## 🎬 La démo

### Architecture et parcours utilisateur

```text
Poste développeur (Linux / macOS / Windows)
  |
  | 1. Lancement manuel du script shell / batch
  | 2. Authentification d'entreprise via le CLI OpenBao
  v
Fichier token OpenBao local
  |
  | 3. Token configuré dans DBeaver / IntelliJ
  v
DBeaver / IntelliJ
  |
  | 4. Demande de credentials PostgreSQL avec le token
  v
OpenBao (Database Secrets Engine)
  |
  | 5. Création d'un compte temporaire avec les bons droits
  v
PostgreSQL
  ^
  | 6. Identifiants transmis au client pour la connexion
  |
DBeaver / IntelliJ

À expiration du lease : OpenBao déclenche la révocation
 du compte PostgreSQL selon la configuration définie.
```

Le script d'authentification est lancé **manuellement** pour obtenir le token OpenBao. DBeaver et IntelliJ utilisent ensuite le fichier token pour demander eux-mêmes les credentials PostgreSQL : **ils ne lancent pas le script**. Une tâche planifiée pour renouveler le token est envisagée, mais n'est pas encore mise en place.

### Démonstrations prévues

1. Se connecter à PostgreSQL depuis DBeaver ou IntelliJ en un double-clic.
2. Configurer le Database Secrets Engine et les rôles PostgreSQL.
3. Générer des credentials dynamiques et vérifier les permissions du compte créé.
4. Utiliser un TTL court pour observer l'expiration d'un lease et la révocation du compte.
5. Montrer la rotation du mot de passe du compte PostgreSQL utilisé par OpenBao.

Un environnement de démonstration isolé sera utilisé, sans dépendance à la production. Les scripts et le runbook reproductible seront publiés progressivement.

## 🛡️ Points de vigilance et retour d'expérience

Le talk abordera aussi les cas qui méritent une attention particulière en production :

- La différence entre la durée de vie du token OpenBao et celle des credentials PostgreSQL.
- Les connexions PostgreSQL encore actives lors de l'expiration d'un lease.
- Les échecs de révocation, les dépendances SQL et les éventuels comptes orphelins.
- Le comportement en cas d'indisponibilité d'OpenBao.
- Le principe du moindre privilège et la protection du fichier token local.
- Le renouvellement de l'authentification sur les postes de travail.

L'expiration d'un lease déclenche une révocation, mais la suppression effective du compte dépend de la configuration et du bon déroulement des opérations PostgreSQL.

## 🔗 Pour aller plus loin

- [Documentation OpenBao](https://openbao.org/docs/)
- [Database Secrets Engine](https://openbao.org/docs/secrets/databases/)
- [Plugin PostgreSQL d'OpenBao](https://openbao.org/docs/secrets/databases/postgresql/)
- [Documentation PostgreSQL](https://www.postgresql.org/docs/)
- [Slidev](https://sli.dev/)
- [Sources du talk (GitHub)](https://github.com/mbatard/talk-openbao-dynamic-credentials-postgresql)

## 📄 Licence

Ce dépôt utilise deux licences selon la nature des contenus :

| Contenu | Licence |
|---|---|
| Slides, schémas, documentation et README | [Creative Commons Attribution 4.0 International (CC BY 4.0)](https://creativecommons.org/licenses/by/4.0/) |
| Scripts, configurations OpenBao/PostgreSQL et exemples de code | [Apache License 2.0](https://www.apache.org/licenses/LICENSE-2.0) |

Les textes complets des licences seront disponibles dans `LICENSE` (CC BY 4.0) et `LICENSE-CODE` (Apache-2.0). Les éventuels éléments tiers conservent leurs propres licences.

## 👤 Auteur

**Mikael Batard** — SRE @ CBA Informatique Libérale

💼 [linkedin.com/in/mbatard](https://www.linkedin.com/in/mbatard/)

🦋 [@mbatard.bsky.social](https://bsky.app/profile/mbatard.bsky.social)

🐙 [github.com/mbatard](https://github.com/mbatard)

🌐 [calmops.fr](https://calmops.fr)

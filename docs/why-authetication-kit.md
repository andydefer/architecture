# Laravel Authentication Kit

## Une fondation d'authentification modulaire pour Laravel

Laravel Authentication Kit est un package d'authentification conçu autour d'un principe simple :

> **séparer complètement le moteur d'authentification de la logique métier.**

L'objectif n'est pas de remplacer Laravel Fortify ou Jetstream, mais de proposer une alternative beaucoup plus modulaire, extensible et adaptée aux architectures modernes.

Au lieu d'imposer un workflow unique, le package fournit uniquement les composants essentiels permettant de construire n'importe quel système d'authentification.

---

# Pourquoi ce package ?

Dans beaucoup de projets, l'authentification devient rapidement un mélange de logique métier, de contrôleurs, de middleware, de génération de tokens et de validation.

Au fil du temps, cela crée plusieurs problèmes :

* duplication de logique ;
* difficulté à tester ;
* difficulté à faire évoluer le système ;
* dépendance forte entre l'authentification et l'application.

Laravel Authentication Kit adopte une approche différente.

L'authentification devient un ensemble de composants indépendants qui collaborent entre eux.

Chaque partie possède une responsabilité unique.

---

# Une architecture basée sur des responsabilités bien définies

Le package repose sur plusieurs briques indépendantes.

## Les Actions

Toute opération est encapsulée dans une Action.

Par exemple :

* inscription
* connexion
* réinitialisation du mot de passe
* vérification d'email

Chaque Action possède un cycle de vie clair :

```
before()

↓

handle()

↓

after()
```

Cette architecture permet :

* d'ajouter facilement des comportements avant ou après une opération ;
* d'éviter les contrôleurs surchargés ;
* de faciliter les tests unitaires ;
* de garder un code lisible.

---

# Les modèles restent maîtres de leur logique

Le package ne décide jamais comment un utilisateur est créé.

Chaque modèle fournit lui-même son service d'authentification.

Exemple :

```php
class User extends Model implements MailAuthenticatable
{
    public static function getMailAuthService(): UserAuthenticationService
    {
        return new UserAuthenticationService();
    }
}
```

Cela signifie qu'un projet peut avoir :

* User
* Admin
* Customer
* Employee
* Partner

avec des règles d'authentification totalement différentes.

Le package ne fait aucune hypothèse.

---

# Aucune dépendance à un modèle User

L'une des principales limitations de nombreuses solutions d'authentification est qu'elles supposent l'existence d'un modèle `User`.

Laravel Authentication Kit fonctionne avec **n'importe quel modèle** implémentant simplement un contrat.

```php
interface MailAuthenticatable
{
    public static function getMailAuthService();
}
```

Cette approche permet de construire des applications multi-utilisateurs sans duplication de code.

---

# Génération des tokens indépendante

Le package délègue complètement la génération des tokens à **Nemesis**.

Chaque authentification produit un token contenant :

* la source du token ;
* les informations du navigateur ;
* le système d'exploitation ;
* le type d'appareil ;
* l'adresse IP ;
* le User-Agent.

Exemple :

```php
new NemesisTokenRecord(
    name: "api",
    source: TokenSource::LOGIN->value,
    metadata: ...
);
```

Le moteur d'authentification reste donc indépendant du système de gestion des tokens.

---

# Traçabilité complète des événements

Chaque opération importante est automatiquement journalisée.

Par exemple :

* connexion réussie
* connexion échouée
* inscription
* erreur
* réinitialisation du mot de passe

Le système enregistre automatiquement :

* l'adresse IP ;
* le navigateur ;
* la plateforme ;
* le type d'appareil ;
* le modèle concerné ;
* le type d'événement.

Cela facilite :

* les audits ;
* le débogage ;
* la détection des comportements suspects ;
* l'analyse de sécurité.

---

# Typage fort de tous les événements

Les événements utilisent des Enum.

```php
EventType::USER_LOGIN_SUCCESS

EventType::USER_LOGIN_FAILED

EventType::USER_REGISTRATION_SUCCESS
```

De même, chaque token possède son origine.

```php
TokenSource::LOGIN

TokenSource::REGISTER

TokenSource::PASSWORD_RESET

TokenSource::EMAIL_VERIFICATION
```

Cette approche évite les chaînes de caractères dispersées dans l'application et améliore la robustesse du code.

---

# Validation avant toute exécution

Avant même qu'une Action ne soit exécutée, un middleware vérifie que le modèle demandé est compatible avec le package.

Il s'assure notamment que :

* la classe existe ;
* elle implémente bien le contrat `MailAuthenticatable`.

Cette validation précoce évite de nombreuses erreurs d'exécution.

---

# Une architecture facilement extensible

Le package fournit uniquement les mécanismes de base.

Libre au développeur d'ajouter ensuite :

* authentification par téléphone ;
* authentification OAuth ;
* authentification LDAP ;
* authentification Active Directory ;
* authentification biométrique ;
* Magic Link ;
* authentification à deux facteurs ;
* WebAuthn / Passkeys ;
* authentification SSO ;
* authentification via API externe.

Aucune partie du package ne limite ces évolutions.

---

# Compatible avec toutes les architectures

Le package peut être utilisé dans :

* une API REST ;
* une application SPA ;
* une application mobile ;
* une architecture microservices ;
* une architecture hexagonale ;
* une architecture DDD ;
* une architecture CQRS.

Il ne dépend d'aucune structure particulière.

---

# Les principes de conception

Laravel Authentication Kit repose sur plusieurs principes :

* séparation des responsabilités ;
* faible couplage ;
* forte cohésion ;
* inversion des dépendances ;
* architecture orientée contrats ;
* composants remplaçables ;
* extensibilité.

L'objectif est que chaque composant puisse évoluer indépendamment des autres.

---

# Exemple de flux d'inscription

```
HTTP Request

↓

Validation

↓

Middleware

↓

EmailRegisterAction

↓

Service métier

↓

Création de l'utilisateur

↓

Création du token (Nemesis)

↓

Journalisation

↓

Réponse JSON
```

Chaque étape est indépendante et peut être remplacée ou enrichie sans modifier le reste du pipeline.

---

# Exemple de flux de connexion

```
HTTP Request

↓

Validation

↓

Middleware

↓

EmailLoginAction

↓

Service métier

↓

Vérification des identifiants

↓

Création du token

↓

Journalisation

↓

Réponse JSON
```

---

# Les avantages

* Architecture modulaire.
* Aucun couplage avec un modèle spécifique.
* Compatible avec plusieurs types d'utilisateurs.
* Génération de tokens indépendante via Nemesis.
* Journalisation automatique des événements.
* Actions facilement testables.
* Validation précoce des modèles.
* Architecture orientée interfaces.
* Facile à étendre sans modifier le cœur du package.
* Compatible avec des applications simples comme avec des architectures complexes.

---

# Conclusion

Laravel Authentication Kit n'est pas un framework d'authentification clé en main. C'est une **boîte à outils** qui fournit les fondations nécessaires pour construire un système d'authentification moderne, robuste et évolutif.

En se concentrant uniquement sur les mécanismes fondamentaux — authentification, génération de tokens, journalisation, validation et orchestration des actions — il laisse au développeur une liberté totale pour implémenter sa logique métier. Cette philosophie favorise un code plus propre, plus testable et plus facile à faire évoluer, quel que soit le type d'application ou l'architecture adoptée.

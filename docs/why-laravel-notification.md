# Pourquoi j'ai créé Laravel Notification

## Le problème

Dans la majorité des applications modernes, les notifications sont devenues un élément central : confirmation d'inscription, réinitialisation de mot de passe, alertes de sécurité, rappels, notifications médicales, messages transactionnels, etc.

Pourtant, lorsqu'un projet commence à grandir, plusieurs problèmes apparaissent rapidement.

La plupart des solutions existantes reposent sur des services tiers (Twilio, Firebase, SendGrid, Slack, Telegram, Meta, etc.). Bien qu'efficaces, elles présentent plusieurs limites :

* dépendance forte envers des plateformes externes ;
* coûts qui augmentent avec le volume de notifications ;
* difficulté à changer de fournisseur ;
* logique métier mélangée avec la logique d'envoi ;
* faible contrôle sur les données et sur l'infrastructure.

Dans certains contextes (administration publique, santé, entreprises privées ou infrastructures locales), il est même impossible de dépendre entièrement de services cloud.

Le besoin était donc de disposer d'un système de notification :

* totalement modulaire ;
* extensible ;
* indépendant des fournisseurs ;
* capable de fonctionner sur une infrastructure minimale ;
* suffisamment flexible pour évoluer avec les besoins de l'application.

C'est dans cette optique qu'est né **Laravel Notification**.

---

# Une architecture pensée pour durer

L'objectif n'était pas simplement d'envoyer des e-mails ou des SMS.

L'objectif était de créer une véritable **couche d'abstraction** entre l'application et les différents moyens de communication.

L'application ne connaît jamais Twilio, Firebase ou Telegram.

Elle connaît uniquement le concept de **notification**.

Le reste est délégué à l'architecture.

Cette approche permet de remplacer un fournisseur sans modifier le code métier.

Demain, si un service devient trop coûteux ou disparaît, il suffit d'écrire un nouveau driver.

Aucune logique fonctionnelle n'a besoin d'être réécrite.

---

# Une architecture basée sur l'extensibilité

Le cœur du projet repose sur un principe simple :

> chaque canal de communication est un module indépendant.

Aujourd'hui le système supporte par exemple :

* Email
* SMS
* WhatsApp
* Telegram
* Slack
* Push Notifications
* Base de données

Mais cette liste n'est volontairement pas figée.

Demain, il est possible d'ajouter :

* Discord
* Microsoft Teams
* Signal
* Matrix
* RabbitMQ
* Kafka
* MQTT
* WebSocket
* Système interne d'entreprise
* API propriétaire

sans modifier le reste du framework.

Chaque nouveau canal devient simplement un nouveau module respectant les mêmes contrats.

Cette architecture suit naturellement le principe **Open/Closed** :

* le système reste fermé aux modifications ;
* mais ouvert aux extensions.

---

# Une indépendance vis-à-vis des fournisseurs

L'un des principaux objectifs était d'éviter qu'un choix technique devienne une dépendance permanente.

Prenons l'exemple des SMS.

Aujourd'hui une entreprise peut utiliser Twilio.

Demain elle préférera Vonage.

Après-demain elle disposera de sa propre passerelle GSM.

Avec Laravel Notification, cette évolution ne nécessite pas de réécrire l'application.

Seul le driver change.

Le reste du système reste identique.

La même philosophie s'applique aux e-mails, aux notifications push ou à n'importe quel autre canal.

Ainsi, l'investissement réalisé dans le développement de l'application est protégé contre les évolutions du marché.

---

# Une infrastructure minimale

Toutes les organisations n'ont pas les moyens — ou l'envie — d'utiliser une multitude de services cloud.

Le système a donc été pensé pour fonctionner avec une infrastructure très simple.

Une application Laravel.

Une base de données.

Un serveur.

Cela suffit déjà pour bénéficier :

* d'un historique des notifications ;
* d'une traçabilité complète ;
* d'une gestion des erreurs ;
* d'un système de reprise ;
* d'une planification des envois.

Les services externes deviennent des options, jamais des obligations.

Cette approche est particulièrement intéressante pour les projets déployés :

* sur des serveurs privés ;
* dans des hôpitaux ;
* dans des administrations ;
* dans des entreprises souhaitant conserver la maîtrise de leurs données.

---

# Le destinataire devient autonome

Chaque modèle peut définir lui-même les canaux sur lesquels il souhaite être contacté.

Un utilisateur peut recevoir :

* un e-mail ;
* un SMS ;
* une notification en base de données.

Un médecin peut recevoir :

* deux adresses e-mail différentes ;
* un SMS ;
* une notification interne.

Un administrateur peut recevoir :

* Slack ;
* Telegram ;
* Push Notification.

L'application n'a pas besoin de connaître ces détails.

Elle demande simplement :

> "Envoie cette notification."

Le destinataire décide lui-même des canaux disponibles.

Cette séparation rend le système beaucoup plus flexible et beaucoup plus facile à maintenir.

---

# Une architecture orientée métier

Le système ne manipule pas directement des API externes.

Il manipule des concepts métier :

* Notification
* Canal
* Destination
* Message
* Résultat d'envoi
* Planification
* Session d'envoi

Cette modélisation permet d'obtenir un code beaucoup plus lisible et beaucoup plus proche du langage fonctionnel de l'entreprise.

---

# Une plateforme évolutive

Le projet ne se limite pas à envoyer une notification.

Il constitue une véritable plateforme de communication.

L'architecture permet déjà de gérer :

* les envois immédiats ;
* les envois différés ;
* les notifications récurrentes ;
* plusieurs canaux simultanément ;
* plusieurs destinations pour un même utilisateur ;
* le suivi des succès et des erreurs ;
* l'historique complet des envois.

Et cette base ouvre naturellement la porte à de nombreuses évolutions :

* politiques de retry avancées ;
* priorisation des notifications ;
* limitation du débit (*rate limiting*) ;
* files d'attente distribuées ;
* statistiques détaillées ;
* tableaux de bord ;
* routage intelligent selon le contexte ;
* préférences utilisateur ;
* notifications multilingues.

Aucune de ces fonctionnalités ne nécessite de remettre en cause l'architecture existante.

---

Je te conseille d'ajouter une section **"Exemples d'utilisation"** à la fin du document. Elle permet de montrer la simplicité de l'API sans entrer dans les détails d'implémentation.

---

# Exemples d'utilisation

L'un des objectifs de Laravel Notification était également de proposer une API simple et expressive.

L'application ne manipule jamais directement les différents drivers (Mail, SMS, WhatsApp, etc.). Elle demande simplement au service de notification d'envoyer un message au destinataire, puis l'architecture se charge de sélectionner automatiquement les canaux disponibles.

## Envoi immédiat

Dans cet exemple, une notification est envoyée immédiatement. Le destinataire définit lui-même les canaux qu'il supporte (Email, Base de données, SMS, WhatsApp, etc.).

```php
use AndyDefer\LaravelNotification\Services\NotificationService;
use AndyDefer\LaravelNotification\Records\SendNowRecord;
use AndyDefer\LaravelNotification\ValueObjects\NotificationMessageVO;

$user = TestUser::find(1);

$message = new NotificationMessageVO(
    subject: 'Bienvenue',
    body: 'Votre compte a été créé avec succès.'
);

app(NotificationService::class)->send(
    $user,
    new SendNowRecord(
        message: $message
    )
);
```

Si le modèle possède une adresse e-mail et une destination Database, la notification sera automatiquement envoyée sur les deux canaux.

---

## Limiter les canaux utilisés

Il est également possible de demander explicitement quels canaux doivent être utilisés.

```php
use AndyDefer\LaravelNotification\Channels\MailChannel;
use AndyDefer\LaravelNotification\Channels\DatabaseChannel;
use AndyDefer\LaravelNotification\Collections\FqcnChannelCollection;
use AndyDefer\LaravelNotification\Records\SendNowRecord;
use AndyDefer\LaravelNotification\ValueObjects\FqcnChannelVO;

$channels = new FqcnChannelCollection([
    new FqcnChannelVO(MailChannel::class),
    new FqcnChannelVO(DatabaseChannel::class),
]);

app(NotificationService::class)->send(
    $user,
    new SendNowRecord(
        message: $message,
        channels: $channels
    )
);
```

Dans cet exemple, même si l'utilisateur possède également un numéro de téléphone, seuls les canaux **Mail** et **Database** seront utilisés.

---

## Envoi différé

Une notification peut être planifiée afin d'être envoyée plus tard.

```php
use AndyDefer\LaravelNotification\Records\SendLaterRecord;

app(NotificationService::class)->send(
    $user,
    new SendLaterRecord(
        message: $message,
        delay_minutes: 30
    )
);
```

La notification est alors enregistrée puis exécutée automatiquement par le système de tâches.

---

## Notification récurrente

Il est également possible de programmer une notification qui sera envoyée périodiquement.

```php
use AndyDefer\LaravelNotification\Records\SendRecurringRecord;

app(NotificationService::class)->send(
    $user,
    new SendRecurringRecord(
        message: $message,
        interval_seconds: 3600
    )
);
```

Cette approche est idéale pour les rappels, les alertes ou les tâches automatiques.

---

# Définition des canaux d'un modèle

Chaque modèle décide lui-même des moyens par lesquels il peut être contacté.

Par exemple, un utilisateur peut recevoir les notifications par e-mail et être enregistré dans la base de données.

```php
public function getNotificationChannels(): NotificationRouteCollection
{
    $routes = new NotificationRouteCollection();

    if ($this->email) {
        $routes->add(
            new NotificationRouteVO(
                channelClass: MailChannel::class,
                destination: $this->email,
            )
        );
    }

    $routes->add(
        new NotificationRouteVO(
            channelClass: DatabaseChannel::class,
            destination: 'database',
        )
    );

    return $routes;
}
```

L'application n'a donc jamais besoin de savoir comment contacter un utilisateur. Elle demande simplement au modèle ses canaux disponibles, puis le moteur de notification se charge du reste.

---

# Ajouter un nouveau canal

L'architecture a été conçue pour être facilement extensible.

Ajouter un nouveau moyen de communication consiste simplement à créer :

* un **Channel** ;
* un **Driver** ;
* un **ConfigRecord**.

Une fois enregistrés dans le conteneur Laravel, ils deviennent immédiatement utilisables par l'ensemble du système, sans modifier les composants existants.

Cette approche permet d'ajouter de nouveaux canaux (Discord, Teams, Signal, Kafka, RabbitMQ, WebSocket, etc.) en respectant le principe d'extension sans avoir à réécrire le moteur de notification. Cela garantit une architecture pérenne, capable d'évoluer avec les besoins métier et les évolutions technologiques.


# Une architecture conçue pour accompagner la croissance

Le véritable objectif de Laravel Notification n'était pas de remplacer les solutions existantes.

Il était de construire une fondation suffisamment solide pour que les besoins futurs puissent être ajoutés sans réécrire le système.

En privilégiant les interfaces, les abstractions et les modules indépendants, l'architecture reste simple à comprendre, facile à maintenir et capable d'évoluer avec les contraintes techniques comme avec les besoins métier.

Au final, Laravel Notification n'est pas seulement une bibliothèque d'envoi de notifications. C'est une **plateforme de communication modulaire**, pensée pour offrir aux développeurs et aux organisations la liberté de choisir leur infrastructure, de maîtriser leurs coûts et de faire évoluer leur système sans dépendre d'un fournisseur unique.

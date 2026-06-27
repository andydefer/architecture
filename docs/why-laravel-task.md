# Pourquoi j'ai créé Laravel Task

## Le problème

Lorsque j'ai développé mon package de notifications asynchrones, je me suis rapidement rendu compte que l'envoi d'une notification n'était qu'un cas particulier d'un problème beaucoup plus large.

Une notification est simplement une tâche.

Cette tâche peut être :

* exécutée immédiatement mais en arrière-plan ;
* exécutée à une heure précise ;
* exécutée régulièrement toutes les heures, tous les jours ou toutes les semaines.

En réalité, une application moderne exécute constamment ce type d'opérations.

Par exemple :

* envoyer un email de bienvenue ;
* envoyer une facture demain à 8h00 ;
* envoyer un rapport chaque lundi matin ;
* nettoyer les fichiers temporaires toutes les nuits ;
* recalculer des statistiques toutes les heures ;
* synchroniser des données avec une API externe toutes les 5 minutes.

Je me suis alors posé une question simple :

> Pourquoi créer un système spécifique pour les notifications alors que le véritable besoin est de pouvoir exécuter n'importe quelle tâche de manière asynchrone ?

C'est de cette réflexion qu'est né **Laravel Task**.

---

# Les limites des solutions classiques

Aujourd'hui, lorsqu'un développeur souhaite exécuter du code en arrière-plan avec Laravel, il utilise généralement les Queues.

Mais les Queues introduisent plusieurs contraintes.

## Dépendance à un worker permanent

Les workers doivent rester actifs en permanence.

Par exemple :

```bash
php artisan queue:work
```

Cela suppose :

* un VPS ;
* Supervisor ;
* systemd ;
* Docker ;
* Horizon.

Sur un simple hébergement mutualisé, ce n'est généralement pas possible.

---

## Dépendance à Redis

Pour de bonnes performances, Laravel recommande Redis.

Mais Redis ajoute :

* une installation supplémentaire ;
* un service à maintenir ;
* davantage de mémoire ;
* davantage de coûts.

Pour un petit projet, cette infrastructure est souvent disproportionnée.

---

## Les tâches planifiées

Laravel Scheduler permet bien de lancer des commandes CRON.

Mais imaginons qu'un utilisateur demande :

> Envoie cet email exactement demain à 14h35.

Le Scheduler ne mémorise pas automatiquement cette demande.

Le développeur doit lui-même :

* créer une table,
* enregistrer la demande,
* rechercher les tâches arrivées à échéance,
* gérer les erreurs,
* gérer les retries,
* gérer les annulations.

Le Scheduler lance simplement une commande.

Il ne gère pas les tâches.

---

## Les tâches récurrentes

Même problème.

Supposons :

> Envoyer un rapport toutes les heures.

Il faut enregistrer :

* la fréquence,
* la prochaine exécution,
* la dernière exécution,
* l'état,
* les erreurs,
* l'arrêt de la tâche.

Tout cela doit être développé manuellement.

---

# La vraie réflexion

Je ne voulais pas créer un système d'envoi de notifications.

Je voulais créer un moteur capable d'exécuter n'importe quel travail.

Une notification n'est qu'une tâche.

Une synchronisation est une tâche.

Un import est une tâche.

Un backup est une tâche.

Un nettoyage est une tâche.

Une génération de PDF est une tâche.

Un calcul statistique est une tâche.

Laravel Task est donc devenu un moteur générique.

---

# Laravel Task repose sur deux types de tâches

## 1. Les tâches uniques (Unique Tasks)

Une tâche unique est exécutée une seule fois.

Exemples :

* envoyer un email dans 10 minutes ;
* générer un PDF demain à 8h00 ;
* envoyer une notification à une date précise ;
* supprimer un fichier dans une heure.

Ces tâches possèdent notamment :

* une date d'exécution (`scheduled_at`) ;
* un nombre maximal de tentatives (`max_attempts`) ;
* un délai de grâce (`grace_period`) ;
* un état (Pending, Completed, Failed, Canceled).

Une fois exécutées, elles ne sont plus rejouées.

---

## 2. Les tâches récurrentes (Recurring Tasks)

Une tâche récurrente est exécutée plusieurs fois.

Par exemple :

* toutes les minutes ;
* toutes les heures ;
* chaque jour ;
* chaque semaine ;
* chaque mois.

Chaque tâche possède notamment :

* une date de début (`start_at`) ;
* une date de fin optionnelle (`end_at`) ;
* un intervalle d'exécution (`interval_seconds`) ;
* la date de la dernière exécution (`last_run_at`) ;
* un nombre maximal d'échecs ;
* un état (Waiting, Playing, Paused, Finished, Canceled).

Le moteur décide automatiquement si la tâche doit être rejouée en comparant l'intervalle configuré avec la date de la dernière exécution.

---

# Les tâches deviennent pilotables

Contrairement à une Queue classique, une tâche Laravel Task possède un véritable cycle de vie.

Par exemple, une tâche récurrente peut être :

* démarrée ;
* mise en pause ;
* reprise ;
* arrêtée définitivement ;
* annulée ;
* supprimée.

Il est également possible de :

* modifier son intervalle d'exécution ;
* repousser sa date de début ;
* prolonger sa date de fin ;
* consulter son état ;
* compter les tâches selon leur statut.

Laravel Task ne se contente donc pas d'exécuter du code : il permet de **gérer le cycle de vie complet** des tâches.

---

# Fonctionne avec un simple Cron

L'objectif principal était de rendre l'asynchrone accessible partout.

Il suffit par exemple d'un Cron :

```cron
* * * * * ./vendor/bin/directive process-tasks
```

ou

```cron
* * * * * ./vendor/bin/directive tasks-watch
```

À chaque exécution :

* les tâches programmées sont récupérées ;
* celles arrivées à échéance sont exécutées ;
* les tâches récurrentes sont relancées si leur intervalle est atteint ;
* les erreurs sont enregistrées ;
* les tentatives sont comptabilisées ;
* les états sont mis à jour.

Aucun worker permanent n'est nécessaire.

---

# Une abstraction de l'asynchrone

Laravel Task ne cherche pas à remplacer uniquement les Queues.

Il cherche à offrir une abstraction unique pour toutes les formes d'exécution différée :

* exécution immédiate en arrière-plan ;
* exécution programmée à une date précise ;
* exécution périodique selon un intervalle ;
* gestion des tentatives et des échecs ;
* reprise, pause et annulation des tâches ;
* suivi du cycle de vie complet des traitements.

Ainsi, le package de notifications asynchrones n'est plus qu'une application concrète de Laravel Task : au lieu d'implémenter sa propre logique de planification, il enregistre simplement une tâche (unique ou récurrente) dans le moteur, qui se charge ensuite de son exécution au moment opportun. Cette approche évite de réécrire la même logique dans chaque projet et fournit une base commune pour tous les traitements asynchrones, qu'il s'agisse d'envoyer une notification, de synchroniser des données, de générer des rapports ou d'exécuter des opérations de maintenance.

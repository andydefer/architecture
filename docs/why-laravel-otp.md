# Pourquoi j’ai créé **Laravel OTP**

## Introduction

Le package **Laravel OTP** est né d’un besoin simple mais critique dans les systèmes modernes :
> **sécuriser les actions sensibles sans dépendre de services externes, coûteux ou limités.**

Aujourd’hui, la plupart des applications utilisent des services tiers pour gérer les OTP (One Time Password) : SMS APIs, email providers, WhatsApp gateways, ou encore des solutions SaaS fermées.

Ces solutions fonctionnent, mais elles introduisent plusieurs problèmes :

* dépendance à des services externes
* coûts récurrents par message ou par utilisateur
* manque de contrôle sur la logique métier
* difficulté à adapter les règles de sécurité
* fragmentation de la logique OTP dans plusieurs systèmes

C’est dans ce contexte que ce package a été conçu.

---

## 1. Le problème réel : l’OTP dispersé et dépendant

Dans beaucoup de projets Laravel, la logique OTP est souvent :

* dans un Controller
* dans un Service isolé
* dans un job ou callback SMS
* ou complètement déléguée à un provider externe

Résultat :

- une logique fragile
- difficile à maintenir
- impossible à standardiser

Et surtout :

> L’OTP devient une fonctionnalité externe au lieu d’être une **brique métier du système**

---

## 2. Une volonté claire : reprendre le contrôle

Ce package a été créé pour une raison principale :

###  Recentrer l’OTP dans le domaine applicatif

Avec une approche :

* basée sur des **Services**
* découplée des canaux (SMS, WhatsApp, Email, API…)
* indépendante des providers payants
* entièrement testable et extensible

---

## 3. Un système OTP multi-usages (pas seulement SMS)

Contrairement à une approche classique, ce système OTP est pensé comme un **moteur universel de validation temporaire**.

Il peut être utilisé pour :

###  Sécurité des comptes

* login avec OTP
* double authentification (2FA)
* validation d’email

### Validation de numéro WhatsApp / SMS

* vérification de téléphone
* onboarding utilisateur

###  Email verification

* validation d’inscription
* confirmation d’actions sensibles

###  Opérations sensibles

* confirmation de paiement
* suppression de compte
* changement de mot de passe

### API Security / Microservices

* validation inter-service
* sécurisation de endpoints critiques

> L’OTP devient un **mécanisme générique de confiance temporaire**

---

## 4. Une architecture orientée service et non framework

Le cœur du design repose sur une idée simple :

> Laravel ne doit pas dicter la logique métier OTP, il doit seulement l’exécuter.

C’est pourquoi :

* la génération est isolée
* la persistance est gérée par repository
* la logique métier est dans les services
* les filtres sont typés et structurés
* les règles de validation sont extensibles

---

## 5. Une sécurité renforcée par design

Le système a été conçu pour éviter les erreurs classiques :

### ✔ Expiration stricte

Chaque OTP est limité dans le temps.

### ✔ Tentatives limitées

Un OTP devient invalide après un nombre d’essais défini.

### ✔ Invalidation automatique

Un nouveau OTP invalide les anciens.

### ✔ Vérification de contexte

OTP lié à :

* un utilisateur
* un modèle morph
* un usage précis (PurposeVO)

> Cela empêche les réutilisations croisées ou attaques par relecture.

---

## 6. Découplage total des canaux (SMS, WhatsApp, Email…)

Le package ne dépend d’aucun canal de communication.

Cela permet :

* d’envoyer un OTP par SMS aujourd’hui
* par WhatsApp demain
* par email ou push notification après

Sans modifier la logique métier.

### Exemple conceptuel :

```php
$otp = $otpService->create(
    identifier: $user,
    purpose: PurposeVO::from(['value' => 'login']),
);
```

- Le service OTP ne sait pas *comment* l’utilisateur recevra le code.
- Il sait uniquement *pourquoi* il est généré.

---

## 7. Intégration native avec Laravel Notification

L’un des points forts du système est sa compatibilité naturelle avec :

> **Laravel Notifications**

Cela permet :

* d’envoyer automatiquement le code OTP
* de changer de canal sans toucher au business logic
* de centraliser la logique d’envoi

### Exemple d’usage :

```php
$otp = $otpService->create($user, $purpose);

$user->notify(new OtpNotification($otp->code));
```

Ou encore :

```php
Notification::route('mail', $user->email)
    ->notify(new OtpNotification($otp->code));
```

- L’OTP est généré par le système
- La notification est responsable de la livraison

---

## 8. Un système extensible et propre grâce aux Services

Le cœur du package repose sur une séparation stricte :

* Repository → accès aux données
* Service → logique métier OTP
* Generator → création des codes
* Value Objects → règles métier (Purpose, TTL, attempts)

 **Cela permet :**

* une maintenance simple
* des tests unitaires faciles
* une évolution sans casse

---

## 9. Exemple concret d’utilisation

### Création d’un OTP

```php
$otp = $otpService->create(
    identifier: $user,
    purpose: PurposeVO::from([
        'value' => 'phone_verification',
        'ttl' => 300,
        'maxAttempts' => 3,
    ]),
);
```

---

### Vérification

```php
$isValid = $otpService->verify(
    identifier: $user,
    code: $request->code,
    purpose: PurposeVO::from(['value' => 'phone_verification']),
);
```

---

### Invalidation automatique (nouvel OTP)

```php
$otpService->invalidate($user, $purpose);
```

---

### Limitation de fréquence (anti-spam)

```php
if ($otpService->isRateLimited($user, $purpose)) {
    throw new Exception("Too many OTP requests");
}
```

---

## 10. Une alternative aux solutions payantes

Ce package permet de remplacer des services comme :

* Twilio Verify
* Firebase Phone Auth
* Auth0 OTP
* WhatsApp Business OTP APIs

### Avantages :

* aucun coût par SMS ou code
* contrôle total de la logique
* personnalisation complète
* hébergement interne
* conformité métier maîtrisée

---

## 11. Une vision : OTP comme brique métier universelle

Ce projet n’est pas juste une librairie.

C’est une idée :

> L’OTP ne doit pas être un service externe, mais une brique fondamentale de sécurité dans une application moderne.

---

## Conclusion

J’ai créé **Laravel OTP** pour :

* reprendre le contrôle sur la sécurité applicative
* supprimer la dépendance aux services externes
* standardiser la gestion des OTP
* rendre le système flexible, testable et extensible
* permettre une intégration propre avec Laravel (notamment Notifications)

Mais surtout :

> transformer l’OTP en un véritable outil métier, indépendant des canaux et des fournisseurs.

---

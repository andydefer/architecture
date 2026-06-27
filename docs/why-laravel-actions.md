# Pourquoi Laravel Actions a été conçu ?

## Le problème

Pendant de nombreuses années, l'architecture MVC traditionnelle a parfaitement répondu aux besoins des applications web. Cependant, avec l'évolution des usages, les projets modernes ne se limitent plus à un simple site web.

Aujourd'hui, une même plateforme doit généralement alimenter plusieurs clients :

* une application web ;
* une application iOS ;
* une application Android ;
* parfois une application desktop ;
* des services tiers via une API.

Avec une architecture MVC classique, il devient fréquent de développer deux backends distincts :

* un backend destiné au site web (contrôleurs, vues Blade, sessions, etc.) ;
* un backend API destiné aux applications mobiles.

Cette approche entraîne rapidement plusieurs problèmes :

* duplication de la logique métier ;
* duplication des validations ;
* duplication des règles de sécurité ;
* maintenance plus coûteuse ;
* risque d'incohérences entre les différentes plateformes.

Chaque nouvelle fonctionnalité doit souvent être développée deux fois : une première fois pour le backend web et une seconde fois pour le backend API.

À mesure que le projet grandit, cette duplication devient difficile à maintenir.

---

# La solution : un backend Headless

Laravel Actions a été conçu avec une idée simple :

**écrire la logique métier une seule fois.**

Au lieu de considérer le site web comme le cœur du projet, le backend devient un véritable **backend Headless**, c'est-à-dire un serveur dont la responsabilité est uniquement d'exposer les fonctionnalités métier sous forme d'API.

Toutes les interfaces utilisateurs deviennent alors de simples consommateurs de cette API.

Ainsi :

* le site web ;
* l'application Android ;
* l'application iOS ;
* les futures applications.

consomment exactement le même backend.

La logique métier n'existe qu'à un seul endroit.

---

# Le rôle de Laravel Actions

Laravel Actions fournit une architecture où chaque endpoint HTTP est représenté par une Action unique.

Chaque Action possède :

* sa Request ;
* son Record ;
* sa logique métier ;
* son DTO de réponse.

Cette organisation facilite la création d'API propres, cohérentes et facilement testables.

Chaque fonctionnalité est développée une seule fois, puis réutilisée par l'ensemble des applications clientes.

---

# Le rôle de Laravel Nemesis

Pour qu'un backend Headless soit réellement utilisable par plusieurs plateformes, il faut un système d'authentification adapté.

C'est précisément le rôle de Laravel Nemesis.

Nemesis permet d'authentifier n'importe quel type de client à l'aide de tokens sécurisés :

* application Web ;
* application Android ;
* application iOS ;
* borne interactive ;
* terminal de contrôle ;
* services partenaires ;
* API publiques ou privées.

Contrairement aux solutions centrées uniquement sur l'utilisateur, Nemesis permet également d'authentifier d'autres modèles métier (par exemple un point de contrôle, une caisse, un terminal ou un client API), chacun avec ses propres permissions.

Chaque application consomme donc exactement le même backend tout en bénéficiant de droits adaptés à son rôle.

---

# Une seule source de vérité

En combinant Laravel Actions et Laravel Nemesis, l'architecture devient beaucoup plus simple.

```
                   Backend Laravel
          (Actions + Nemesis + Domain)

                   API REST Headless
                          │
      ┌───────────────────┼───────────────────┐
      │                   │                   │
      ▼                   ▼                   ▼
 Application Web    Application iOS   Application Android
      │                   │                   │
      └───────────────────┼───────────────────┘
                          │
             Même logique métier
             Même validation
             Même authentification
             Même règles métier
```

Toutes les applications utilisent la même API.

- Les règles métier ne sont écrites qu'une seule fois.
- Les validations sont centralisées.
- Les contrôles de sécurité sont identiques sur toutes les plateformes.

**Les évolutions sont immédiatement disponibles pour l'ensemble de l'écosystème.**


## Exemple concret : Une seule Action consommée par plusieurs applications

L'un des principaux avantages de Laravel Actions est que **la logique métier n'est écrite qu'une seule fois**.

Prenons l'exemple d'une fonctionnalité d'inscription d'utilisateur.

Au lieu de développer :

- un contrôleur pour le site web ;
- une API spécifique pour Android ;
- une API spécifique pour iOS ;

nous développons **une seule Action** exposée par notre backend Headless.

### Backend Laravel

#### Route

```php
use function action_route;

Route::post(
    '/api/auth/register',
    action_route(RegisterRequest::class, RegisterAction::class)
);
```

---

#### RegisterRequest

```php
final class RegisterRequest extends AbstractRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required'],
            'email' => ['required', 'email'],
            'password' => ['required', 'min:8'],
        ];
    }

    public function getRecord(): AbstractRecord
    {
        return RegisterRecord::from([
            'name' => $this->input('name'),
            'email' => $this->input('email'),
            'password' => $this->input('password'),
        ]);
    }
}
```

---

#### RegisterAction

```php
final class RegisterAction extends AbstractAction
{
    public function __construct(
        private readonly UserRepository $users,
        private readonly NemesisService $nemesis,
    ) {}

    protected function handle(AbstractRecord $record): ResponseFactory
    {
        /** @var RegisterRecord $record */

        $user = $this->users->create([
            'name' => $record->name,
            'email' => $record->email,
            'password' => Hash::make($record->password),
        ]);

        [$token, $plainToken] = $this->nemesis->createWithPlainToken(
            NemesisTokenRecord::from([
                'name' => 'Application',
                'source' => 'mobile',
            ]),
            $user
        );

        return ResponseFactory::json(
            RegisterData::from([
                'user' => UserData::from($user),
                'token' => $plainToken,
            ]),
            201
        );
    }
}
```

---

# Application Web

Le frontend web appelle exactement la même API.

```javascript
await fetch("/api/auth/register", {
    method: "POST",
    headers: {
        "Content-Type": "application/json"
    },
    body: JSON.stringify({
        name: "Andy",
        email: "andy@example.com",
        password: "password123"
    })
});
```

---

# Android (Jetpack Compose + Ktor)

```kotlin
val response = httpClient.post("/api/auth/register") {
    contentType(ContentType.Application.Json)

    setBody(
        RegisterRequest(
            name = "Andy",
            email = "andy@example.com",
            password = "password123"
        )
    )
}
```

---

# iOS (Swift)

```swift
let body = RegisterRequest(
    name: "Andy",
    email: "andy@example.com",
    password: "password123"
)

let data = try JSONEncoder().encode(body)

var request = URLRequest(
    url: URL(string: "/api/auth/register")!
)

request.httpMethod = "POST"
request.httpBody = data
request.setValue(
    "application/json",
    forHTTPHeaderField: "Content-Type"
)

let (_, _) = try await URLSession.shared.data(for: request)
```

---

## Une seule logique métier

Les trois applications utilisent exactement le même endpoint :

```
POST /api/auth/register
```

La validation est effectuée une seule fois.

La création de l'utilisateur est effectuée une seule fois.

La génération du token Nemesis est effectuée une seule fois.

Le format de réponse est identique pour tous les clients.

Ainsi, lorsqu'une règle métier évolue (par exemple l'ajout d'une vérification d'âge ou l'envoi d'un e-mail de confirmation), une seule Action est modifiée. Toutes les applications bénéficient immédiatement de cette évolution sans qu'il soit nécessaire de dupliquer la logique dans plusieurs backends.


---

# Les bénéfices

Cette approche apporte plusieurs avantages :

* suppression de la duplication du code métier ;
* réduction des coûts de maintenance ;
* cohérence entre toutes les plateformes ;
* développement plus rapide des nouvelles fonctionnalités ;
* architecture facilement extensible ;
* meilleure testabilité ;
* possibilité d'ajouter de nouveaux clients (desktop, IoT, partenaires, etc.) sans modifier la logique métier.

---

# Conclusion

Laravel Actions n'a pas été conçu pour remplacer les contrôleurs Laravel uniquement par préférence architecturale.

Il répond à un problème concret rencontré dans les projets modernes : la multiplication des interfaces clientes et la duplication du code entre plusieurs backends.

Associé à Laravel Nemesis, il permet de construire un **backend Headless unique**, sécurisé et maintenable, capable de servir simultanément les applications web, Android, iOS et tout autre client, tout en garantissant une seule source de vérité pour l'ensemble de la logique métier.

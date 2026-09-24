# Standard de code personnel

> **Version :** 1.0  
> **Périmètre :** PHP, Laravel, TypeScript et React  
> **Objectif :** produire du code prévisible, lisible, sûr et économe en ressources.

## Sommaire

- [1. Principes Clean Code](#1-principes-clean-code)
- [2. Lisibilité et fonctionnement cognitif](#2-lisibilité-et-fonctionnement-cognitif)
- [3. Bande passante](#3-bande-passante)
- [4. Latence](#4-latence)
- [5. RAM](#5-ram)
- [6. Espace disque](#6-espace-disque)
- [7. CPU](#7-cpu)
- [8. GPU](#8-gpu)
- [9. Sécurité OWASP Top 10](#9-sécurité-owasp-top-10)
- [10. Conventions PHP et Laravel](#10-conventions-php-et-laravel)
- [11. Conventions TypeScript et React](#11-conventions-typescript-et-react)
- [12. Accessibilité et tests](#12-accessibilité-et-tests)
- [Annexe A — Checklist avant livraison](#annexe-a--checklist-avant-livraison)

## Comment lire ce standard

- **Une règle = un bloc.** Chaque bloc donne une règle, puis un exemple.
- `Mauvais ❌` montre le comportement interdit ou fragile.
- `Bon ✅` montre le comportement attendu.
- Le code utilise des **apostrophes simples** quand une chaîne est nécessaire.
- La longueur cible d'une ligne de code est de **80 caractères**.
- Une exception est documentée dans le code ou dans la revue.

## Tableau des règles cardinales

| Priorité | Règle | Vérification |
| --- | --- | --- |
| 1 | Le code doit être compréhensible. | Relire chaque nom et chaque fonction. |
| 2 | Le serveur doit valider les entrées. | Tester validation et autorisation. |
| 3 | Une requête doit être mesurée. | Vérifier N+1, index et temps. |
| 4 | Une ressource doit être chargée au besoin. | Vérifier payload, mémoire et chunks. |
| 5 | Une exception doit être explicite. | Documenter la raison et la date. |

---

# 1. Principes Clean Code

Ces règles reprennent les principes de Clean Code associés à Robert C. Martin.
Elles s'appliquent au code métier, aux contrôleurs, aux composants et aux tests.

## 1.1 Donner des noms significatifs

**Règle :** un nom décrit l'intention, le contenu et l'unité si nécessaire.

**Mauvais ❌**

```php
$u = User::find($id);
$d = $u->created_at;
```

**Bon ✅**

```php
$user = User::findOrFail($userId);
$createdAt = $user->created_at;
```

## 1.2 Éviter les abréviations

**Règle :** écrire le mot complet, sauf acronyme métier officiellement défini.

**Mauvais ❌**

```ts
const usrSvc = new UsrSvc();
```

**Bon ✅**

```ts
const userService = new UserService();
```

## 1.3 Écrire des fonctions courtes

**Règle :** une fonction fait une action lisible de bout en bout.

**Mauvais ❌**

```php
public function saveOrder(Request $request): RedirectResponse
{
    $data = $request->all();
    $order = Order::create($data);
    Mail::to($order->user)->send(new OrderCreated($order));
    Log::info('Order created', ['id' => $order->id]);
    return redirect()->route('orders.show', $order);
}
```

**Bon ✅**

```php
public function store(StoreOrderRequest $request): RedirectResponse
{
    $order = $this->orderCreator->create($request->validated());

    return redirect()->route('orders.show', $order);
}
```

## 1.4 Respecter le principe de responsabilité unique

**Règle :** une classe ou un fichier possède une raison principale de changer.

**Mauvais ❌**

```ts
export function UserPage() {
  // Récupération HTTP, validation, rendu et formatage au même endroit.
  return <div>Utilisateur</div>;
}
```

**Bon ✅**

```ts
export function UserPage() {
  const { user, isLoading } = useUser();

  if (isLoading) {
    return <p>Chargement...</p>;
  }

  return <UserCard user={user} />;
}
```

## 1.5 Réduire les commentaires

**Règle :** le code explique le quoi. Un commentaire explique uniquement le pourquoi.

**Mauvais ❌**

```php
// Incrémenter i de 1.
$index++;
```

**Bon ✅**

```php
// Le fournisseur limite les appels à 10 par seconde.
usleep(100000);
```

## 1.6 Ne pas dupliquer le code

**Règle :** une logique métier répétée est extraite dans une fonction ou un service.

**Mauvais ❌**

```php
$totalWithTax = $price * 1.2;
// Dans un autre fichier :
$anotherTotalWithTax = $anotherPrice * 1.2;
```

**Bon ✅**

```php
function calculateTotalWithTax(float $price, float $taxRate): float
{
    return $price * (1 + $taxRate);
}
```

## 1.7 Préférer la simplicité

**Règle :** choisir la solution la plus simple qui respecte le besoin.

**Mauvais ❌**

```ts
const isEnabled = Boolean(Boolean(config?.feature?.enabled ?? false));
```

**Bon ✅**

```ts
const isEnabled = config?.feature?.enabled ?? false;
```

## 1.8 Limiter les effets de bord

**Règle :** une fonction indique clairement les données qu'elle lit et modifie.

**Mauvais ❌**

```php
function formatUser(array $user): array
{
    $GLOBALS['lastUser'] = $user;
    $user['name'] = trim($user['name']);
    return $user;
}
```

**Bon ✅**

```php
function normalizeUser(array $user): array
{
    return [
        ...$user,
        'name' => trim($user['name']),
    ];
}
```

## 1.9 Échouer tôt et explicitement

**Règle :** valider les préconditions au début et retourner une erreur claire.

**Mauvais ❌**

```php
function findInvoice(?int $invoiceId): Invoice
{
    return Invoice::find($invoiceId);
}
```

**Bon ✅**

```php
function findInvoice(int $invoiceId): Invoice
{
    return Invoice::findOrFail($invoiceId);
}
```

## 1.10 Éviter les valeurs magiques

**Règle :** donner un nom aux seuils, statuts et durées.

**Mauvais ❌**

```ts
if (attempts > 3) {
  throw new Error('Too many attempts');
}
```

**Bon ✅**

```ts
const MAX_LOGIN_ATTEMPTS = 3;

if (attempts > MAX_LOGIN_ATTEMPTS) {
  throw new Error('Too many attempts');
}
```

---

# 2. Lisibilité et fonctionnement cognitif

Le code est destiné à être relu. Il doit rester stable, linéaire et prévisible.

## 2.1 Limiter la longueur des lignes

**Règle :** viser 80 caractères. Ne jamais dépasser 100 caractères sans raison.

**Mauvais ❌**

```php
return $this->repository->findActiveUsersByOrganizationAndPermission($organizationId, $permission);
```

**Bon ✅**

```php
return $this->repository->findActiveUsers(
    organizationId: $organizationId,
    permission: $permission,
);
```

## 2.2 Utiliser une indentation constante

**Règle :** PHP utilise 4 espaces. TypeScript utilise 2 espaces. Jamais de tabulation.

**Mauvais ❌**

```ts
function getLabel(value: string): string {
    return value.trim();
}
```

**Bon ✅**

```ts
function getLabel(value: string): string {
  return value.trim();
}
```

## 2.3 Utiliser un nom explicite

**Règle :** le nom doit être compréhensible sans lire son implémentation.

**Mauvais ❌**

```ts
function p(d: number): number {
  return d * 0.2;
}
```

**Bon ✅**

```ts
function calculateTax(amount: number): number {
  return amount * 0.2;
}
```

## 2.4 Garder une structure de fichier prévisible

**Règle :** garder cet ordre : imports, types, constantes, composant ou classe,
fonctions auxiliaires, export.

**Bon ✅**

```ts
import { useMemo } from 'react';

import type { Product } from './types';

const EMPTY_PRODUCTS: Product[] = [];

type ProductListProps = {
  products: Product[];
};

export function ProductList({ products }: ProductListProps) {
  const visibleProducts = useMemo(
    () => products.filter((product) => product.isVisible),
    [products],
  );

  return <ProductRows products={visibleProducts} />;
}
```

## 2.5 Donner une seule responsabilité à un fichier

**Règle :** séparer le composant, son type, son accès HTTP et son test si le fichier
devient difficile à parcourir.

**Mauvais ❌**

```text
UserPage.tsx
├── composant React
├── client HTTP
├── types de données
├── règles de validation
└── tests
```

**Bon ✅**

```text
users/
├── UserPage.tsx
├── userApi.ts
├── userTypes.ts
├── userValidation.ts
└── UserPage.test.tsx
```

## 2.6 Écrire une condition par ligne

**Règle :** nommer les conditions complexes avant de les utiliser.

**Mauvais ❌**

```ts
if (user && user.isActive && user.roles.includes('admin')) {
  showAdminPanel();
}
```

**Bon ✅**

```ts
const isActiveAdmin =
  user?.isActive === true && user.roles.includes('admin');

if (isActiveAdmin) {
  showAdminPanel();
}
```

---

# 3. Bande passante

## 3.1 Envoyer des payloads API minimaux

**Règle :** ne renvoyer que les champs utilisés par l'écran.

**Mauvais ❌**

```php
return User::with('profile', 'roles', 'permissions', 'sessions')->get();
```

**Bon ✅**

```php
return User::query()
    ->select(['id', 'name', 'email'])
    ->where('is_active', true)
    ->get();
```

## 3.2 Activer la compression HTTP

**Règle :** compresser les réponses textuelles au niveau du serveur ou du proxy.

**Mauvais ❌**

```nginx
location /api/ {
    proxy_pass http://app;
}
```

**Bon ✅**

```nginx
gzip on;
gzip_types application/json text/css application/javascript;
gzip_min_length 1024;
```

## 3.3 Charger les composants React à la demande

**Règle :** utiliser le lazy loading pour les écrans rarement visités.

**Mauvais ❌**

```ts
import AdminReport from './AdminReport';
```

**Bon ✅**

```tsx
import { lazy, Suspense } from 'react';

const AdminReport = lazy(() => import('./AdminReport'));

export function AdminRoute() {
  return (
    <Suspense fallback={<p>Chargement du rapport...</p>}>
      <AdminReport />
    </Suspense>
  );
}
```

## 3.4 Découper le code JavaScript

**Règle :** produire des chunks par route ou fonctionnalité.

**Bon ✅**

```ts
const SettingsPage = lazy(() => import('./pages/SettingsPage'));
const BillingPage = lazy(() => import('./pages/BillingPage'));
```

## 3.5 Utiliser le cache HTTP et ETag

**Règle :** permettre au client de réutiliser une réponse inchangée.

**Bon ✅**

```php
return response()->json($payload)
    ->setEtag(hash('sha256', json_encode($payload)))
    ->setMaxAge(60);
```

## 3.6 Éviter les librairies lourdes

**Règle :** mesurer la taille de la dépendance avant de l'ajouter.

**Mauvais ❌**

```ts
import entireUtilityLibrary from 'entire-utility-library';
```

**Bon ✅**

```ts
import { formatDate } from './formatDate';
```

---

# 4. Latence

## 4.1 Sélectionner uniquement les colonnes nécessaires

**Règle :** un écran ne doit pas charger des colonnes inutilisées.

**Bon ✅**

```php
$orders = Order::query()
    ->select(['id', 'user_id', 'total', 'created_at'])
    ->where('user_id', $userId)
    ->latest()
    ->get();
```

## 4.2 Précharger les relations nécessaires

**Règle :** utiliser eager loading lorsque la relation est affichée.

**Mauvais ❌**

```php
foreach ($orders as $order) {
    echo $order->user->name;
}
```

**Bon ✅**

```php
$orders = Order::with('user:id,name')
    ->select(['id', 'user_id', 'total'])
    ->get();
```

## 4.3 Éviter les requêtes N+1

**Règle :** vérifier le nombre de requêtes avec le profiler ou un test SQL.

**Bon ✅**

```php
$this->assertDatabaseCount('orders', 3);

Order::preventLazyLoading();
```

## 4.4 Ajouter les index utiles

**Règle :** indexer les colonnes utilisées pour filtrer et trier souvent.

**Bon ✅**

```php
Schema::table('orders', function (Blueprint $table): void {
    $table->index(['user_id', 'created_at']);
});
```

## 4.5 Utiliser le cache Laravel avec une durée explicite

**Règle :** définir la clé, la durée et l'invalidation.

**Bon ✅**

```php
$categories = Cache::remember(
    key: 'categories:active',
    ttl: now()->addHour(),
    callback: fn (): Collection => Category::active()->get(),
);
```

## 4.6 Debouncer les recherches React

**Règle :** attendre la fin de la saisie avant d'appeler l'API.

**Bon ✅**

```tsx
const debouncedSearch = useMemo(
  () => debounce((value: string) => searchProducts(value), 300),
  [],
);
```

## 4.7 Throttler les événements fréquents

**Règle :** limiter les événements de scroll et de resize.

**Bon ✅**

```ts
window.addEventListener('resize', handleResize, { passive: true });
```

---

# 5. RAM

## 5.1 Ne pas charger des données inutiles

**Règle :** filtrer en base, puis transférer les colonnes nécessaires.

**Mauvais ❌**

```php
$users = User::all()->filter(fn (User $user): bool => $user->is_active);
```

**Bon ✅**

```php
$users = User::query()
    ->where('is_active', true)
    ->select(['id', 'email'])
    ->get();
```

## 5.2 Traiter les grandes collections par morceaux

**Règle :** utiliser `chunkById` pour les mises à jour massives.

**Bon ✅**

```php
User::query()
    ->whereNull('normalized_email')
    ->chunkById(500, function (Collection $users): void {
        foreach ($users as $user) {
            $user->updateQuietly([
                'normalized_email' => strtolower($user->email),
            ]);
        }
    });
```

## 5.3 Utiliser un générateur PHP

**Règle :** produire une valeur à la fois pour les flux volumineux.

**Bon ✅**

```php
function readLines(string $path): Generator
{
    $handle = fopen($path, 'rb');

    while (($line = fgets($handle)) !== false) {
        yield rtrim($line, "\r\n");
    }

    fclose($handle);
}
```

## 5.4 Éviter les clones inutiles

**Règle :** ne copier une structure que si l'isolation est nécessaire.

**Mauvais ❌**

```ts
const copiedUsers = [...users].map((user) => ({ ...user }));
```

**Bon ✅**

```ts
const activeUsers = users.filter((user) => user.isActive);
```

## 5.5 Libérer les ressources

**Règle :** fermer les handles et annuler les abonnements.

**Bon ✅**

```tsx
useEffect(() => {
  const controller = new AbortController();

  loadUsers({ signal: controller.signal });

  return () => controller.abort();
}, []);
```

---

# 6. Espace disque

## 6.1 Garder uniquement les dépendances nécessaires

**Règle :** chaque dépendance a un usage réel, documenté et maintenu.

**Mauvais ❌**

```json
{
  "dependencies": {
    "date-fns": "^4.0.0",
    "moment": "^2.30.1"
  }
}
```

**Bon ✅**

```json
{
  "dependencies": {
    "date-fns": "^4.0.0"
  }
}
```

## 6.2 Ne pas dupliquer les librairies

**Règle :** une fonction standard possède une seule implémentation partagée.

**Mauvais ❌**

```text
src/utils/date.ts
src/helpers/date.ts
src/shared/date.ts
```

**Bon ✅**

```text
src/shared/date/formatDate.ts
```

## 6.3 Nettoyer les artefacts générés

**Règle :** ne pas versionner les caches, logs, builds ou secrets.

**Bon ✅**

```gitignore
/vendor/
/node_modules/
/storage/logs/*.log
.env
.env.*
!.env.example
```

## 6.4 Auditer les dépendances

**Règle :** supprimer les dépendances inutilisées et corriger les vulnérabilités.

**Bon ✅**

```bash
composer audit
npm audit
```

---

# 7. CPU

## 7.1 Choisir un algorithme simple

**Règle :** choisir une complexité adaptée au volume réel avant d'optimiser.

**Mauvais ❌**

```ts
for (const user of users) {
  for (const order of orders) {
    if (order.userId === user.id) {
      user.orders.push(order);
    }
  }
}
```

**Bon ✅**

```ts
const ordersByUserId = new Map<number, Order[]>();

for (const order of orders) {
  const userOrders = ordersByUserId.get(order.userId) ?? [];
  userOrders.push(order);
  ordersByUserId.set(order.userId, userOrders);
}
```

## 7.2 Éviter les boucles imbriquées inutiles

**Règle :** indexer les données avant la recherche répétée.

**Bon ✅**

```php
$ordersByUserId = $orders->groupBy('user_id');

foreach ($users as $user) {
    $userOrders = $ordersByUserId->get($user->id, collect());
}
```

## 7.3 Utiliser `useMemo` avec discernement

**Règle :** mémoriser seulement un calcul mesuré comme coûteux.

**Bon ✅**

```tsx
const sortedProducts = useMemo(
  () => sortProducts(products, sortOrder),
  [products, sortOrder],
);
```

## 7.4 Utiliser `useCallback` avec discernement

**Règle :** mémoriser un callback uniquement si un enfant en dépend réellement.

**Bon ✅**

```tsx
const handleSelect = useCallback((productId: number) => {
  setSelectedProductId(productId);
}, []);
```

## 7.5 Ne pas optimiser avant de mesurer

**Règle :** mesurer avec un profiler, noter le problème, puis vérifier le gain.

**Bon ✅**

```text
1. Reproduire le ralentissement.
2. Mesurer le temps et la fréquence.
3. Modifier une seule chose.
4. Mesurer à nouveau.
5. Garder la modification si le gain est réel.
```

---

# 8. GPU

## 8.1 Animer seulement `transform` et `opacity`

**Règle :** éviter les animations qui déclenchent un recalcul de mise en page.

**Mauvais ❌**

```css
.card {
  transition: width 300ms ease, box-shadow 300ms ease;
}
```

**Bon ✅**

```css
.card {
  transition: transform 200ms ease, opacity 200ms ease;
}

.card:hover {
  transform: translateY(-2px);
}
```

## 8.2 Éviter les effets lourds

**Règle :** limiter les filtres, flous, ombres multiples et grandes surfaces animées.

**Mauvais ❌**

```css
.hero {
  filter: blur(20px) drop-shadow(0 0 30px #000);
}
```

**Bon ✅**

```css
.hero {
  box-shadow: 0 2px 8px rgb(0 0 0 / 15%);
}
```

## 8.3 Respecter `prefers-reduced-motion`

**Règle :** supprimer les mouvements non nécessaires sur demande du système.

**Bon ✅**

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    transition-duration: 0.01ms !important;
    animation-duration: 0.01ms !important;
  }
}
```

---

# 9. Sécurité OWASP Top 10

La sécurité est une condition de livraison. Les contrôles sont faits côté serveur,
même si l'interface les fait aussi.

## 9.1 Contrôler les injections SQL

**Règle :** utiliser Eloquent ou des bindings. Ne jamais concaténer une entrée SQL.

**Mauvais ❌**

```php
$users = DB::select("SELECT * FROM users WHERE email = '$email'");
```

**Bon ✅**

```php
$users = DB::select(
    'SELECT id, email FROM users WHERE email = ?',
    [$email],
);
```

## 9.2 Contrôler les XSS

**Règle :** laisser Blade échapper les valeurs et éviter `dangerouslySetInnerHTML`.

**Mauvais ❌**

```blade
{!! $user->biography !!}
```

```tsx
<div dangerouslySetInnerHTML={{ __html: biography }} />
```

**Bon ✅**

```blade
{{ $user->biography }}
```

```tsx
<p>{biography}</p>
```

## 9.3 Contrôler le CSRF

**Règle :** conserver la protection CSRF Laravel sur les formulaires et requêtes web.

**Mauvais ❌**

```blade
<form method="POST" action="/profile">
    <input name="name">
</form>
```

**Bon ✅**

```blade
<form method="POST" action="{{ route('profile.update') }}">
    @csrf
    @method('PATCH')
    <input name="name">
</form>
```

## 9.4 Authentifier chaque accès privé

**Règle :** une route privée utilise un mécanisme d'authentification explicite.

**Mauvais ❌**

```php
Route::get('/account', [AccountController::class, 'show']);
```

**Bon ✅**

```php
Route::middleware('auth')->get(
    '/account',
    [AccountController::class, 'show'],
);
```

## 9.5 Contrôler les autorisations

**Règle :** vérifier la propriété ou la permission côté serveur.

**Mauvais ❌**

```php
public function show(Order $order): Order
{
    return $order;
}
```

**Bon ✅**

```php
public function show(Order $order): Order
{
    $this->authorize('view', $order);

    return $order;
}
```

## 9.6 Valider strictement les entrées

**Règle :** utiliser un Form Request avec types, formats et limites explicites.

**Bon ✅**

```php
final class StoreUserRequest extends FormRequest
{
    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:100'],
            'email' => ['required', 'email:rfc', 'max:255'],
        ];
    }
}
```

## 9.7 Garder les secrets hors du code

**Règle :** lire les secrets depuis l'environnement et ne jamais les committer.

**Mauvais ❌**

```php
$apiKey = 'live-secret-key';
```

**Bon ✅**

```php
$apiKey = config('services.payment.key');
```

```dotenv
PAYMENT_KEY=
```

## 9.8 Auditer les dépendances

**Règle :** mettre à jour les dépendances et vérifier les avis de sécurité.

**Bon ✅**

```bash
composer audit --no-interaction
npm audit --omit=dev
```

## 9.9 Limiter le débit

**Règle :** protéger les endpoints sensibles contre les essais répétés.

**Bon ✅**

```php
Route::middleware('throttle:login')->post(
    '/login',
    [AuthenticatedSessionController::class, 'store'],
);
```

## 9.10 Ajouter des headers de sécurité

**Règle :** configurer au minimum HTTPS, HSTS, CSP et anti-embedding.

**Bon ✅**

```php
return $next($request)
    ->header('X-Content-Type-Options', 'nosniff')
    ->header('X-Frame-Options', 'DENY')
    ->header('Referrer-Policy', 'strict-origin-when-cross-origin')
    ->header('Content-Security-Policy', "default-src 'self'");
```

## 9.11 Protéger les logs

**Règle :** ne jamais écrire un mot de passe, un token ou une donnée sensible.

**Mauvais ❌**

```php
Log::info('Login request', ['password' => $request->password]);
```

**Bon ✅**

```php
Log::info('Login request', ['user_id' => $user->id]);
```

## 9.12 Gérer les erreurs sans fuite d'information

**Règle :** montrer un message générique au client et conserver les détails côté serveur.

**Bon ✅**

```php
report($exception);

return response()->json([
    'message' => 'Une erreur est survenue.',
], 500);
```

---

# 10. Conventions PHP et Laravel

## 10.1 Respecter PSR-12

**Règle :** formater avec Laravel Pint et respecter PSR-12.

**Bon ✅**

```bash
./vendor/bin/pint --test
./vendor/bin/pint
```

## 10.2 Activer les types stricts

**Règle :** placer `strict_types` en première instruction du fichier PHP.

**Bon ✅**

```php
<?php

declare(strict_types=1);

function add(int $first, int $second): int
{
    return $first + $second;
}
```

## 10.3 Déclarer les types de paramètres et de retours

**Règle :** typer les entrées, les sorties et les propriétés.

**Mauvais ❌**

```php
public function total($items)
{
    return $items->sum('price');
}
```

**Bon ✅**

```php
public function total(Collection $items): float
{
    return (float) $items->sum('price');
}
```

## 10.4 Utiliser les Form Requests

**Règle :** ne pas valider les données utilisateur dans un contrôleur volumineux.

**Bon ✅**

```php
public function store(StoreProductRequest $request): JsonResponse
{
    $product = Product::create($request->validated());

    return response()->json($product, 201);
}
```

## 10.5 Utiliser les conventions Laravel

**Règle :** suivre les noms et emplacements Laravel avant de créer une abstraction.

**Bon ✅**

```text
app/Http/Controllers/
app/Http/Requests/
app/Models/
app/Policies/
app/Services/
routes/web.php
routes/api.php
```

## 10.6 Utiliser les policies pour l'autorisation

**Règle :** centraliser la décision d'accès dans une Policy.

**Bon ✅**

```php
public function update(User $user, Project $project): bool
{
    return $project->owner_id === $user->id;
}
```

## 10.7 Utiliser les jobs pour le travail asynchrone

**Règle :** déplacer les tâches lentes ou externes dans une queue.

**Bon ✅**

```php
SendInvoiceEmail::dispatch($invoice)->onQueue('emails');
```

---

# 11. Conventions TypeScript et React

## 11.1 Activer le mode strict

**Règle :** le compilateur doit refuser les types implicites dangereux.

**Bon ✅**

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true
  }
}
```

## 11.2 Utiliser des types explicites

**Règle :** typer les props, les retours publics et les données API.

**Mauvais ❌**

```tsx
export function UserCard({ user }) {
  return <p>{user.name}</p>;
}
```

**Bon ✅**

```tsx
type User = {
  id: number;
  name: string;
};

type UserCardProps = {
  user: User;
};

export function UserCard({ user }: UserCardProps) {
  return <p>{user.name}</p>;
}
```

## 11.3 Utiliser des composants fonctionnels

**Règle :** préférer une fonction simple et composable.

**Bon ✅**

```tsx
export function SaveButton({ onSave }: { onSave: () => void }) {
  return <button onClick={onSave}>Enregistrer</button>;
}
```

## 11.4 Respecter le naming

**Règle :** composants en `PascalCase`, fonctions en `camelCase`, constantes en
`SCREAMING_SNAKE_CASE`.

**Mauvais ❌**

```ts
const maxItems = 20;
function User_card() {}
```

**Bon ✅**

```ts
const MAX_ITEMS = 20;
function UserCard() {}
function loadUserProfile() {}
```

## 11.5 Ne pas muter l'état React

**Règle :** créer une nouvelle valeur lors d'une mise à jour.

**Mauvais ❌**

```tsx
items.push(newItem);
setItems(items);
```

**Bon ✅**

```tsx
setItems((currentItems) => [...currentItems, newItem]);
```

## 11.6 Gérer les états visibles

**Règle :** prévoir au minimum chargement, erreur, vide et succès.

**Bon ✅**

```tsx
if (isLoading) {
  return <p>Chargement...</p>;
}

if (error) {
  return <p role="alert">Impossible de charger les données.</p>;
}

if (items.length === 0) {
  return <p>Aucun élément.</p>;
}

return <ItemList items={items} />;
```

---

# 12. Accessibilité et tests

## 12.1 Utiliser le HTML sémantique

**Règle :** utiliser l'élément HTML qui décrit l'action ou la structure.

**Mauvais ❌**

```tsx
<div onClick={handleSubmit}>Enregistrer</div>
```

**Bon ✅**

```tsx
<button type="button" onClick={handleSubmit}>
  Enregistrer
</button>
```

## 12.2 Associer chaque champ à un label

**Règle :** un champ de formulaire possède un label visible et associé.

**Bon ✅**

```tsx
<label htmlFor="email">Adresse e-mail</label>
<input id="email" name="email" type="email" />
```

## 12.3 Prévoir le clavier et le focus

**Règle :** toute action utilisable à la souris doit être utilisable au clavier.

**Bon ✅**

```css
:focus-visible {
  outline: 3px solid #1456a0;
  outline-offset: 2px;
}
```

## 12.4 Tester le comportement public

**Règle :** tester ce que voit et fait l'utilisateur, pas l'implémentation interne.

**Bon ✅**

```tsx
it('affiche le nom de l'utilisateur', () => {
  render(<UserCard user={{ id: 1, name: 'Ada' }} />);

  expect(screen.getByText('Ada')).toBeInTheDocument();
});
```

## 12.5 Tester les règles métier Laravel

**Règle :** tester succès, validation, autorisation et erreur.

**Bon ✅**

```php
it('refuse une adresse e-mail invalide', function (): void {
    $response = $this->postJson('/api/users', [
        'name' => 'Ada',
        'email' => 'adresse-invalide',
    ]);

    $response->assertStatus(422)
        ->assertJsonValidationErrors(['email']);
});
```

## 12.6 Automatiser les contrôles

**Règle :** la CI lance formatage, analyse statique, tests et audit.

**Bon ✅**

```text
1. Pint ou Prettier.
2. PHPStan ou Larastan.
3. TypeScript sans erreur.
4. Tests unitaires et fonctionnels.
5. Tests d'accessibilité ciblés.
6. Audit des dépendances.
```

---

# Annexe A — Checklist avant livraison

## Code

- [ ] Chaque nom est explicite.
- [ ] Chaque fonction a une responsabilité claire.
- [ ] Les lignes restent sous 100 caractères.
- [ ] PHP utilise 4 espaces.
- [ ] TypeScript utilise 2 espaces.
- [ ] Les chaînes de code utilisent des apostrophes simples.
- [ ] Les commentaires expliquent seulement le pourquoi.
- [ ] Les duplications ont été supprimées.

## Performance

- [ ] Le payload API contient uniquement les champs utilisés.
- [ ] Les réponses textuelles sont compressées.
- [ ] Les routes lentes utilisent le lazy loading si nécessaire.
- [ ] Les requêtes utilisent `select` et eager loading.
- [ ] Aucun N+1 n'est détecté.
- [ ] Les index correspondent aux filtres fréquents.
- [ ] Les grandes collections sont traitées par morceaux.
- [ ] Les animations utilisent `transform` ou `opacity`.

## Sécurité

- [ ] Les requêtes utilisent Eloquent ou des bindings.
- [ ] Les sorties HTML sont échappées.
- [ ] Le CSRF est actif.
- [ ] L'authentification est vérifiée.
- [ ] L'autorisation est vérifiée.
- [ ] Les entrées sont validées côté serveur.
- [ ] Aucun secret n'est dans le dépôt.
- [ ] Le rate limiting protège les endpoints sensibles.
- [ ] Les headers de sécurité sont présents.
- [ ] Les dépendances sont auditées.

## Accessibilité et qualité

- [ ] Les boutons et liens sont sémantiques.
- [ ] Les champs ont un label.
- [ ] Le parcours clavier fonctionne.
- [ ] Les états chargement, erreur, vide et succès existent.
- [ ] Les tests couvrent le comportement principal.
- [ ] La CI est verte.

## Règle finale

> Si une règle doit être ignorée, écrire la raison, la portée et la date de révision.

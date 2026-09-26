# Guide Complet d'Éco-Conception Logicielle : Spécial PHP & Laravel

Ce document regroupe l'intégralité des stratégies d'optimisation énergétique logicielle, classées par ordre de complexité, avec une application dédiée à l'écosystème **PHP et Laravel**.

---

## 📊 Matrice de Synthèse Énergétique

| Niveau | Domaine d'action | Cible Prioritaire | Gain Énergétique Potentiel | Effort de Dev |
| :--- | :--- | :--- | :--- | :--- |
| **1. Très Facile** | Entrées/Sorties (I/O) & SQL | Disque dur, Réseau & RAM | 🟢 Moyen à Élevé | Très Faible |
| **2. Facile** | Logique de contrôle & Cache | Cycles CPU inutiles | 🟢 Très Élevé | Faible |
| **3. Modéré** | Algorithmique & Structures | Complexité temporelle $O(N)$ | 🟡 Élevé | Modéré |
| **4. Avancé** | Gestion Mémoire & Laravel | Allocation RAM & Pics CPU | 🟠 Élevé | Avancé |
| **5. Très Avancé** | Protocoles & Sérialisation | Parsing de chaînes & Paquets | 🟠 Moyen à Élevé | Difficile |
| **6. Complexe** | Concurrence & Architecture | Threads bloqués (I/O Wait) | 🔴 Très Élevé | Très Difficile |
| **7. Expert** | Architecture & Matériel | Instructions CPU natives | 🔴 Extrême | Extrême |

---

## 🛠️ Optimisations Spécifiques PHP & Laravel

### 1. Commandes de mise en cache (Niveau Très Facile)
Bannissez le chargement et le parsing dynamique des fichiers de configuration, de routes et de vues à chaque requête HTTP.
```bash
# Compilation globale pour la production
php artisan optimize

# Optimisation de l'autoloader Composer
composer install --optimize-autoloader --no-dev
```

### 2. Dompter l'ORM Eloquent (Niveau Modéré)

#### Résolution du problème des requêtes N+1 via le Eager Loading
*   **Énergivore :** `$posts = Post::all(); foreach($posts as $p) { echo $p->author->name; }` (Génère 101 requêtes pour 100 articles).
*   **Économe :** `$posts = Post::with('author')->get();` (Génère exactement 2 requêtes SQL au total).

#### Traitement de gros volumes de données sans saturer la RAM
*   **Énergivore :** `User::all();` (Charge des milliers d'objets en mémoire d'un coup, provoquant d'immenses surcharges pour le Garbage Collector).
*   **Économe :** `User::lazy()->each(function ($user) { ... });` (Utilise un curseur SQL transparent pour maintenir l'empreinte mémoire minimale).

#### Éviter les colonnes inutiles
*   **Économe :** `User::select('id', 'email')->where('active', true)->get();` (Réduit la taille de l'objet Eloquent instancié en mémoire).

---

## 📦 Implémentation Native de la Compression dans Laravel

### 1. Middleware de Compression HTTP (Gzip à la volée)
Ce middleware intercepte la réponse HTML ou JSON pour la compresser avant l'envoi, réduisant drastiquement les transferts réseau (I/O).

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class CompressResponse
{
    public function handle(Request $request, Closure $next): Response
    {
        $response = $next($request);

        if (function_exists('gzencode') && in_array('gzip', $request->getEncodings())) {
            $content = $response->getContent();

            // Éco-conception : on évite de solliciter le CPU pour moins de 1 Ko
            if (strlen($content) > 1024) {
                $compressedContent = gzencode($content, 5); // Niveau 5 : équilibre CPU/Taille
                $response->setContent($compressedContent);

                $response->headers->set('Content-Encoding', 'gzip');
                $response->headers->set('Length', strlen($compressedContent));
            }
        }

        return $response;
    }
}
```

### 2. Compression à l'écriture en Base de Données (Eloquent Custom Cast)
Idéal pour compresser de gros payloads JSON ou des logs textuels non indexés dans des colonnes de type `BLOB` ou `BINARY`.

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class GzipJson implements CastsAttributes
{
    public function get(Model $model, string $key, mixed $value, array $attributes): mixed
    {
        if (empty($value)) return null;
        return json_decode(gzuncompress($value), true);
    }

    public function set(Model $model, string $key, mixed $value, array $attributes): mixed
    {
        if (empty($value)) return null;
        return gzcompress(json_encode($value), 6); // Compression standard de niveau 6
    }
}
```

*Utilisation dans le modèle Eloquent :*
```php
protected $casts = [
    'payload' => \App\Casts\GzipJson::class,
];
```

---

## 🚀 Le Saut Ultime : Architecture In-Memory (Laravel Octane)
Pour éliminer le coût CPU du cycle de démarrage classique de PHP-FPM (qui détruit et reconstruit l'application à chaque requête HTTP), passez à **Laravel Octane** avec Swoole ou RoadRunner.
*   **Principe :** L'application est démarrée une seule fois en mémoire RAM au boot du serveur. Les requêtes HTTP entrantes réutilisent l'instance déjà initialisée.
*   **Résultat :** Le coût énergétique d'initialisation du framework passe de 100% à ~0% par requête, divisant les temps de réponse par 3 ou 4.
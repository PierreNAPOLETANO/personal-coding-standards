# Guide Complet d'Éco-Conception Logicielle : Serveurs & Laravel

Ce document rassemble l'ensemble des techniques pour réduire la consommation d'énergie d'un serveur uniquement par le code, classées du plus facile au plus complexe à mettre en œuvre.

---

## 📊 Tableau de synthèse de l'éco-conception logicielle

| Niveau | Domaine d'action | Cible prioritaire | Gain énergétique |
| :--- | :--- | :--- | :--- |
| **1. Très Facile** | Entrées/Sorties (I/O) & SQL | Disque dur, Réseau & RAM | 🟢 Moyen à Élevé |
| **2. Facile** | Logique de contrôle & Cache | Cycles CPU inutiles | 🟢 Très Élevé |
| **3. Modéré** | Algorithmique & Structures | Complexité temporelle $O(N)$ | 🟡 Élevé |
| **4. Avancé** | Gestion Mémoire (GC) | Allocation RAM & Pics CPU | 🟠 Élevé |
| **5. Très Avancé** | Protocoles & Sérialisation | Parsing de chaînes & Paquets | 🟠 Moyen à Élevé |
| **6. Complexe** | Concurrence & Asynchronisme | Threads bloqués (I/O Wait) | 🔴 Très Élevé |
| **7. Expert** | Architecture & Matériel | Instructions CPU natives | 🔴 Extrême |

---

## 🚀 Optimisations Spécifiques à PHP & Laravel

### 1. Mise en cache de l'architecture (Très Facile)
Bannir l'analyse et la compilation dynamique des fichiers à chaque requête HTTP.
```bash
# Compilation des configurations, routes et vues
php artisan optimize

# Optimisation de l'autoloader Composer
composer install --optimize-autoloader --no-dev
```

### 2. Dompter l'ORM Eloquent (Modéré)

#### Éviter les requêtes N+1 (Eager Loading)
*   **Énergivore :**
    ```php
    $posts = Post::all();
    foreach ($posts as $post) {
        echo $post->author->name; // 1 requête SQL par tour de boucle
    }
    ```
*   **Économe :**
    ```php
    $posts = Post::with('author')->get(); // Seulement 2 requêtes au total
    foreach ($posts as $post) {
        echo $post->author->name;
    }
    ```

#### Chunking & Lazy Loading pour les gros volumes
*   **Énergivore :**
    ```php
    $users = User::all(); // Charge 50 000 objets en RAM en même temps
    ```
*   **Économe :**
    ```php
    User::lazy()->each(function ($user) {
        // Traite les utilisateurs par petits flux de mémoire stables
    });
    ```

### 3. Architecture "In-Memory" avec Laravel Octane (Complexe)
Passer de PHP-FPM (qui détruit et reconstruit l'application à chaque requête) à **Laravel Octane** (Swoole / RoadRunner). L'application reste en RAM, éliminant le coût CPU d'initialisation de Laravel de 100% des requêtes à 0%.

---

## 🗜️ Implémentation Native de la Compression dans Laravel

### 1. Compression des Réponses HTTP (Middleware)
Créer un middleware `CompressResponse` pour compresser dynamiquement les réponses lourdes en Gzip.

```php
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

            // Éco-conception : Éviter de compresser les réponses de moins de 1 Ko
            if (strlen($content) > 1024) {
                $compressedContent = gzencode($content, 5); // Niveau 5 équilibré
                $response->setContent($compressedContent);

                $response->headers->set('Content-Encoding', 'gzip');
                $response->headers->set('Length', strlen($compressedContent));
            }
        }

        return $response;
    }
}
```

### 2. Compression en Base de Données (Custom Cast Eloquent)
Compresser les lourds payloads textuels/JSON avant stockage en base de données. La colonne doit être de type `BLOB` ou `BINARY`.

```php
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
        return gzcompress(json_encode($value), 6);
    }
}
```

Application sur le modèle Eloquent :
```php
protected $casts = [
    'payload' => \App\Casts\GzipJson::class,
];
```

---
*Note d'éco-conception globale : Privilégier dès que possible la délégation de la compression HTTP à un serveur web amont comme NGINX (`gzip on;`) pour économiser les threads et les cycles CPU du moteur PHP.*

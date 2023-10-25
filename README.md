# Laravel Proxy

Laravel proxy trait to resolve the currently bound values of the given contract.

## Installation

You can install the package via composer:

```bash
composer require fer-projekt/laravel-proxy
```

## Usage

Easily swap container bindings without touching anything else in the codebase.

```php
namespace App\Models;

use Ecommerce\Models\Product as BaseProduct;

class Product extends BaseProduct
{
    //
}
```

After you created your customizable model, you can swap the container binding for the product contract. You might do that in one of your service provider's `register` method:

```php
use App\Models\Product;
use Ecommerce\Contracts\Product as Contract;

public function register()
{
    $this->app->bind(Contract::class, Product::class);
}
```

> Note, you should extend the **base model** then bind it to the **interface**.

## Proxies

Proxies are representing the classes that are currently bound to the container. You usually won't use these a lot, but in some cases – like when you develop a package or extension – proxies can be convenient.

```php
use Ecommerce\Models\Product;

// Available proxy methods
Product::proxy();
Product::getProxiedClass();
Product::getProxiedContract();

// Proxied calls to the bound instance
Product::proxy()->newQuery()->where(...);
Product::proxy()->variation(...);

// Dynamic usage of bound classes
public function product()
{
    $this->belongsTo(Product::getProxiedClass());
}
```

Laravel Proxy provides the `InteractsWithProxy` trait, that makes the desired class proxyable. The trait comes with an abstract methods to be able to resolve the proxies class from the container automatically:

```php
namespace App\Models;

use App\Contracts\Models\Addon as Contract;
use FerProjekt\LaravelProxy\InteractsWithProxy;
use Illuminate\Databse\Eloquent\Model;

class Addon extends Model implements Contract
{
    use InteractsWithProxy;

    /**
     * Get the proxied interface.
     *
     * @return string
     */
    public static function getProxiedInterface(): string
    {
        return Contract::class;
    }
}
```

Also, we need to bind the contract to the container in a service provider:

```php
public function register(): void
{
    $this->app->bind(\App\Contracts\Models\Addon::class, \App\Models\Addon::class);
}
```

From this point, our `Addon` model is proxyable, which means if we swap the binding in the container, the proxied class will be the currently bound value and not the original one. This adds more flexibility when extending our application or using packages.

```php
// Use the Addon proxy

Addon::proxy()->newQuery();
```

## Development

Clone the package from the github:

```bash
git clone https://github.com/fer-projekt/laravel-proxy.git

cd laravel-proxy
```

Starting & stopping docker

```bash
docker-compose up -d
```

```bash
docker-compose down
```

Install dependencies via composer and testing:

```bash
docker-compose exec app bash
```

```bash
composer update
```

```bash
phpunit
```

## Credits

- [conedevelopment](https://github.com/conedevelopment)

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.

# EasyCo Core Foundation

## Status

**Phase 1 — Foundation**

The initial EasyCo Core package has been implemented and verified against a Bagisto application.

## Goal

The purpose of this milestone is to establish the mechanism by which EasyCo functionality is attached to Bagisto without modifying Bagisto core files.

The implementation uses standard Laravel package mechanisms.

## Architecture

```text
Bagisto Application
        │
        ▼
Composer Path Repository
        │
        ▼
EasyCo Package
        │
        ▼
Laravel Package Discovery
        │
        ▼
Service Provider
        │
        ├── Configuration
        ├── Routes
        ├── Views
        ├── Migrations
        └── Other module services
```

## Verified Extension Mechanism

A temporary `EasyCo/System` package was created as a proof of concept.

The package successfully demonstrated:

1. Local Composer path repository discovery.
2. Installation as a Composer dependency.
3. Laravel package discovery.
4. Automatic Service Provider registration.
5. Route loading through `loadRoutesFrom()`.
6. Package configuration through `mergeConfigFrom()`.

The temporary System package was removed after the mechanism was verified.

## EasyCo Core

The first permanent EasyCo package is:

```text
packages/EasyCo/Core
```

Current structure:

```text
Core/
├── composer.json
├── config/
│   └── core.php
└── src/
    └── Providers/
        └── CoreServiceProvider.php
```

### Composer Package

The package declares:

```json
{
    "name": "easyco/core",
    "type": "library"
}
```

and exposes the namespace:

```text
EasyCo\Core\
```

through PSR-4 autoloading.

The Laravel Service Provider is registered through Composer package discovery.

### Service Provider

The Core Service Provider currently provides the package configuration:

```php
$this->mergeConfigFrom(
    __DIR__.'/../../config/core.php',
    'easyco.core'
);
```

The configuration is accessible through Laravel's normal configuration system:

```php
config('easyco.core.name');
```

## Architectural Constraint

EasyCo must not modify Bagisto core files.

The following remain outside the EasyCo extension layer:

```text
vendor/
packages/Webkul/
```

EasyCo functionality belongs under:

```text
packages/EasyCo/
```

where practical.

## Why Composer Package Discovery

Using Composer package discovery allows EasyCo modules to remain self-contained.

A module can provide its own:

* Service Provider
* configuration
* routes
* views
* migrations
* commands
* events and listeners
* translations

without requiring manual registration in Bagisto's application bootstrap.

This provides a clean separation between the EasyCo layer and the underlying commerce engine.

## Next Step

The next Core milestone is to establish the shared EasyCo foundation used by other modules.

This should be kept intentionally small.

Potential responsibilities include:

* shared contracts
* common events
* module configuration conventions
* shared services
* common extension infrastructure

Specific functionality should remain in dedicated EasyCo modules rather than being accumulated in Core.

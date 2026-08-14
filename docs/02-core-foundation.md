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

The Core Service Provider currently has two responsibilities: package configuration, and dynamic registration of EasyCo submodules.

Configuration:

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

### Dynamic Submodule Registration

`registerSubModules()` is the permanent replacement for the old manual `EasyCo/System` proof-of-concept. For each known submodule (currently just `Admin`), Core checks whether `packages/EasyCo/{Name}/src` exists, and if so:

1. Registers the submodule's PSR-4 namespace at runtime via the Composer autoloader (`$loader->addPsr4(...)`).
2. Registers the submodule's Service Provider with the Laravel container (`$this->app->register($providerClass)`).

Adding a new submodule currently requires adding one entry to a hardcoded map inside `CoreServiceProvider` — no changes to root `composer.json` or Bagisto files. This map should eventually be replaced with real filesystem discovery once more than a couple of modules exist.

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

## First Submodule: EasyCo/Admin

`EasyCo/Admin` is the first module registered through the mechanism above. It currently provides:

* runtime menu injection into Bagisto's admin sidebar (`menu.admin`);
* in-memory Bulgarian/English translation merging for both the admin and storefront namespaces;
* a single verification route (`admin/easyco-test`).

See `docs/06-admin.md` for the full description of this module and its planned UX work. This module's real functionality (product editor, Simple Mode config) has not started yet — the current route is a smoke test only.

## Next Step

Core's foundation and dynamic submodule registration are verified. Shared Core contracts/services should only be introduced once a second module creates a genuine, concrete need for them — not speculatively.

Specific functionality should remain in dedicated EasyCo modules rather than being accumulated in Core.

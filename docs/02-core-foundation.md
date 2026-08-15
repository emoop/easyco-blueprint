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

Adding a new submodule currently requires adding one entry to a hardcoded map inside `CoreServiceProvider` — no changes to root `composer.json` or Bagisto files.

This map should eventually be replaced with real filesystem discovery once more than a couple of modules exist.

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

`EasyCo/Admin` is the first module registered through the mechanism above.

It currently provides:

* runtime menu injection into Bagisto's admin sidebar (`menu.admin`);
* a custom Bulgarian translation loader for admin and storefront namespaces;
* EasyCo Bulgarian translation overlays for the admin and storefront;
* a single verification route (`admin/easyco-test`).

See `docs/06-admin.md` for the full description of this module and its planned UX work.

This module's real functionality (product editor, Simple Mode config) has not started yet — the current route is a smoke test only.

## Translation Architecture

EasyCo provides its own Bulgarian translation layer without modifying Bagisto translation files.

The translation mechanism is implemented inside `EasyCo/Admin`, because the module currently owns the EasyCo Bulgarian translations for the admin and storefront.

The module provides:

```text
EasyCo\Admin\Translation\MergingTranslationLoader
```

This loader decorates Laravel's existing `translation.loader` service through the Laravel service container.

The architecture is:

```text
Laravel Translation System
          │
          ▼
MergingTranslationLoader
          │
          ├── Original Laravel / Bagisto Loader
          │
          └── EasyCo Bulgarian Translation Files
```

For Bulgarian (`bg`), the loader combines the available translation sources in memory.

The intended priority is:

```text
English fallback
       ↓
Bagisto Bulgarian
       ↓
EasyCo Bulgarian
```

This means:

1. English provides the base fallback.
2. Existing Bagisto Bulgarian translations override the English values.
3. EasyCo Bulgarian translations override Bagisto values where EasyCo provides its own translation.

The merge is performed using Laravel/PHP translation arrays with `array_replace_recursive()`.

EasyCo does not need to maintain a complete copy of Bagisto's Bulgarian translation files. EasyCo can provide only the translations it owns or intentionally overrides.

When an EasyCo translation key is not present, the existing Bagisto translation remains available. If no Bulgarian translation exists, the English fallback remains available instead of exposing a raw translation key.

Current EasyCo Bulgarian translation files are:

```text
packages/EasyCo/Admin/src/Resources/lang/bg/
├── app.php
└── shop.php
```

The loader is registered by `AdminServiceProvider` using Laravel's container extension mechanism.

Conceptually:

```php
$this->app->extend('translation.loader', function ($activeLoader, $app) {
    return new MergingTranslationLoader(
        $activeLoader,
        realpath(__DIR__ . '/../Resources/lang')
    );
});
```

The custom loader delegates namespace registration, JSON translation paths, and namespace discovery to the underlying Laravel loader.

No Bagisto translation files are modified on disk.

This keeps the translation layer isolated inside EasyCo and preserves the upgrade path for the underlying Bagisto installation.

### Translation Ownership

Translation files belong to the EasyCo module that provides the corresponding functionality.

Currently:

```text
EasyCo/Admin
    └── Admin and storefront Bulgarian translations
```

Translation infrastructure should not be moved into `EasyCo/Core` merely because Core exists.

A shared translation service should only be introduced if another EasyCo module creates a concrete requirement for the same infrastructure.

This follows the broader EasyCo principle of keeping Core small and avoiding speculative abstractions.

## Next Step

Core's foundation and dynamic submodule registration are verified.

`EasyCo/Admin` is also verified as the first real EasyCo module, including menu injection, translation loading, and route registration.

The next implementation phase should focus on turning `EasyCo/Admin` from a proof of concept into real merchant-facing functionality.

Shared Core contracts and services should only be introduced once a second module creates a genuine, concrete need for them — not speculatively.

Specific functionality should remain in dedicated EasyCo modules rather than being accumulated in Core.


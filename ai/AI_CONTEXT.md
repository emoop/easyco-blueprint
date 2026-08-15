# EasyCo AI Context

This document provides the current technical context for AI-assisted development of EasyCo.

It is intended to prevent AI development sessions from making assumptions based on outdated conversations or implementing functionality that has already been completed.

The repository state and the EasyCo Blueprint are authoritative.

---

## 1. Project

**EasyCo (Леснотийка)** is an open-source e-commerce platform for small and medium-sized businesses.

The primary goal is to provide a platform that is:

* simple to operate
* fast
* secure
* extensible
* easy to deploy
* suitable for small and medium-sized stores

Core philosophy:

> **Powerful under the hood. Simple in the hands of the merchant.**

EasyCo aims to hide technical complexity from merchants while retaining the flexibility required by a modern e-commerce platform.

---

## 2. Repository Structure

The project is split between two repositories.

### EasyCo

Main source repository:

```text
https://github.com/emoop/easyco
```

This repository contains the actual EasyCo source code and packages.

### EasyCo Blueprint

Architecture and development documentation:

```text
https://github.com/emoop/easyco-blueprint
```

The Blueprint contains:

* architecture
* technical decisions
* RFCs
* roadmap
* development phases
* AI development context
* implementation guidance

When implementation and documentation disagree, inspect the current repository and Blueprint before making assumptions.

---

## 3. Current Foundation

EasyCo v1 is initially built on:

* Laravel
* Bagisto

Bagisto is the initial **commerce engine**.

Bagisto is not the identity of EasyCo.

EasyCo should provide an independent platform and extension layer above the commerce engine.

The intended architecture is:

```text
EasyCo
   ↓
EasyCo Modules / Extension Layer
   ↓
Bagisto
   ↓
Laravel
```

EasyCo must avoid unnecessary coupling to Bagisto internals.

---

## 4. Bagisto Integration Principle

EasyCo must not modify Bagisto core unless there is an exceptional and explicitly justified architectural reason.

Preferred mechanisms include:

* Laravel packages
* Service Providers
* Composer package discovery
* dependency injection
* events
* listeners
* middleware
* configuration
* routes
* views
* Blade components
* APIs
* migrations
* adapters
* contracts
* extension points provided by Bagisto

Avoid direct modifications to:

```text
packages/Webkul/
vendor/
```

unless a specific architectural decision explicitly permits it.

The goal is to preserve a clean upgrade path for Bagisto.

---

## 5. Current Development Environment

The current development application is a Bagisto installation running locally.

Current project path:

```text
C:\laragon\www\mshop
```

Local application URL:

```text
http://mshop.test/
```

The project uses Composer path repositories for local EasyCo packages.

Current root Composer configuration contains:

```json
"repositories": [
    {
        "type": "path",
        "url": "packages/*/*",
        "options": {
            "symlink": true
        }
    }
]
```

This allows EasyCo packages to be developed directly inside the Bagisto application while remaining independent Composer packages.

---

## 6. Current Laravel / Bagisto Versions

The current Bagisto application uses:

* Laravel 12.x
* PHP 8.3.x
* Bagisto 2.x

Do not assume versions from older EasyCo discussions.

Always inspect the current `composer.json` when version-specific implementation decisions are required.

---

# 7. Current Development Phase

## Phase 1 — Foundation

The project is currently in the initial **EasyCo Core / Foundation** phase.

The initial package integration mechanism has already been implemented and verified.

The following have been completed:

* EasyCo package directory structure
* Composer path repository integration
* EasyCo Core Composer package
* Laravel package discovery
* automatic Core Service Provider registration
* Core configuration
* verification against a running Bagisto application
* dynamic submodule registration mechanism in Core (`registerSubModules()`)
* first business module: `EasyCo/Admin` (menu injection, Bulgarian translation loader, translation merging, initial route)

The temporary `EasyCo/System` package was created only as a proof of concept and has been removed after the extension mechanism was verified. Its role has effectively been superseded by Core's `registerSubModules()` mechanism, which is now the permanent way EasyCo submodules attach themselves.

---

# 8. Verified EasyCo Extension Mechanism

A temporary package named:

```text
EasyCo/System
```

was created to verify how EasyCo packages can attach to Bagisto.

The package successfully demonstrated:

1. Composer path repository discovery.
2. Installation as a Composer dependency.
3. Laravel package discovery.
4. Automatic Service Provider registration.
5. Route loading using `loadRoutesFrom()`.
6. Package configuration using `mergeConfigFrom()`.

The temporary package was then removed.

This means the mechanism is no longer theoretical.

It has been tested against the real Bagisto application.

---

# 9. EasyCo Core

The first permanent EasyCo package is:

```text
packages/EasyCo/Core
```

Current structure:

```text
packages/EasyCo/Core/
├── composer.json
├── config/
│   └── core.php
└── src/
    └── Providers/
        └── CoreServiceProvider.php
```

---

## 9.1 Core Composer Package

Current package name:

```text
easyco/core
```

Package type:

```text
library
```

PSR-4 namespace:

```text
EasyCo\Core\
```

The package declares its Laravel Service Provider through Composer package discovery.

Current provider:

```text
EasyCo\Core\Providers\CoreServiceProvider
```

The package is installed locally through the application's Composer path repository.

---

## 9.2 Core Service Provider

Current provider:

```text
packages/EasyCo/Core/src/Providers/CoreServiceProvider.php
```

Its current responsibilities are:

1. Package configuration registration.
2. Dynamic discovery and registration of EasyCo submodules.

Configuration is loaded using:

```php
$this->mergeConfigFrom(
    __DIR__.'/../../config/core.php',
    'easyco.core'
);
```

The provider is discovered automatically by Laravel.

It does not need to be manually added to Bagisto's application provider list.

---

## 9.2.1 Dynamic Submodule Registration

`CoreServiceProvider::registerSubModules()` is the current, permanent replacement for the old manual `EasyCo/System` proof-of-concept.

It works as follows:

1. A static map of known submodule names to their Service Provider class is declared inside `CoreServiceProvider` (currently only `Admin => EasyCo\Admin\Providers\AdminServiceProvider`).
2. For each entry, Core checks whether `packages/EasyCo/{Name}/src` exists on disk.
3. If it exists, Core registers the submodule's PSR-4 namespace at runtime using the Composer autoloader instance (`$loader->addPsr4(...)`).
4. If the submodule's Service Provider class is available, Core registers it directly with the Laravel container (`$this->app->register($providerClass)`).

This means a new EasyCo submodule can currently be picked up by adding its folder under `packages/EasyCo/` and adding one line to the `$modules` map in `CoreServiceProvider` — no changes to root `composer.json` or Bagisto files are required.

Known limitation: the module map is currently a hardcoded array inside Core, not automatic filesystem discovery. This is acceptable for the current number of modules but should be revisited (e.g. filesystem scan or a manifest file) if the module count grows.

---

## 9.3 Core Configuration

Current configuration file:

```text
packages/EasyCo/Core/config/core.php
```

The configuration is available through Laravel's standard configuration system.

Example:

```php
config('easyco.core.name');
```

The configuration was successfully tested through Laravel Tinker.

Current initial configuration contains:

```php
return [
    'enabled' => true,

    'name' => 'EasyCo',

    'version' => '1.0.0',
];
```

This is currently foundation-level configuration only.

Do not start adding unrelated functionality to Core simply because Core exists.

---

## 9.4 EasyCo Admin Module (first business module)

The first EasyCo business module has been implemented and verified:

```text
packages/EasyCo/Admin
```

Current structure:

```text
packages/EasyCo/Admin/
├── composer.json
└── src/
    ├── Config/
    │   └── menu.php
    ├── Providers/
    │   └── AdminServiceProvider.php
    ├── Resources/
    │   └── lang/
    │       └── bg/
    │           ├── app.php
    │           └── shop.php
    ├── Routes/
    │   └── web.php
    └── Translation/
        └── MergingTranslationLoader.php
```

### Composer package

```text
easyco/admin
```

PSR-4 namespace:

```text
EasyCo\Admin\
```

Registered via Core's `registerSubModules()` mechanism (see 9.2.1), not through the root `composer.json` provider list.

### 9.4.1 Runtime Menu Injection

`AdminServiceProvider::register()` merges `Config/menu.php` into Bagisto's `menu.admin` configuration key using `mergeConfigFrom()`.

Currently registered menu entries:

* `easyco` — top-level "EasyCo Panel" sidebar item.
* `catalog.easyco_inventory` — submenu injected under Bagisto's existing "Catalog" menu via dot-notation.

No Bagisto menu configuration files are modified directly.

### 9.4.2 Bulgarian Localization (Admin + Storefront)

`EasyCo/Admin` provides a custom translation layer through:

```text
EasyCo\Admin\Translation\MergingTranslationLoader
```

`AdminServiceProvider` decorates Laravel's `translation.loader` service using the Laravel service container's `extend()` mechanism.

For Bulgarian (`bg`), the custom loader:

1. Loads the translation data through Laravel's original translation loader.
2. Loads the corresponding English translation as a fallback.
3. Loads the EasyCo Bulgarian translation file when one exists.
4. Merges the available translation sources in memory using `array_replace_recursive()`.

The intended priority is:

```text
English fallback
       ↓
Bagisto Bulgarian
       ↓
EasyCo Bulgarian
```

This means EasyCo translations override Bagisto translations where EasyCo provides a value, while keys not provided by EasyCo remain available from Bagisto or English fallback.

Current EasyCo translation files are:

```text
packages/EasyCo/Admin/src/Resources/lang/bg/
├── app.php
└── shop.php
```

The loader also delegates Laravel's namespace registration, JSON translation path registration, and namespace discovery methods to the underlying translation loader.

No Bagisto translation files are modified on disk.

The translation layer is intentionally implemented inside `EasyCo/Admin`, because that module currently owns the EasyCo admin and storefront Bulgarian translations.

It should only move into Core if a concrete second module requires the same shared translation infrastructure.

### 9.4.3 Initial Route

`Routes/web.php` currently registers one route as a manual verification point:

```text
GET admin/easyco-test  →  name: admin.easyco.index
```

This route is a smoke test confirming the module boots and its routes load; it is not yet real admin functionality. Both menu entries in 9.4.1 currently point at this test route.

### 9.4.4 Not Yet Implemented

Per `docs/06-admin.md`, the following planned Admin UX work has **not** started yet:

* WooCommerce-style unified product editor (single-page product creation, replacing Bagisto's multi-tab/reload flow).
* Simplified Config "Simple Mode" (collapsing Bagisto's advanced settings behind an accordion, exposing only Store Name, Contact Email, Currency, Tax Rules by default).

---

# 10. Important Architectural Decision

EasyCo modules should be independent Laravel packages whenever practical.

A typical EasyCo module should be structured similarly to:

```text
packages/EasyCo/<Module>/
├── composer.json
├── config/
├── routes/
├── resources/
├── database/
└── src/
    └── Providers/
```

The exact structure may vary according to the module's requirements.

Each module should own its own:

* Service Provider
* configuration
* routes
* views
* migrations
* translations
* commands
* services
* events
* listeners

Core should not become a container for unrelated business functionality.

---

# 11. Core Responsibility

EasyCo Core should remain deliberately small.

Core may eventually provide shared infrastructure such as:

* common contracts
* shared services
* module conventions
* common events
* extension infrastructure
* common configuration conventions
* shared utilities that genuinely belong to the platform foundation

Business-specific functionality should live in dedicated modules.

Examples:

```text
EasyCo/Shipping
EasyCo/Marketing
EasyCo/POS
EasyCo/Analytics
EasyCo/Notifications
```

should not automatically become part of:

```text
EasyCo/Core
```

Avoid creating abstractions until there is a real need for them.

---

# 12. Planned EasyCo Areas

These are project goals, not necessarily implemented functionality.

Always inspect the repository before claiming that a feature exists.

Planned areas include:

* Product management
* Product variations
* Media and catalog management
* Shipping zones
* Shipping rules
* Speedy integration
* Econt integration
* BoxNow integration
* Marketing
* Meta Pixel
* Meta Conversions API
* POS
* Online/offline inventory synchronization
* Analytics
* Business reporting
* Security
* Anti-bot protection
* Performance
* Caching
* Storefront customization
* Discounts
* Promotional tools
* Abandoned carts
* Basic email marketing
* Push notifications
* AI-friendly catalog APIs
* AI-friendly platform APIs
* Simple deployment
* Server installation and configuration

These should be implemented incrementally.

---

# 13. Development Philosophy

EasyCo development should follow:

```text
Problem
   ↓
Architecture / RFC
   ↓
Implementation
   ↓
Testing
   ↓
Documentation
   ↓
Commit
```

Prefer small working milestones over large speculative implementations.

Do not implement large amounts of functionality before the underlying architecture has been verified.

Do not over-engineer.

Do not create abstractions merely because they might be useful someday.

---

# 14. AI Development Rules

AI-assisted development must follow these rules.

### Rule 1 — Inspect before changing

Before implementing anything substantial:

1. Inspect the current repository.
2. Read the relevant Blueprint documentation.
3. Read relevant RFCs.
4. Read relevant AI context.
5. Inspect the actual project files involved.
6. Determine whether the requested functionality already exists.

Never recreate functionality that already exists.

---

### Rule 2 — Repository is authoritative

Previous conversations are not authoritative.

The current repository is authoritative.

If an older conversation says something different from the current code or Blueprint:

1. inspect the current state
2. identify the discrepancy
3. explain it
4. follow the current repository unless a new decision is explicitly made

Do not silently rely on historical assumptions.

---

### Rule 3 — Keep scope focused

When working on a milestone:

* work only on the current objective
* avoid unrelated refactoring
* avoid speculative features
* avoid unnecessary dependencies
* avoid premature abstractions

---

### Rule 4 — Do not modify Bagisto core casually

Before changing a Bagisto file, determine whether the same result can be achieved using:

* a package
* Service Provider
* event
* listener
* middleware
* configuration
* dependency injection
* route registration
* view override
* Blade component
* API
* adapter
* contract
* another supported extension mechanism

Direct modification of Bagisto should be treated as an exception.

---

### Rule 5 — Verify with the real application

Architecture should be validated against the actual Bagisto application rather than assumed from documentation alone.

When testing package integration, prefer small concrete tests.

Examples:

* package discovery
* Service Provider loading
* configuration loading
* route loading
* event registration
* translation loading and fallback behavior

---

# 15. Current Verified Tests

The following tests have already succeeded.

## Package Discovery

Composer successfully installed:

```text
easyco/core
```

and Laravel package discovery reported:

```text
easyco/core ................................................................ DONE
```

---

## Service Provider

`CoreServiceProvider` was successfully discovered and executed automatically.

This was verified by temporarily executing code from the provider during application boot.

---

## Configuration

The following was successfully tested:

```php
config('easyco.core.name');
```

Result:

```text
EasyCo
```

The Core configuration therefore successfully loads through Laravel's package configuration mechanism.

---

## Admin Translation Loader

The custom translation loader has been implemented as:

```text
packages/EasyCo/Admin/src/Translation/MergingTranslationLoader.php
```

It decorates Laravel's existing `translation.loader` service through `AdminServiceProvider`.

The loader was verified against the running Bagisto application.

Its purpose is to provide EasyCo Bulgarian translations without modifying Bagisto translation files.

For Bulgarian translations, the effective priority is:

```text
English fallback
       ↓
Bagisto Bulgarian
       ↓
EasyCo Bulgarian
```

The EasyCo translation layer currently covers the admin and storefront namespaces through:

```text
packages/EasyCo/Admin/src/Resources/lang/bg/app.php
packages/EasyCo/Admin/src/Resources/lang/bg/shop.php
```

The loader performs the merge in memory.

---

# 16. Temporary System Package

A temporary package was previously created:

```text
packages/EasyCo/System
```

Its purpose was solely to prove the package extension mechanism.

It demonstrated:

```text
Composer
   ↓
Package Discovery
   ↓
Service Provider
   ↓
Routes
   ↓
Configuration
```

The package has now been removed from the application.

Do not recreate `EasyCo/System` unless a new, explicit architectural reason exists.

---

# 17. Current Next Step

Core's foundation (configuration + dynamic submodule registration) and the first business module (`EasyCo/Admin` — menu injection, translation loader, Bulgarian translation merging, test route) are both verified.

The immediate next milestone is to turn `EasyCo/Admin` from a proof-of-concept into real merchant-facing functionality.

Two candidate directions from `docs/06-admin.md` (pick one, do not start both at once):

1. **WooCommerce-style product editor** — a single-page product creation/edit flow (title, description, gallery, and Inventory/Shipping/Variations as accordion cards) replacing Bagisto's multi-tab reload-heavy flow.
2. **Simplified Config "Simple Mode"** — collapse Bagisto's advanced settings behind an accordion, exposing only Store Name, Contact Email, Currency, Tax Rules by default.

RFC 0001 (EasyCo Extension Architecture) was formally **Accepted on 2026-08-14**; the codebase already conformed to it.

Also still open, lower priority than the above:

1. Define shared Core contracts only where a real second module creates the need.
2. Add automated tests for Core bootstrapping and submodule registration.
3. Add automated tests for Admin's translation-merge logic.
4. Replace the hardcoded submodule map in `CoreServiceProvider` with real discovery if/when more than a couple of modules exist.

The next implementation should remain small and testable, and should replace the `admin/easyco-test` route rather than accumulate alongside it.

---

# 18. Documentation Requirements

Significant architectural decisions should be documented in the EasyCo Blueprint.

Relevant documentation locations:

```text
docs/
rfcs/
ai/
```

The AI context should be updated whenever a significant implementation milestone changes the current state.

Documentation should describe:

* what has actually been implemented
* what has been verified
* what remains unfinished
* important architectural constraints
* the next intended milestone

Do not document planned features as completed features.

---

# 19. Git and Release Discipline

Prefer small meaningful commits.

Examples:

```text
feat: add initial EasyCo Core package
```

```text
test: verify Core package discovery
```

```text
docs: document EasyCo Core foundation milestone
```

Avoid mixing unrelated features and documentation into large commits when a cleaner history is practical.

Do not publish packages or create releases before the package is sufficiently stable.

---

# 20. Security

Security is a core architectural concern.

Do not introduce:

* unnecessary dependencies
* insecure defaults
* arbitrary code execution mechanisms
* unsafe file handling
* unvalidated input processing
* unnecessary access to Bagisto internals

Composer security advisories must be investigated when they affect production dependencies.

The current development application has reported Composer security advisories.

These have not yet been resolved as part of the Core milestone.

Do not silently upgrade or downgrade dependencies merely to eliminate the warning without first identifying the affected packages and compatibility implications.

---

# 21. Performance

EasyCo is intended to be fast.

Performance decisions should consider:

* database queries
* caching
* HTTP response time
* frontend payload
* asset loading
* background processing
* queue usage
* image handling
* API efficiency

Do not optimize prematurely.

Measure before introducing complicated performance infrastructure.

---

# 22. Merchant Experience

Technical complexity should remain hidden from the merchant wherever possible.

EasyCo should prefer:

```text
Complex technical system
        ↓
Simple merchant interface
```

rather than exposing unnecessary configuration and infrastructure concepts to store owners.

Power belongs underneath the interface.

Simplicity belongs at the merchant level.

---

# 23. Overall Architectural Direction

The intended long-term architecture is:

```text
                    EasyCo
                      │
        ┌─────────────┼─────────────┐
        │             │             │
      Core        Business       Integrations
        │          Modules          │
        │             │             │
        └─────────────┼─────────────┘
                      │
                 Extension Layer
                      │
                    Bagisto
                      │
                    Laravel
```

EasyCo should use Bagisto as a proven commerce foundation while gradually providing a simpler and more cohesive platform experience.

Bagisto should remain replaceable at the architectural boundary wherever practical.

---

# 24. Current State Summary

At the current point in development:

### Completed

* EasyCo project direction defined.
* Bagisto selected as the initial commerce engine.
* Extension-first architecture established.
* Composer path repository verified.
* Temporary System package used to prove the extension mechanism.
* Temporary System package removed.
* EasyCo Core package created.
* Core Composer package installed.
* Laravel package discovery verified.
* Core Service Provider verified.
* Core configuration verified.
* Dynamic submodule registration mechanism (`registerSubModules()`) implemented in Core, replacing the old manual System-package approach.
* First EasyCo business module, `EasyCo/Admin`, created and verified.
* Admin: runtime menu injection into Bagisto's admin sidebar (`menu.admin`).
* Admin: custom Laravel translation loader implemented through `MergingTranslationLoader`.
* Admin: Bulgarian translation overlay for both admin and storefront namespaces.
* Admin: English fallback and Bagisto Bulgarian translations are preserved through in-memory merging.
* Admin: test route confirming module routes load correctly.
* RFC 0001 (Extension Architecture) formally Accepted.
* No Bagisto core modifications required so far.

### Not yet completed

* Core automated tests
* Admin translation-merge automated tests
* Shared Core contracts
* Shared Core services
* Module conventions documented
* Real Admin functionality behind the test route (WooCommerce-style product editor, Simple Mode config)
* Filesystem-based (non-hardcoded) submodule discovery in Core
* Shipping integrations
* Marketing integrations
* POS
* Analytics
* Storefront features
* Deployment tooling
* Production release process

---

# 25. AI Instruction

When continuing EasyCo development:

**Do not start by implementing features.**

First determine:

```text
What exists?
What is documented?
What was already verified?
What is the current milestone?
What is the smallest correct next step?
```

Then implement only that step.

The objective is not to build everything quickly.

The objective is to build a clean, maintainable and extensible platform without accumulating unnecessary architectural debt.

> **Powerful under the hood. Simple in the hands of the merchant.**


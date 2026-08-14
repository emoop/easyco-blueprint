# RFC 0001: EasyCo Extension Architecture

**Status:** Accepted
**Date:** 2026-08-11
**Accepted:** 2026-08-14
**Milestone:** 0.2
**Area:** Architecture

---

## 1. Summary

This RFC defines the extension architecture for EasyCo.

EasyCo will be built on top of Bagisto and Laravel while keeping EasyCo functionality separate from the Bagisto core.

The primary objective is:

> **Extend the commerce engine without modifying its core.**

This should allow EasyCo to evolve independently while maintaining the ability to upgrade Bagisto with minimal disruption.

---

# 2. Problem

EasyCo needs to add substantial functionality that may not exist in the underlying Bagisto installation.

Examples include:

* improved administration UX;
* advanced shipping;
* Bulgarian courier integrations;
* Meta Pixel and Conversions API;
* POS;
* security;
* analytics;
* merchant automation;
* AI integrations;
* storefront customization.

A naive implementation could modify Bagisto directly.

For example:

```text
Bagisto core
    ↓
custom modifications
    ↓
EasyCo functionality
```

This creates a serious long-term problem.

When Bagisto is upgraded:

```text
New Bagisto version
        ↓
Existing custom modifications
        ↓
Conflicts
        ↓
Manual migration
```

This creates the same class of problem experienced by mature extension ecosystems when extensions depend heavily on internal implementation details.

EasyCo must avoid this wherever practical.

---

# 3. Goals

The EasyCo extension architecture should provide:

1. Minimal modification of Bagisto core.
2. Clear separation between EasyCo and Bagisto.
3. Installable and removable EasyCo modules.
4. Composer-based dependency management.
5. Explicit module dependencies.
6. Version compatibility declarations.
7. Laravel-native development patterns.
8. Easy testing.
9. Upgrade-friendly architecture.
10. Clear ownership of EasyCo database migrations.
11. Ability to disable optional functionality.
12. Ability to replace individual implementations.
13. Clear APIs between EasyCo modules.
14. Predictable extension lifecycle.

---

# 4. Non-Goals

This RFC does not attempt to:

* replace Bagisto's commerce engine;
* create a second ORM;
* duplicate Bagisto's product/order/customer models unnecessarily;
* introduce a generic microservice architecture;
* abstract every Bagisto API behind an artificial interface;
* support multiple commerce engines immediately;
* solve every future extensibility problem.

The architecture should remain pragmatic.

---

# 5. Core Principle

The fundamental rule is:

> **EasyCo owns EasyCo functionality. Bagisto owns Bagisto functionality.**

For example:

```text
Bagisto
├── Products
├── Orders
├── Customers
├── Inventory
├── Channels
└── Core Commerce

EasyCo
├── Shipping
├── POS
├── Marketing
├── Security
├── Analytics
├── AI
└── Merchant UX
```

When EasyCo needs information owned by Bagisto, it should consume that information through supported integration points rather than copying the underlying domain unnecessarily.

---

# 6. Module Architecture

An EasyCo module should be a Laravel package.

Conceptually:

```text
packages/
└── easyco/
    ├── shipping/
    ├── marketing/
    ├── pos/
    ├── security/
    ├── analytics/
    └── ai/
```

Each package should have its own:

```text
src/
config/
database/
resources/
routes/
tests/
```

where required.

A typical module may look like:

```text
packages/easyco/shipping/

├── composer.json
├── README.md
│
├── src/
│   ├── Providers/
│   │   └── ShippingServiceProvider.php
│   ├── Models/
│   ├── Services/
│   ├── Actions/
│   ├── Events/
│   ├── Listeners/
│   └── Support/
│
├── config/
│   └── shipping.php
│
├── database/
│   └── migrations/
│
├── resources/
│   └── views/
│
├── routes/
│   └── web.php
│
└── tests/
```

Not every module must contain every directory.

Modules should remain as small as practical.

---

# 7. Composer

Composer should be the primary dependency mechanism.

An EasyCo package should declare its dependencies explicitly.

Conceptually:

```json
{
    "name": "easyco/shipping",
    "type": "library",
    "require": {
        "php": "^8.3",
        "easyco/core": "^1.0"
    }
}
```

If a module depends on a specific Bagisto capability, that dependency should be declared where practical.

Version requirements should be explicit rather than relying on undocumented compatibility.

---

# 8. EasyCo Core

A small `easyco/core` package may eventually provide shared infrastructure.

Potential responsibilities:

* module registration;
* configuration conventions;
* shared contracts;
* common events;
* permissions;
* settings;
* feature flags;
* module metadata;
* common helpers.

However:

> `easyco/core` must not become a dumping ground.

Functionality belongs in its appropriate module.

---

# 9. Service Providers

Laravel Service Providers are the primary module bootstrap mechanism.

A module provider may register:

* configuration;
* routes;
* migrations;
* translations;
* views;
* events;
* bindings;
* commands.

Conceptually:

```php
class ShippingServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        $this->mergeConfigFrom(
            __DIR__.'/../../config/shipping.php',
            'easyco.shipping'
        );
    }

    public function boot(): void
    {
        $this->loadMigrationsFrom(
            __DIR__.'/../../database/migrations'
        );

        $this->loadViewsFrom(
            __DIR__.'/../../resources/views',
            'easyco-shipping'
        );
    }
}
```

The exact implementation should follow the conventions established by the actual EasyCo codebase.

---

# 10. Events

Events should be preferred when EasyCo needs to react to commerce activity.

Examples:

```text
OrderPlaced
     ↓
┌────┴─────────────┐
│                  │
Analytics        Marketing
│                  │
Track sale       Send event
```

Another example:

```text
ProductUpdated
       ↓
 ┌─────┼─────┐
 │     │     │
Cache Feed  AI
```

Modules should avoid tightly coupling themselves to unrelated modules.

---

# 11. Database Ownership

EasyCo modules may create their own database tables.

For example:

```text
Bagisto tables
    ↓
Owned by Bagisto

EasyCo tables
    ↓
Owned by EasyCo modules
```

An EasyCo module must not directly alter Bagisto tables unless there is a documented architectural reason.

When interaction with existing Bagisto tables is necessary, prefer:

* relationships;
* extension points;
* events;
* configuration;
* services;
* observers;
* supported model mechanisms.

Direct schema modification should be treated as an exception.

---

# 12. Models

EasyCo should avoid duplicating Bagisto domain models unnecessarily.

For example, EasyCo should not create:

```text
EasyCoProduct
EasyCoOrder
EasyCoCustomer
```

simply because those concepts already exist in Bagisto.

Instead, EasyCo should extend or reference the existing commerce entities where appropriate.

However, EasyCo may create domain-specific models.

For example:

```text
ShippingRule
ShippingZone
ShippingPoint
PosSale
PosSession
MarketingEvent
SecurityRule
```

These represent functionality owned by EasyCo.

---

# 13. Contracts

Contracts should be introduced when they provide a real architectural benefit.

Examples:

```php
interface ShippingProvider
{
    public function calculate(
        ShippingRequest $request
    ): ShippingRateCollection;
}
```

A provider implementation could then be:

```text
ShippingProvider
      │
      ├── SpeedyProvider
      ├── EcontProvider
      └── BoxNowProvider
```

This allows the shipping engine to remain independent from individual courier implementations.

However:

> Do not create interfaces merely because "interfaces are good practice."

Introduce abstractions where multiple implementations, testing, replacement, or module boundaries justify them.

---

# 14. Module Dependencies

Modules should explicitly declare dependencies.

For example:

```text
easyco/pos
     │
     ├── easyco/core
     └── Bagisto
```

Marketing may depend on core:

```text
easyco/marketing
        │
        └── easyco/core
```

A future advanced analytics module could depend on:

```text
easyco/analytics
        │
        ├── easyco/core
        └── easyco/marketing
```

Circular dependencies must be avoided.

---

# 15. Versioning

EasyCo packages should follow Semantic Versioning where practical:

```text
MAJOR.MINOR.PATCH
```

For example:

```text
1.0.0
1.1.0
1.1.1
2.0.0
```

General meaning:

### PATCH

Bug fixes without intended breaking changes.

### MINOR

Backward-compatible functionality.

### MAJOR

Breaking API or architectural changes.

---

# 16. Compatibility

Each EasyCo module should eventually declare compatibility information.

Conceptually:

```text
EasyCo Shipping 1.4
Compatible with:

EasyCo:
    ^1.0

Bagisto:
    ^2.4

Laravel:
    ^13
```

The exact compatibility mechanism will be defined during implementation.

The important principle is:

> Compatibility must be explicit.

---

# 17. Upgrade Strategy

EasyCo should aim for the following upgrade path:

```text
EasyCo 1.0
    │
    ├── EasyCo modules remain compatible
    │
    ▼
Bagisto upgrade
    │
    ▼
Run migrations/tests
    │
    ▼
EasyCo continues operating
```

A Bagisto upgrade should not require manually reapplying custom edits to Bagisto source files.

If an EasyCo module depends on an internal Bagisto implementation detail, that dependency should be documented and isolated.

---

# 18. Configuration

Configuration should use Laravel configuration files where appropriate.

Example:

```text
config/
└── easyco/
    ├── core.php
    ├── shipping.php
    ├── marketing.php
    ├── security.php
    └── pos.php
```

Configuration should distinguish between:

* developer configuration;
* system configuration;
* merchant settings.

Merchant settings should generally be stored through the application's settings mechanism rather than requiring manual editing of configuration files.

---

# 19. Admin Integration

EasyCo modules should integrate into the Bagisto administration without unnecessarily replacing the entire admin application.

Possible approaches include:

* additional menu items;
* configuration pages;
* dashboards;
* widgets;
* resources;
* relation managers;
* custom pages;
* custom components.

The exact integration mechanism should be determined per module.

The goal is:

> **Extend the administration experience rather than fork it.**

---

# 20. Frontend Integration

EasyCo modules may contribute storefront functionality through:

* Blade views;
* components;
* assets;
* routes;
* APIs;
* theme extension points.

Modules should avoid assuming a single permanent storefront implementation.

This allows the EasyCo storefront to evolve independently.

---

# 21. Testing

Each EasyCo module should contain its own tests.

At minimum:

```text
Unit tests
Feature tests
Integration tests
```

where appropriate.

Important module behavior should not depend solely on manual testing.

For integrations such as couriers or Meta APIs, external services should be mockable.

---

# 22. Documentation

Every public EasyCo module should eventually contain:

```text
README.md
```

with:

* purpose;
* installation;
* dependencies;
* configuration;
* usage;
* compatibility;
* extension points;
* known limitations.

Architecturally significant modules should also have corresponding Blueprint documentation.

---

# 23. What We Should Avoid

EasyCo should explicitly avoid:

### Bagisto Core Forking

```text
Bagisto fork
    ↓
custom EasyCo modifications
```

unless absolutely necessary.

### Giant EasyCo Package

```text
easyco/
    everything/
```

This would recreate the monolithic architecture we are trying to avoid.

### Excessive Abstraction

Do not create:

```text
Interface
    ↓
AbstractFactory
    ↓
StrategyFactory
    ↓
AdapterFactory
    ↓
Implementation
```

when a simple service would solve the problem.

### Hidden Dependencies

Modules should not rely on undocumented behavior of unrelated modules.

---

# 24. Decision

The proposed EasyCo architecture is:

```text
Laravel
   │
Bagisto
   │
EasyCo Core
   │
┌──┼─────────────┬──────────────┐
│  │             │              │
POS Shipping Marketing     Security
│  │             │              │
└──┴─────────────┴──────────────┘
            │
        Analytics
            │
            AI
```

Each major EasyCo capability should be implemented as an independent module where practical.

The Bagisto core should remain untouched.

---

# 25. Future Consideration

The architecture should leave room for a future adapter layer if EasyCo eventually supports another commerce engine.

However, this is **not a v1 requirement**.

We should not build abstractions for hypothetical engines before EasyCo has a real need for them.

---

# 26. Acceptance Criteria

This RFC can be considered accepted when:

* EasyCo modules can be installed independently.
* Modules can declare dependencies.
* EasyCo modules do not require Bagisto core modifications.
* Module migrations are isolated.
* Module configuration is isolated.
* Modules can be tested independently.
* Bagisto upgrades do not require manually reapplying EasyCo source changes.
* The architecture remains understandable to developers and AI coding agents.

---

# 27. Final Principle

The extension architecture exists to protect EasyCo from its own success.

As the project grows:

```text
More modules
     ↓
More contributors
     ↓
More integrations
     ↓
More merchants
```

should not result in:

```text
More core modifications
     ↓
More coupling
     ↓
More upgrade problems
     ↓
Technical debt
```

The desired direction is:

```text
More functionality
       ↓
More independent modules
       ↓
Stable boundaries
       ↓
Predictable upgrades
```

**EasyCo should be easy to extend without becoming difficult to maintain.**

---

# 28. Acceptance Note

Accepted 2026-08-14.

At acceptance, the implementation already satisfies the criteria in section 26:

* `EasyCo/Core` and `EasyCo/Admin` are installed as independent Composer packages via a local path repository.
* Core's `registerSubModules()` provides explicit, declared module registration (currently a hardcoded map — acceptable for the current module count, flagged for revisit as the platform grows).
* No Bagisto core files have been modified; all integration goes through Service Providers, `mergeConfigFrom()`, runtime PSR-4 registration, and in-memory translator injection.
* `EasyCo/Admin`'s migrations/config/routes are self-contained within the module.
* The architecture has been validated against a running Bagisto application, not just documented in theory.

Open follow-ups tracked outside this RFC (see `ai/AI_CONTEXT.md` §17): replacing the hardcoded submodule map with real filesystem/manifest-based discovery, and adding automated tests for Core bootstrapping and submodule registration.

# AI Development Context: EasyCo (Леснотийка)

This file serves as the system prompt and contextual anchor for all AI-assisted coding sessions on the EasyCo project. Read this file carefully before generating any code or proposing architectural changes.

---

## 1. Project Overview
* **Name:** EasyCo (Леснотийка)
* **Vision:** A modular, high-performance, and secure e-commerce platform that combines WooCommerce's ease of use with the robust architecture of Laravel and Bagisto.
* **Target Audience:** Small and medium-sized businesses.

---

## 2. Technical Stack
* **Base Commerce Engine:** Bagisto (v2.x)
* **Framework:** Laravel (v10.x / v11.x)
* **Frontend:** Tailwind CSS, Blade Templates, and Vue.js (depending on Bagisto's frontend stack)
* **Database:** MySQL / PostgreSQL (via Laravel Eloquent ORM)

---

## 3. The Golden Rule (Architecture Constraint)
> **NEVER MODIFY THE BAGISTO CORE FILES.**
Any changes, new features, or modifications must be developed strictly as **Laravel Packages** inside the `packages/` directory.

### Extensibility Guidelines:
* Use **Laravel Event Listeners** and Bagisto's internal events to hook into the checkout, order, and product life cycles.
* Use **Service Providers** to load custom routes, controllers, views, and migrations.
* Override Bagisto views only by registering custom themes or using view-overriding techniques supported by Laravel/Bagisto.
* Extend Eloquent models using inheritance or custom relationships, never by editing files inside `vendor/` or the core package directory.

---

## 4. Current Phase: Phase 1 – Foundation (EasyCo Core)
We are currently starting **Phase 1 (EasyCo Core)**. The initial goals are:
* Establish the base package structure under `packages/EasyCo/Core`.
* Register the core service provider.
* Set up configurations that override or adjust Bagisto defaults without core changes.

---

## 5. Coding Principles for AI Assistants
When writing or proposing code for EasyCo, ensure you:
1. **Prefer Convention over Configuration:** Use standard Laravel and Bagisto directory structures.
2. **Write Clean, Clean Code:** Focus on Object-Oriented PHP, strictly typed methods, and proper docblocks where necessary.
3. **Use Repository Pattern:** Follow Bagisto's database interaction model by using Repositories instead of direct Eloquent queries in controllers where practical.
4. **Security by Default:** Always validate requests using Laravel FormRequests, sanitize user inputs, and ensure proper authorization gates are checked.
5. **Keep Upgrade Path Clean:** Ensure that updating the underlying Bagisto version in `composer.json` will not break EasyCo packages.

---

## 6. Project Directory Structure (For reference)
```text
bagisto-root/
├── app/
├── config/
├── packages/
│   └── EasyCo/
│       ├── Core/            <-- Current Focus
│       ├── Admin/
│       └── Shipping/
└── vendor/
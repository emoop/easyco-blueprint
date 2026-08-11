# Admin Experience (Easy Admin)

The administrative panel is the primary touchpoint for merchants managing their online stores. While the underlying commerce engine (Bagisto) provides a solid technical framework, its native user interface is often too complex and fragmented for small and medium-sized businesses. 

The goal of **Easy Admin** is to hide this complexity under an intuitive, simplified, and unified administrative experience, matching the usability of WooCommerce while preserving Laravel's robust scalability.

---

## 1. Architectural Core Principles
To respect our strict **"Zero Modifications to Bagisto Core"** principle, Easy Admin relies on non-intrusive extension patterns rather than direct code overrides.

### A. Runtime Menu & ACL Customization
We do not modify Bagisto's `Webkul/Admin` configuration files. Instead, `EasyCo\Admin` dynamically registers new menus, submenus, and access control lists by merging configurations at runtime:

* **Main Sidebar Items:** Registered under unique keys (e.g., `easyco`).
* **Submenu Injection:** Attached to core menus using dot-notation (e.g., `catalog.easyco_inventory` to inject a submenu under the core "Catalog" menu).

    $this->mergeConfigFrom(
        dirname(__DIR__) . '/Config/menu.php',
        'menu.admin'
    );

### B. Gateway Autoloading Pattern
To avoid manual edits to the root `composer.json` for every new EasyCo feature, the **EasyCo Core** package acts as a gateway. It dynamically registers the PSR-4 namespaces and Service Providers of EasyCo submodules (such as `Admin`, `Shipping`, or `POS`) at runtime:

    $loader = require base_path('vendor/autoload.php');
    $loader->addPsr4("EasyCo\\Admin\\", base_path('packages/EasyCo/Admin/src'));
    $this->app->register(\EasyCo\Admin\Providers\AdminServiceProvider::class);

### C. Safe Memory-Injected Localization with Fallback
Overriding translations in Laravel by simply swapping the namespace path removes all core translation keys, which breaks the application when fallback languages are used. 

To solve this, Easy Admin uses an **in-memory injection pattern**:
1. Load the original, stable English (`en`) translations from Bagisto Core as a base.
2. Load the custom, high-quality Bulgarian (`bg`) translations from the EasyCo package.
3. Recursively merge the Bulgarian translations over the English ones.
4. Inject this merged array directly into Laravel's `Translator` in-memory `$loaded` cache.

This ensures that **any untranslated string gracefully defaults to English** instead of displaying raw key paths like `admin::app.some.untranslated.key`.

---

## 2. Planned Interface Redesigns

### WooCommerce-Style Product Editor
Bagisto's native product creation requires selecting SKU and attribute families, forcing page reloads, and scattering options across multiple, complex tabs. 

Easy Admin will consolidate this flow into a single, unified page:
* **Primary Column:** Title, main description, short description, gallery, and product data panels (Inventory, Shipping, Variations as accordion cards).
* **Sidebar Column:** Publishing controls, category assignment (with quick-add functionality), and main product image.

### Simplified Config System
Bagisto's configuration pages contain hundreds of deep, technical settings. Easy Admin will introduce a "Simple Mode" where 90% of the advanced options are hidden behind an accordion, exposing only what the merchant needs to get started (Store Name, Contact Email, Currency, and Tax Rules).

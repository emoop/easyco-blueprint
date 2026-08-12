# Admin & Storefront Experience (Easy Admin & Shop)

The administrative panel and the storefront (shop) are the primary touchpoints for merchants and customers. While the underlying commerce engine (Bagisto) provides a solid technical framework, its native user interface and default translations are often too complex or unlocalized for the local market.

The goal of **EasyCo** is to simplify and localize this entire experience under an intuitive, unified, and fully Bulgarian-localized environment, preserving Laravel's robust scalability without altering the core engine.

---

## 1. Architectural Core Principles
To respect our strict **"Zero Modifications to Bagisto Core"** principle, both Admin and Storefront custom designs and translations rely on non-intrusive extension patterns.

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

### C. Safe Memory-Injected Localization (Admin & Storefront)
Overriding translations in Laravel by simply swapping the namespace path removes all core translation keys, breaking the application when fallback languages are used. 

To solve this, EasyCo implements a unified **in-memory translation injection pattern** for both the Admin Panel and the Storefront (Shop):

* **Admin Namespace (`admin::app`):** Custom translations in `EasyCo/Admin/.../lang/bg/app.php` are merged recursively over Bagisto's core English admin translations (`Webkul/Admin/.../lang/en/app.php`).
* **Storefront Namespace (`shop::app`):** Custom storefront translations in `EasyCo/Admin/.../lang/bg/shop.php` are merged recursively over Bagisto's core English storefront translations (`Webkul/Shop/.../lang/en/app.php`).

The process operates entirely in-memory during boot:
1. Load the original, stable English (`en`) translations from Bagisto Core as a base.
2. Load the custom, high-quality Bulgarian (`bg`) translations from the EasyCo package.
3. Recursively merge the Bulgarian translations over the English ones using PHP's `array_replace_recursive`.
4. Inject these merged arrays directly into Laravel's `Translator` in-memory `$loaded` cache (`$loaded['admin']['app']['bg']` and `$loaded['shop']['app']['bg']`).

This approach achieves:
* **100% Core Upgrade-Friendliness:** No core files are touched, and zero modifications are made to the root `composer.json` or `config/app.php` locales.
* **Seamless Fallbacks:** Any key not yet translated into Bulgarian immediately and gracefully defaults to its English counterpart, eliminating raw translation key paths (e.g., `shop::app.checkout.mini-cart...`) from being displayed on the storefront.

---

## 2. Planned Interface Redesigns

### WooCommerce-Style Product Editor
Bagisto's native product creation requires selecting SKU and attribute families, forcing page reloads, and scattering options across multiple, complex tabs. 

Easy Admin will consolidate this flow into a single, unified page:
* **Primary Column:** Title, main description, short description, gallery, and product data panels (Inventory, Shipping, Variations as accordion cards).
* **Sidebar Column:** Publishing controls, category assignment (with quick-add functionality), and main product image.

### Simplified Config System
Bagisto's configuration pages contain hundreds of deep, technical settings. Easy Admin will introduce a "Simple Mode" where 90% of the advanced options are hidden behind an accordion, exposing only what the merchant needs to get started (Store Name, Contact Email, Currency, and Tax Rules).
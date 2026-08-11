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

```php
$this->mergeConfigFrom(
    dirname(__DIR__) . '/Config/menu.php',
    'menu.admin'
);
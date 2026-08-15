# EasyCo Product Model and Product Editor

## Status

**Planned — Product Management**

This document defines the intended EasyCo approach to product creation, editing, variations, SKUs, and cost prices.

The implementation has not yet started.

The current Bagisto product management interface remains the underlying system until the EasyCo product editor is implemented.

## Goal

EasyCo should provide a product management experience that is significantly simpler and more intuitive than the default Bagisto product interface.

The merchant should primarily think in terms of:

Product
├── Main information
├── Images
├── Pricing
├── Inventory
└── Variations

rather than treating every variation as an independent product.

The intended user experience is closer to the WooCommerce product editor, while retaining Bagisto's stronger underlying support for configurable products and variation-specific data.

## Core Product Concept

A configurable product is one product.

Its variations belong to that product and should be managed together.

Conceptually:

Product
│
├── Parent SKU
├── Name
├── Description
├── Images
├── Base information
│
└── Variations
    ├── Variation A
    │   ├── Attributes
    │   ├── SKU
    │   ├── Price
    │   ├── Cost
    │   └── Inventory
    │
    ├── Variation B
    │   ├── Attributes
    │   ├── SKU
    │   ├── Price
    │   ├── Cost
    │   └── Inventory
    │
    └── Variation C
        ├── Attributes
        ├── SKU
        ├── Price
        ├── Cost
        └── Inventory

The parent product remains the primary merchant-facing entity.

Variations are properties of that product rather than separate products from the merchant's point of view.

## Parent SKU

A configurable product has its own SKU.

The parent SKU should remain visible and editable in the EasyCo product editor.

Variations may also have their own SKUs where required by the underlying commerce engine and inventory system.

Therefore EasyCo should support both:

**Parent product SKU**

and:

**Variation SKU**

These should not be presented as contradictory concepts.

The parent SKU identifies the overall product, while a variation SKU identifies a specific sellable variant.

## Variation Presentation

The EasyCo product list should not present every variation as a separate product by default.

Instead, the product table should primarily show the parent products.

For example:

**Mayoral T-Shirt**
SKU: MAY-123
8 variations

rather than displaying:

**Mayoral T-Shirt - 8Y**
**Mayoral T-Shirt - 10Y**
**Mayoral T-Shirt - 12Y**
**Mayoral T-Shirt - 14Y**
**Mayoral T-Shirt - 16Y**

as independent products.

This is an important difference between the EasyCo merchant experience and the default Bagisto presentation.

The objective is to make the catalog manageable when a store has many configurable products with numerous variations.

## Product List

The main EasyCo product table should represent products, not individual variations.

A configurable product may display summarized information such as:

**Product**
**SKU**
**Type**
**Price**
**Cost**
**Stock**
**Variations**
**Status**

For configurable products, the Variations column may show the number of available variations.

Example:

**Mayoral T-Shirt**
SKU: MAY-123
8 variations

The merchant should be able to open the product and manage all variations from the same product editor.

A variation-specific view may be available when necessary, but it should be secondary rather than the default catalog representation.

## Variation Editor

The product editor should provide a dedicated variation section.

A possible structure is:

Product
├── General
├── Images
├── Pricing
├── Inventory
├── Shipping
└── Variations

The variation section should allow the merchant to manage:

* variation attributes
* variation SKU
* variation price
* variation cost
* variation stock
* variation weight where applicable
* variation images where applicable
* other variation-specific properties supported by the commerce engine

The merchant should not need to leave the product editor to manage these values.

## Variation Price

A variation may have a different selling price from the parent product or from other variations.

EasyCo should support this.

For example:

Product: Children's Jacket

2–9 years — €39.90
10–18 years — €44.90

The variations remain part of the same product.

Different prices therefore do not require creating separate parent products.

## Variation Cost

Bagisto supports a cost value for individual product variations.

EasyCo should preserve and expose this capability.

This is particularly useful for stores where acquisition cost differs between variations.

For example:

Product: Children's Jacket

2–9 years
Cost: €18.00

10–18 years
Cost: €21.00

The accounting and profitability calculations can then use the actual cost associated with the sold variation.

This provides more accurate profitability analysis than storing one universal cost value for the entire parent product.

## Shared Cost

Although variation-specific cost is useful, many stores purchase all variations of a product at the same cost.

In that situation, requiring the merchant to enter the same cost repeatedly would make product management unnecessarily slow.

EasyCo should therefore provide a convenient shared-cost mechanism.

The product editor should allow the merchant to enter one cost value for the entire product and use it as the default cost for its variations.

Conceptually:

**Product Cost**

[ €18.00 ]

**Apply to all variations**

The merchant should then be able to override the cost for individual variations where necessary.

Example:

Variation | Cost
2–9 years | €18.00
10–18 years | €21.00

The product-level value provides the fast path.

The variation-level value provides the flexibility.

## Cost Inheritance

The intended behaviour is:

Product cost
↓
Default for variations
├── Variation A → inherited
├── Variation B → inherited
└── Variation C → overridden

A variation should therefore not require manually entering a cost when all variations share the same acquisition price.

Where a variation has a different cost, the merchant should be able to override the inherited value.

The exact persistence mechanism should be determined during implementation based on Bagisto's existing product and inventory model.

EasyCo should avoid duplicating cost data unnecessarily.

## Profitability

The variation cost should be available to future EasyCo analytics and reporting.

When a variation is sold, profitability calculations should use the cost associated with that specific variation.

Conceptually:

Selling price − Variation cost = Gross product margin

This becomes particularly important when different age groups, sizes, colors, or other variations have different acquisition costs.

The EasyCo POS and analytics modules should therefore treat variation cost as a first-class value.

## WooCommerce Compatibility

The EasyCo product editor should aim for the usability advantages demonstrated by WooCommerce while not copying its underlying data model unnecessarily.

WooCommerce provides a useful merchant-facing concept:

One product
↓
Variations

EasyCo should preserve this simplicity.

However, EasyCo can take advantage of Bagisto's richer variation model where it provides useful functionality, including variation-specific:

* SKU
* price
* cost
* inventory
* other product data

The objective is therefore:

**WooCommerce-like simplicity + Bagisto's variation capabilities = EasyCo Product Editor**

## Product Import

EasyCo should eventually provide a straightforward product import mechanism for merchants moving from another e-commerce platform.

WooCommerce should be considered an important migration source.

The import system should support, where practical:

* simple products
* configurable products
* variations
* parent SKU
* variation SKU
* attributes
* prices
* cost
* stock
* images
* categories
* brands
* descriptions
* product status

The importer should map source-platform concepts into EasyCo's product model rather than blindly reproducing the source database structure.

## CSV Import

CSV should be considered the primary simple migration format.

A merchant should be able to export products from another platform and import them into EasyCo without manually recreating every product.

For configurable products, the import format should preserve the relationship between:

Parent product
↓
Variations

rather than creating unrelated products for every variation.

## Import and Cost

The CSV import system should support both:

**product-level cost**

and:

**variation-level cost**

When a product contains one shared cost, the importer should be able to populate the product-level/default cost.

When the source data contains different costs for variations, those values should be imported as variation-specific costs.

This should allow EasyCo to support both simple and detailed catalog imports.

## Product Editor UX Principle

The EasyCo editor should optimize for the most common merchant workflow.

A merchant should be able to create a normal product quickly without being confronted with unnecessary advanced settings.

For example:

Product name
Description
Images
Price
Cost
Stock
Categories
Variations

Advanced options should remain available without dominating the interface.

## Product Editor and Bagisto

The EasyCo product editor should operate as an extension layer over Bagisto.

It should not modify Bagisto core product implementation unless a specific architectural decision determines that this is unavoidable.

The preferred approach is to use:

* Bagisto models
* Bagisto repositories where appropriate
* existing product services
* existing contracts
* events
* configuration
* extension points
* EasyCo adapters where necessary

The EasyCo editor should provide a better interface while keeping the underlying commerce engine compatible with Bagisto.

## Architectural Boundary

The distinction should remain clear:

EasyCo Product Editor
↓
EasyCo Product Services / Adapters
↓
Bagisto Product Model
↓
Laravel

The merchant-facing model should belong to EasyCo.

The underlying commerce persistence remains initially provided by Bagisto.

## Important Constraint

EasyCo should not immediately replace Bagisto's product system.

The first implementation should wrap and simplify the existing functionality.

Only after the actual limitations of Bagisto's product model are understood should EasyCo introduce additional abstraction or its own persistence model.

This follows the project's general principle:

> Solve the real problem first. Abstract only when necessary.

## Planned Product Editor

The first EasyCo product editor should aim for a single-page workflow.

A possible structure is:

Product
├── Basic information
├── Media
├── Pricing
├── Inventory
├── Shipping
└── Variations

Each section may be represented as an accordion/card.

The merchant should be able to complete the common workflow without navigating Bagisto's existing multi-tab/reload-heavy interface.

## Current Status

The product editor is **not yet implemented**.

The following are architectural decisions for the upcoming implementation:

* configurable products remain one merchant-facing product;
* the parent product has its own SKU;
* variations may have their own SKUs;
* variations should not appear as independent products in the main product table by default;
* variations remain managed together with the parent product;
* variations may have different selling prices;
* variations may have different acquisition costs;
* a shared product-level cost should be available as a convenient default;
* individual variation costs should be overridable;
* future profitability calculations should use the actual variation cost;
* CSV import should eventually support parent products and variations;
* WooCommerce should be considered an important migration source;
* the EasyCo editor should provide a WooCommerce-like merchant experience while retaining useful Bagisto capabilities.

## Next Implementation Step

The next implementation should not attempt to build the entire product editor at once.

The first step should be to inspect Bagisto's existing product creation/editing flow and determine:

1. how configurable products and their variations are persisted;
2. how parent and variation SKUs are represented;
3. how variation cost is stored;
4. how variation price is stored;
5. how inventory is associated with variations;
6. which existing Bagisto services/repositories/contracts can be reused;
7. which parts of the existing admin UI can be replaced or extended without modifying `packages/Webkul/`.

Only after this inspection should the first EasyCo product-editor milestone be implemented.

The first implementation should remain small, testable, and reversible.

The objective is not to replace Bagisto's product system immediately.

The objective is to create a significantly better merchant-facing product workflow on top of it.

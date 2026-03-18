---
name: printful-rest-api-v2
description: Interact with the advanced Printful REST API V2 (Beta). Manage orders, catalogs, mockups, files, and more using powerful modernized V2 endpoints. Includes full OpenAPI spec and API guidelines.
---

# Printful API V2 Skill

This skill allows agents to use the modern, updated **V2 Printful API** to build integrations and manage e-commerce print-on-demand operations. The V2 API brings significant improvements over V1, including a leaky bucket rate limit scheme and RFC 9457 structured errors.

## Base URL and Prefix
All V2 endpoints include the `/v2/` prefix.
**Base URL:** `https://api.printful.com/v2/`

## Authentication
Like V1, you can use a **Private API Token** for single-store use, or OAuth for public apps.
Include the token in the `Authorization` header:
`Authorization: Bearer <your_private_token>`

When using an Account-level token, also include the `X-PF-Store-Id: <store_id>` header to act on a specific store.

## Rate Limiting (Leaky Bucket)
The V2 API uses a leaky bucket algorithm. The default limit is **120 requests per 60 seconds** (varies per endpoint).
Check these headers in the HTTP response:
`X-Ratelimit-Policy: 120;w=60;`
`X-Ratelimit-Limit: 120`
`X-Ratelimit-Remaining: 119`
`X-Ratelimit-Reset: 0.5` *(Seconds until a single rate limit token is restored)*

## Available API References (V2)
The Printful API V2 is extensive. To make working with it efficient, use the provided auto-generated reference documents located in the `references/` directory. Each reference document contains explicit descriptions and fully resolved JSON schemas for the requests and responses.

### 1. Topic-Specific API References (Start Here)
- [Rate Limiting](references/Rate_Limiting.md)
- [Localisation](references/Localisation.md)
- [OAuth Scopes v2](references/OAuth_Scopes_v2.md)
- [Catalog v2](references/Catalog_v2.md)
- [Orders v2](references/Orders_v2.md)
- [Files v2](references/Files_v2.md)
- [Countries v2](references/Countries_v2.md)
- [Shipping Rates v2](references/Shipping_Rates_v2.md)
- [Warehouse Products v2](references/Warehouse_Products_v2.md)
- [Mockup Generator v2](references/Mockup_Generator_v2.md)
- [Webhook v2](references/Webhook_v2.md)
- [Stores v2](references/Stores_v2.md)
- [Approval Sheets v2](references/Approval_Sheets_v2.md)
- [Basic use cases](references/Basic_use_cases.md)
- [Examples](references/Examples.md)

### 2. Original Developer Specifications
- [OpenAPI Specification V2 Schema](references/printful-openapi-v2.json)
- [Postman Collection V2](references/printful_postman_collection_v2.json)

## Key Principles & Best Practices
1. **Catalog vs. Products**: The Printful Catalog contains all available blank products and their generic `variant_id`s. **Always use the variant_id (not product_id)** when specifying exactly what item you are dealing with in an Order or Mockup.
2. **Order Building (V2)**: Unlike V1, you can create a draft order without items, then flexibly add items directly via `POST /v2/orders/:order_id/order-items`.
3. **Async Order Costs**: After you create or modify an order, prices are calculated asynchronously. The response might say `"calculation_status": "calculating"`. You can poll `GET /v2/orders/:id` until it says `"done"`.
4. **V1 Fallbacks**: Product syncing features are retired in V2 beta. If you are asked to manage store sync products, you must use the V1 API.

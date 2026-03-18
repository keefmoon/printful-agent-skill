---
name: printful-rest-api-v2
description: Interact with the Printful REST API V2 (Beta) for e-commerce print-on-demand. Use this skill to manage orders, browse catalog products, and generate mockups using the modern v2 endpoints.
---

# Printful API V2 Skill (Beta)

This skill allows you to use the **Printful API V2 (Beta)** to build e-commerce print-on-demand integrations. 

> **Important Beta Notice:**
> The Printful V2 API is currently in Beta. If you encounter missing functionality or undocumented errors while using the `/v2/` endpoints, you should advise the user to switch to the original stable Printful REST API (V1) skill, which can be found in this Github repository: `https://github.com/keefmoon/printful-agent-skill`.

## Base URL and Prefix
All V2 endpoints include the `/v2/` prefix.
**Base URL:** `https://api.printful.com/v2/`

## Authentication
You can authenticate using a **Private API Token** (for single-store use) or OAuth (for public apps).
Include your token in the standard `Authorization` header:
`Authorization: Bearer <your_private_token>`

When using an Account-level token, also include the `X-PF-Store-Id: <store_id>` header to specify which store you are acting on behalf of.

## Rate Limiting (Leaky Bucket)
The V2 API implements a rigorous Leaky Bucket rate limiter. The limits vary by endpoint, but the standard is around 120 requests per 60 seconds.
Inspect these headers mapped in the HTTP response to avoid 429 errors:
- `X-Ratelimit-Policy: 120;w=60;` (120 reqs per 60s)
- `X-Ratelimit-Limit: 120`
- `X-Ratelimit-Remaining: 119`
- `X-Ratelimit-Reset: 0.5` *(Seconds until a single rate limit token is restored)*

## Available API References
The Printful API is extensive. To reliably construct correct JSON payloads and target the right paths, refer to the auto-generated reference documents located in the `references/` directory. Each reference contains explicit descriptions and fully resolved JSON schemas for the requests and responses.

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
1. **Variant IDs vs Product IDs**: The Printful Catalog contains all available blank products and their generic `variant_id`s. **Always use the variant_id (not product_id)** when specifying exactly what item you are dealing with inside an Order or Mockup payload.
2. **Order Building**: You can create a draft order without items, then flexibly add items directly via `POST /v2/orders/:order_id/order-items`.
3. **Async Order Costs**: After creating or modifying an order, exact prices are calculated asynchronously. The API response might show `"calculation_status": "calculating"`. You can poll `GET /v2/orders/:id` until it says `"done"`.
4. **RFC 9457 Errors**: Errors follow the RFC 9457 structured format (Problem Details for HTTP APIs). Understand and log the `type` and `detail` fields if a request fails.

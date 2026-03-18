---
name: printful-rest-api
version: 1.0.0
author: Keith Moon
description: Interact with the Printful REST API. Use this skill for managing products, catalog items, orders, mockups, file uploads, and shipping rates on Printful. Includes full OpenAPI spec and API guidelines.
---

# Printful REST API Skill

This skill enables agents to build integrations and manage operations using the Printful REST API.

## Authentication

All API requests require authentication. The simplest method is a **Private API Token**.
Include the token in the `Authorization` header:
`Authorization: Bearer <your_private_token>`

If you are using an Account-level token and need to target a specific store, also include the `X-PF-Store-Id: <store_id>` header.

Base URL for all requests: `https://api.printful.com/`

## Available Reference Materials

The Printful API is extensive. To make working with it efficient, use the provided reference documents located in the `references/` directory:

### 1. [API Reference Guide](references/API_REFERENCE.md)

Start here. This guide distills the most commonly used endpoints, such as checking the catalog, creating orders, generating mockups, and handling file uploads. Read this before attempting to blindly guess endpoints.

### 2. Topic-Specific API References

For detailed schemas, endpoints, and usage instructions for specific parts of the API, refer to these reference documents:

- [Localisation](references/Localisation.md)
- [Authorization](references/Authorization.md)
- [Using Private Token](references/Using_Private_Token.md)
- [Public App authorization](references/Public_App_authorization.md)
- [OAuth API](references/OAuth_API.md)
- [Catalog API](references/Catalog_API.md)
- [Products API](references/Products_API.md)
- [Product Templates API](references/Product_Templates_API.md)
- [Orders API](references/Orders_API.md)
- [File Library API](references/File_Library_API.md)
- [Shipping Rate API](references/Shipping_Rate_API.md)
- [Ecommerce Platform Sync API](references/Ecommerce_Platform_Sync_API.md)
- [Country/State Code API](references/Country_State_Code_API.md)
- [Tax Rate API](references/Tax_Rate_API.md)
- [Webhook API](references/Webhook_API.md)
- [Store Information API](references/Store_Information_API.md)
- [Mockup Generator API](references/Mockup_Generator_API.md)
- [Warehouse Products API](references/Warehouse_Products_API.md)
- [Reports API](references/Reports_API.md)
- [Approval Sheets API](references/Approval_Sheets_API.md)
- [Common](references/Common.md)
- [Other resources](references/Other_resources.md)
- [Basic use cases](references/Basic_use_cases.md)
- [Tutorials](references/Tutorials.md)
- [Errors](references/Errors.md)
- [Examples](references/Examples.md)

### 3. [Complete OpenAPI Specification](references/openapi.json)

If the above resources don't cover the specific endpoint or payload parameter you need, refer to the full OpenAPI specification. It contains detailed schemas for all request/response bodies.

### 4. [Postman Collection](references/printful_postman_collection.json)

This collection contains concrete examples of HTTP requests to the Printful API. Use it to see example JSON payloads and endpoint structures.

### 5. [Agent Skills Specification](references/skills-specification.md)

The specification of the exact format that this `SKILL.md` adheres to, in case you need to validate or extend this skill.

## Key Principles & Best Practices

1. **Catalog vs. Products**: The Printful Catalog contains all available blank products and their generic `variant_id`s. Your Store's Products are specific designs applied to Catalog variants.
2. **Variant IDs**: It's critical to use the `variant_id` (not `product_id`) when specifying the exact item in Orders, Products, or Mockup generation. Get the `variant_id` from the Catalog API.
3. **File Handling**: You can attach print files to an order simply by providing their URL: `{"url": "https://example.com/image.png"}`. Printful handles the downloading and processing automatically.
4. **Rate Limits**: The general rate limit is **120 requests per minute**. Operations like mockup generation have lower limits. Be respectful of rate limits when iterating over lists or syncing large catalogs.

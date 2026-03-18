# Printful API Reference Guide

This guide is a distilled summary of the Printful REST API. For exhaustive details, check `references/openapi.json`.

## Base URL & Authentication
- **Base URL**: `https://api.printful.com/`
- **Headers**: 
  - `Authorization: Bearer <your_token>`
  - (Optional) `X-PF-Store-Id: <store_id>` for Account-level tokens.
  - (Optional) `X-PF-Language: es_ES` for localized responses (default is `en_US`).
- **Response Format**: Status 200 for success. Result is wrapped in a `result` object, e.g., `{"code": 200, "result": {...}}`.

## 1. Catalog API
Browse blank products, variants, and print file constraints.
- `GET /categories` - List all catalog categories.
- `GET /products?category_id=<id>` - List products within a category.
- `GET /products/<id>` - Details for a product, including its variants (size, color) and their `id`s (variant IDs).
- `GET /mockup-generator/printfiles/<id>` - Get print file templates, dimensions, and placements for a catalog product ID.

## 2. Orders API
Create and confirm fulfillment orders.
- `GET /orders` - List all store orders.
- `POST /orders` - Create a draft order.
- `POST /orders?confirm=1` - Create and immediately submit an order for fulfillment.
- `POST /orders/<id>/confirm` - Confirm an existing draft order.

### Order Payload Example
```json
{
    "recipient": {
        "name": "John Doe",
        "address1": "19749 Dearborn St",
        "city": "Chatsworth",
        "state_code": "CA",
        "country_code": "US",
        "zip": "91311"
    },
    "items": [
        {
            "variant_id": 11513, // Must be a variant_id, not product_id!
            "quantity": 1,
            "files": [
                {
                    "url": "http://example.com/files/posters/poster_1.jpg"
                }
            ]
        }
    ]
}
```

## 3. Products (Store Sync)
Create and manage products in the integrated Printful store (not applicable for external e-commerce integrations like Shopify).
- `GET /store/products` - List synced products.
- `POST /store/products` - Create a new product.

## 4. File Library
Upload and manage files to be printed.
When making requests that need a file (orders, products), you can just pass a URL and Printful will add it to the library automatically.
- *Note*: Endpoint `/files` is deprecated. Do not use.

## 5. Mockup Generator API
Generate product mockup images for your store.
1. `POST /mockup-generator/create-task/<product_id>` - Create a task to generate mockups. Returns a `task_key`.
2. `GET /mockup-generator/task?task_key=<task_key>` - Check task status. When `status` is `completed`, it returns the mockup URLs. (Urls expire in 72 hours).

## 6. Shipping Rates
Calculate shipping costs before submitting an order.
- `POST /shipping/rates` - Get available shipping methods and rates for given items and recipient.

## Rate Limits
- General limit: 120 API calls per minute.
- Mockup generation and intensive endpoints may be strictly lower.

# Basic use cases

# Using the Printful v2 API for a Basic Use Case: Customer-Designed Products

This guide explains how to use the Printful v2 API for a basic use case where a merchant allows customers to design and order products through their shop. The integration involves fetching catalog products, enabling customers to design them, and creating orders for fulfillment.

---

## Overview of the Flow

1. **Authenticate**: Obtain your private API key from the Printful dashboard.
2. **Retrieve Catalog Products**: Fetch available catalog products to display to customers.
3. **Customize Products**: Allow customers to design products using your shop interface.
4. **Create Orders**: Send the customized product details and order data to Printful.
5. **Track Orders**: Use the API to track the status of orders and shipments.

---

## Sequence Diagram

The following diagram illustrates the interaction between the customer, the shop, and the Printful API:

<pre class="mermaid">
sequenceDiagram
    participant Customer as End Customer
    participant Shop as Merchant's Shop
    participant Printful as Printful API

    Customer->>Shop: Browse Catalog Products
    Shop->>Printful: GET /v2/catalog/products
    Printful->>Shop: Return Product List
    Shop->>Customer: Display Product Catalog

    Customer->>Shop: Design Product
    Shop->>Printful: POST /v2/orders (Draft Order)
    Printful->>Shop: Return Draft Order ID

    Customer->>Shop: Confirm Order
    Shop->>Printful: POST /v2/orders/{order_id}/confirm
    Printful->>Shop: Return Order Confirmation

    Shop->>Customer: Provide Order Confirmation and Tracking
</pre>


# Basic use cases

# Basic use cases

All the API integrations begin with a decision on how to authenticate with the Printful API. The most common use cases for Printful API integrations are:

<pre class="mermaid">
    graph TD
    A[Authentication Methods] --> B[Private API Token]
    A --> C[Public App OAuth]
    B --> D[Single Printful Account]
    C --> G[Single Printful Account of Marketplace Owner]
    G --> E[Multiple Printful Accounts]
    E --> H[Merchant 1]
    E --> I[Merchant 2]
    E --> J[Merchant 3]
</pre>

- Private API token
  Means that you are building a solution for your own store, using a single Printful account.

<pre class="mermaid">
    sequenceDiagram
    participant User as End User
    participant Shop as Single Shop
    participant Printful as Printful API

    User->>Shop: Access Single Shop
    Shop->>User: User Account Linked Successfully

    Shop->>Printful: Authenticate with API Token
    Printful->>Shop: Return Authentication Success

    User->>Shop: Request Product Catalog
    Shop->>Printful: Retrieve Product Catalog
    Printful->>Shop: Return Product Catalog
    Shop->>User: Display Product Catalog

    Note over Shop,User: User can now design and order products

    User->>Shop: Place Order
    Shop->>Printful: Create Order
    Printful->>Shop: Order Created Successfully
    Shop->>User: Order Confirmation and Tracking
</pre>

- Public App OAuth
  Is used for more complex integrations, where you are building an app that will be used by multiple users, each with their own Printful account. For example a merchant platform.

<pre class="mermaid">
sequenceDiagram
    participant User as End User
    participant App as Public App (Marketplace)
    participant Printful as Printful API

    User->>App: Access Public App (Marketplace)
    App->>User: Redirect to Printful Authorization Page
    User->>Printful: Authorize App (Grant Permissions)
    Printful->>User: Return Authorization Code
    User->>App: Provide Authorization Code
    App->>Printful: Exchange Authorization Code for Access Token
    Printful->>App: Return Access Token
    App->>User: User Account Linked Successfully

    User->>Printful: Request Product Catalog
    Printful->>User: Return Product Catalog

    Note over App,User: User can now design and order products

    User->>App: Place Order
    App->>Printful: Create Order on Behalf of User
    Printful->>App: Order Created Successfully
    App->>User: Order Confirmation and Tracking

    Note over App,Printful: App manages multiple users independently
</pre>

Both private api token and public app can be created in https://developers.printful.com

## Webshop integration offering pre-designed products to the end customers

1.Authenticate: Obtain your private API key from the Printful dashboard.

2.Set Up Webhooks: Configure webhooks to receive updates about orders, products, and other events.

3.Sync Products: Use the API to fetch and create designed products in the Printful.

4.Create Orders: Send order data to Printful when a customer places an order on a sync product.

5.Track Orders: Use the API to track the status of orders and shipments.

<pre class="mermaid">
    graph LR
    A[Obtain API Key] --> B[Set Up Webhooks]
    B --> C[Sync Products]
    C --> D[Create Orders]
    D --> E[Track Orders]
    E --> F[Update Webshop]
</pre>

## Webshop integration with the products designed on the fly by the end customers

1.Authenticate: Obtain your private API key from the Printful dashboard.

2.Retrieve Catalog Products: Use the API to fetch available catalog products and display them to customers for customization.

3.Customize Products: Allow customers to design products (e.g., upload images, select colors, etc.) using your webshop interface.

4.Create Orders on the Fly: When a customer places an order, send the customized product details and order data directly to Printful.

5.Track Orders: Use the API to track the status of orders and shipments.

6.Update Webshop: Update your webshop with order statuses and shipment tracking information.

<pre class="mermaid">
    graph LR
    A[Obtain API Key] --> B[Retrieve Catalog Products]
    B --> C[Customize Products]
    C --> D[Create Orders on the Fly]
    D --> E[Track Orders]
    E --> F[Update Webshop]
</pre>

## Webshop integration with the product catalog created by the merchant

<pre class="mermaid">
    sequenceDiagram
    participant User as End User
    participant Merchant
    participant Printful as Printful API

    User->>Merchant: Access Merchant Platform
    Merchant->>User: User Account Linked Successfully

    Merchant->>Printful: Authenticate with API Token
    Printful->>Merchant: Return Authentication Success
    Note over Merchant,Printful: Merchant can now design and sell products

    Merchant->>Printful: Retrieve Product Catalog
    Printful->>Merchant: Return Product Catalog

    Merchant->>Printful: Sync Product Data
    Printful->>Merchant: Product Synced Successfully

    User->>Merchant: Request sync product catalog
    Merchant->>Printful: Request sync products
    Printful->>Merchant: Return sync products
    Merchant->>User: Display sync product catalog

    User->>Merchant: Place Order
    User->>Printful: Create Order
    Printful->>Merchant: Order Created Successfully
    Merchant->>User: Order Confirmation and Tracking
</pre>

## Embedded Designer integration

Embedded designer is embeddable JavaScript component that allows the shop owner not to build their own product designer, but instead use Printful's product designer. It is used to allow design products by the end customers.
It is used in conjunction with Printful's product catalog and order API. The end customer can select a product, design it, and then order it.
The following diagram illustrates the flow of the embedded designer integration:

<pre class="mermaid">
graph TD
    subgraph Shop[Shop]
        style Shop fill:#f9f,stroke:#333,stroke-width:2px
        A[Displays Printful Product Catalog]
        subgraph EDM[Embedded Designer]
            style EDM fill:#ff9,stroke:#333,stroke-width:2px
            B[Designs Product]
            C[Saves Design]
        end
        D[Orders Product Template]
    end

    subgraph Printful[Printful API]
        style Printful fill:#99f,stroke:#333,stroke-width:2px
        E[Processes Order Request]
    end

    A --> B
    B --> C
    C --> D
    D --> E
    EDM -.-> Printful
</pre>

The more detailed flow of the embedded designer integration is as follows:

<pre class="mermaid">
    graph TD
    subgraph Shop[Shop]
        style Shop fill:#f9f,stroke:#333,stroke-width:2px
        A[Displays Printful Product Catalog]
        C[Retrieves Nonce Token via API]
        D[Embeds EDM JavaScript Component]
        E[Uses Nonce to Establish Single Use Session for EDM]
        H[Calls Printful API to Order Product Template]
    end

    subgraph EndCustomer[End Customer]
        style EndCustomer fill:#9f9,stroke:#333,stroke-width:2px
        B[Selects Product to Design]
        F[Designs Product in Embedded Designer]
        G[Saves Design, Producing a Product Template]
        I[Orders Product Template]
    end

    subgraph API[Printful API]
        style API fill:#99f,stroke:#333,stroke-width:2px
        J[Handles Nonce Token Request]
        K[Processes Order Request]
        L[Marks Nonce as Used]
        M[Invalidates Token]
    end

    A --> B
    B --> C
    C --> J
    J --> D
    D --> E
    E --> L
    E --> F
    F --> G
    G --> M
    G --> I
    I --> H
    H --> K
</pre>

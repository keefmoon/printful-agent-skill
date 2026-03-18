# Tutorials

## Make your first order through the API

### API Flow Overview

Here's a quick visual overview of how your system communicates with Printful's API during the order creation flow:

![Authentication setup](images/tutorial/api-integration/authentication_setup.png)
![OAuth integration flow](images/tutorial/api-integration/oauth_integration.png)
![Order creation and fulfillment](images/tutorial/api-integration/order_creation.png)
![Webhook handling](images/tutorial/api-integration/product_management.png)
![Product management](images/tutorial/api-integration/webhook_handling.png)

### Entity Relationship Reference

The following diagram shows key API entities:

![Entity relationship diagram](images/tutorial/entity_relationship_diagram.png)

### Introduction

In this tutorial, we're going to show you how to start making orders through the Printful API.
You will learn to send postcards to friends or customers through the API,
and select new images or addresses for every order.

You'll be able to take the lessons learned here and start making orders for any kind of product 
for your customers.

### Set up

Before anything else make sure you have created a native or "manual order platform store"
if you haven't already.

- Go to this link: https://www.printful.com/dashboard/store.
- Click the "Create" button under "Manual order platform/API"
- Choose a name and click continue

As well as this you will need to create an OAuth token for this store.

- Go to the developer portal: https://developers.printful.com/login
- Sign in with your Printful account
- Go to this link: https://developers.printful.com/tokens/add-new-token
- Make sure you have set the access level to "A single store" and select your store
- Make sure you have the following scopes selected for this tutorial:
   -  `orders` ("View and manage orders of the authorized store")
- Fill in all other fields as you please
- Create your token
- You will receive an access key, you will only see the token once, store it securely


To double-check that your token works, and that you have the right store selected, try running the following command
replacing `{your_token}` with the access token created in the developer portal.
```
curl --location --request GET 'https://api.printful.com/stores' \
--header 'Authorization: Bearer {your_token}'
```

You should see your store in the response. If you don't see the right store there, or you get an error,
double-check the token is correct and if necessary create a new token


### Making an order

Before starting let's figure out what we need to make an order through the Printful API.

According to [the docs](#operation/createOrder)
there are only two fields in the create orders request that are absolutely required,
the `recipient` object and the `items` array. So let's focus on those for now.

### The recipient

Let's start with the recipient, this will depend on the location you want to send it to. Mine will
look like this:

```
{
    "name": "recipients name",
    "address1": "12 address avenue, Bankstown",
    "city": "Sydney",
    "state_code": "NSW",
    "country_code": "AU",
    "zip": "2200"
}
```

Not all state and country codes are accepted by Printful, so I double-check to make sure that
AU and NSW are accepted by running this curl command:

```
curl --location --request GET 'https://api.printful.com/countries' \
--header 'Authorization: Bearer {your_token}'
```

```
{
    "code": 200,
    "result": [
        {
            "name": "Australia",
            "code": "AU",
            "states": [
                ...,
                {
                    "code": "NSW",
                    "name": "New South Wales"
                },
                ...
            ]
        }
    ],
    "extra": []
}
```

Sure enough, both codes are there.

**Note**: If your recipient isn't in the US, Australia, or Canada the state code is not necessary.
[Read more](#tag/CountryState-Code-API).

### The items

We only want one item, but first, we have to find the `variant_id` of that item.

Let's first see if we can find a category that might have postcards in it, with this command:

```
curl --location --request GET 'https://api.printful.com/categories' \
--header 'Authorization: Bearer {your_token}'
```

Thankfully there's a postcard category.

```
{
    "code": 200,
    "result": [
        ...,
        {
            "id": 197,
            "parent_id": 190,
            "image_url": "https://s3-printful.stage.printful.dev/upload/catalog_category/da/da6caac995335b87ace2d79af70eef5f_t?v=1652883254",
            "title": "Postcards"
        },
        ...
    ],
    "extra": []
}
```

Now let's see what options there are in that category
```
curl --location --request GET 'https://api.printful.com/products?category_id=197' \
--header 'Authorization: Bearer {your_token}'
```

```
{
    "id": 433,
    "main_category_id": 197,
    "type": "POSTCARD",
    "type_name": "Standard Postcard",
    "title": "Standard Postcard",
    "brand": null,
    "model": "Standard Postcard",
    "image": "https://s3-printful.stage.printful.dev/products/433/product_1602054891.jpg",
    "variant_count": 1,
    "currency": "USD",
    "options": [],
    "dimensions": null,
    "is_discontinued": false,
    "avg_fulfillment_time": null,
    "techniques": [
        {
            "key": "DIGITAL",
            "display_name": "Digital printing",
            "is_default": true
        }
    ],
    "files": [
        {
            "id": "default",
            "type": "default",
            "title": "Print file",
            "additional_price": null
        },
        {
            "id": "preview",
            "type": "mockup",
            "title": "Mockup",
            "additional_price": null
        }
    ],
    "description": "These postcards are made from thick high-quality matte paper, so they serve as a great addition to a gift or just a thoughtful written note to a friend.\n• Cardboard paper\n• Paper weight: 7.67–10.32 oz/yd² (260–350 g/m²)\n• Size: 4″ × 6″ (101 × 152 mm)\n• Paper thickness: 0.013″ (0.34 mm)\n• Coated outer surface\n• Blank product materials sourced from Sweden, US, Brazil, or China",
    "catalog_categories": {
        "0": 197,
        "1": 5,
        "2": 122,
        "4": 213,
        "5": 230,
        "6": 242
    }
}
```

The standard option looks good so let's get some more information about the product with:

```
curl --location --request GET 'https://api.printful.com/products/433' \
--header 'Authorization: Bearer {your_token}'
```

```
{
    "code": 200,
    "result": {
        "product": {
            ...
        },
        "variants": [
            {
                "id": 11513,
                "product_id": 433,
                "name": "Standard Postcards (4″×6″)",
                "size": "4″×6″",
                "color": null,
                "color_code": null,
                "color_code2": null,
                "image": "https://s3-printful.stage.printful.dev/products/433/11513_1592384192.jpg",
                "price": "1.50",
                "in_stock": true,
                "availability_regions": {
                    "US": "United States",
                    "EU": "Europe",
                    "EU_LV": "Latvia",
                    "AU": "Australia"
                },
                "availability_status": [
                    {
                        "region": "US",
                        "status": "in_stock"
                    },
                    {
                        "region": "EU",
                        "status": "in_stock"
                    },
                    {
                        "region": "EU_LV",
                        "status": "in_stock"
                    },
                    {
                        "region": "AU",
                        "status": "in_stock"
                    }
                ]
            }
        ]
    },
    "extra": []
}
```

Our choice here should be easy, there is only one variant, and it's available in the region
we need to send it to.

Now we can add this id (`11513`) to an item in our order payload.

````
{
    ...,
    "items": [
        {
            "variant_id": 11513
        }
    ]
}
````

Next, we need to add a `files` array to the item and decide on the "`quantity`" we want to print
(let's just print 1 for now). This way Printful knows what to print, and how many
items should be sent to the recipient.

````
{
    "variant_id": 11513,
    "quantity": 1,
    "files": [
        {
            "url": "http://example.com/files/posters/poster_1.jpg"
        }
    ]
}
````

### Putting it all together

Now we can finish our orders payload and create our first order.
The final payload should look like this:

````
{
    "recipient": {
        "name": "recipients name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
        {
            "variant_id": 11513,
            "quantity": 1,
            "files": [
                {
                    "url": "http://example.com/files/posters/poster_1.jpg"
                }
            ]
        }
    ]
}
````

We can now make a `POST` request to `https://api.printful.com/orders` with this payload, and
we'll get a draft order using a request like this:

```
curl --location --request POST 'https://api.printful.com/orders' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "recipient": {
        "name": "recipients name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
        {
            "variant_id": 11513,
            "quantity": 1,
            "files": [
                {
                    "url": "http://example.com/files/posters/poster_1.jpg"
                }
            ]
        }
    ]
}'
```

Now your order is ready to be made, you will even be able to see the draft already in your
dashboard here:
`https://www.printful.com/dashboard/default/orders`

### Confirming an order

To confirm the order we need the `id` of our new order, it should have been returned by the response to
the request above. If you've lost it, you can find it again by making a GET request to
`https://www.api.printful.com/orders` and find your order in the returned list.
```
curl --location --request GET 'https://api.printful.com/orders' \
--header 'Authorization: Bearer {your_token}'
```
Assuming you haven't made any other orders it will probably be the first item returned in that list.

Once you have your order id you can confirm the order and have it sent to fulfillment.

**Note**: If you do want the order to be fulfilled remember to [set up billing](https://www.printful.com/dashboard/billing/how-does-billing-work).
If billing is not set up first the order will fail.

**Warning**: Do not run the following command if you do not want to be charged for this order.
```
curl --location --request POST 'https://api.printful.com/orders/{your_order_id}/confirm' \
--header 'Authorization: Bearer {your_token}' \
```

### Automatically confirming new orders

Now I want to be able to send these postcards in one step, without 
needing to confirm every time.

To do that, all I have to do is add `confirm=1` as a query parameter to the `POST` request. 
So that the request looks like this: `POST https://api.printful.com/orders?confirm=1`

Now, whenever I need to send a postcard with a new image I just change the item file URL in this request:

**Warning**: Do not run the following command if you do not want to be charged for this order. 
```
curl --location --request POST 'https://api.printful.com/orders?confirm=1' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "recipient": {
        "name": "recipient name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
        {
            "variant_id": 11513,
            "quantity": 1,
            "files": [
                {
                    "url": "{change_this_url}"
                }
            ]
        }
    ]
}'
```

If I then wanted to send it to someone else all I'd need to do is change the address object.

### Next steps
Try making some more draft requests like this for different items, and see how they look in the dashboard.
Try moving the requests into an application you've already created so that your customers can make orders through a UI.

If you'd like to sell your own products that you've pre-built for your store you
can follow our tutorial on [creating products for your store through the API](#tag/Tutorials/Creating-products-for-your-store-through-the-API).


## Creating products for your store through the API

### Set up
To follow along with this tutorial, you will need to do the following:

Make sure you have created a native or
"manual order platform store" if you haven't already.

- Go to this link: https://www.printful.com/dashboard/store.
- Click the "Create" button under "Manual order platform/API"
- Choose a name and click continue

As well as this you will need to create an OAuth token for this store.

- Go to the developer portal: https://developers.printful.com/login
- Sign in with your Printful account
- Go to this link: https://developers.printful.com/tokens/add-new-token
- Make sure you have set the access level to "A single store" and select your store
- Make sure you have the following scopes selected:
    - `orders` View and manage orders of the authorized store
    - `sync_products` View and manage store products
- Fill in all other fields as you please
- Create your token
- You will receive an access token, you will only see the token once, store it securely

To double-check that your token works, and that you have the right store selected, try running the following command
replacing `{your_token}` with the access token created in the developer portal.
```
curl --location --request GET 'https://api.printful.com/stores' \
--header 'Authorization: Bearer {your_token}'
```

You should see your store there. If you don't see the right store there, or you get an error,
double-check the token is correct and if necessary create a new token

### Get Sync Products

I've created a new store, "My Photos Store", and I want to start selling landscapes.

The first thing I want to do is see what products I have in my store. To do this I will use the
[Products API](#tag/Products-API), which lets me create, modify and delete products
in my Printful store.

So I make this request with curl.

```
curl --location --request GET 'https://api.printful.com/store/products' \
--header 'Authorization: Bearer {your_token}'
```

And it looks like I have no products in my store yet.

**Note**: If you've already created some products you'll see them here.
```
{
    "code": 200,
    "result": [],
    "extra": [],
    "paging": {
        "total": 0,
        "offset": 0,
        "limit": 20
    }
}
```

To create a new product for our store we'll need to make a post request to that same endpoint.

According to [the docs](#operation/createSyncProduct),
there are two required fields, an object `sync_product` and an array `sync_variants`.

The product only has one required field, `name`, and the variants only require the `id`
and `files` fields to be present.

So to make a simple first product, let's try and build an object like this:

```
{
    "sync_product": {
        "name": "string"
    },
    "sync_variants": [
        {
            "variant_id": 1,
            "files": [
                {
                    "url": "string"
                }
            ]
        }
    ]
}

```

Our first step is to figure out the `variant_ids` that we need.

### The Printful catalog

My main goal is to sell landscape photos, so let's check the
[Get Categories endpoint](#operation/getCategories)
to see if there's anything suitable for that.

```
curl --location --request GET 'https://api.printful.com/categories' \
--header 'Authorization: Bearer {your_token}'
```

```
{
    "code": 200,
    "result": {
        "categories": [
            ...
            {
                "id": 55,
                "parent_id": 21,
                "image_url": "https://s3-printful.stage.printful.dev/upload/catalog_category/6f/6f2f0c50f2558af01e4f8eebbc09a66d_t?v=1652883254",
                "title": "Posters"
            },
            {
                "id": 56,
                "parent_id": 21,
                "image_url": "https://s3-printful.stage.printful.dev/upload/catalog_category/34/347883396e6a71fdb25121f20c85e2b3_t?v=1652883254",
                "title": "Framed posters"
            },
            {
                "id": 57,
                "parent_id": 21,
                "image_url": "https://s3-printful.stage.printful.dev/upload/catalog_category/7c/7c2dd885646f3971b7199ac833a0232f_t?v=1652883254",
                "title": "Canvas prints"
            },
            ...
        ]
    },
    "extra": []
}
```

Posters, Framed Posters and Canvas prints all sound suitable.
Though I'd prefer my photos framed before sending them to customers, so I will look into
the category with `id` `56`.

See if you can find a product that suits your needs best from the list returned from the `/categories` endpoint.

To see all options for framed posters let's use the `category_id` (`56`)
with the [Get Products endpoint](#operation/getProducts).

We can pass the `id` of the category we're interested in as a query parameter, so I'll pass `56` in as a parameter with a
request:

```
curl --location --request GET 'https://api.printful.com/products?category_id=56' \
--header 'Authorization: Bearer {your_token}'
```

This returns 4 products in an array:

```
{
    "code": 200,
    "result": [
        {
            "id": 2,
            "main_category_id": 56,
            "title": "Enhanced Matte Paper Framed Poster (in)",
            ...
        },
        {
            "id": 304,
            "main_category_id": 56,
            "title": "Enhanced Matte Paper Framed Poster (cm)",
            ...
        },
        {
            "id": 366,
            "main_category_id": 56,
            "title": "Framed Poster with Frame Mat (cm)",
            ...
        },
        {
            "id": 172,
            "main_category_id": 56,
            "title": "Premium Luster Photo Paper Framed Poster (in)",
            ...
        }
    ],
    "extra": []
}
```

This will give us a lot of information about the products,
but all we really need at this point is the `id`.

I'm interested in that "Premium Luster Photo Paper Framed Poster (in)" product,
so to find what variants are available to me, I'll search for it by its `id`
(`172`) and find the `variant_id`s I'm looking for.

```
curl --location --request GET 'https://api.printful.com/products/172' \
--header 'Authorization: Bearer {your_token}'
```

```
{
    "code": 200,
    "result": {
        "product": {
            "id": 172,
            "main_category_id": 56,
            "type": "FRAMED-POSTER",
            "type_name": "Premium Luster Photo Paper Framed Poster (in)",
            "title": "Premium Luster Photo Paper Framed Poster (in)",
            ...
        },
        "variants": [
            ...,
            {
                "id": 10760,
                "product_id": 172,
                "name": "Premium Luster Photo Paper Framed Poster (White/8″×10″)",
                "size": "8″×10″",
                "color": "White",
                "color_code": "#ffffff",
                "color_code2": null,
                "image": "https://s3-printful.stage.printful.dev/products/172/10760_1565081295.jpg",
                "price": "23.00",
                "in_stock": true,
                "availability_regions": {
                    "US": "United States",
                    "EU": "Europe",
                    "EU_LV": "Latvia",
                    "AU": "Australia",
                    "UK": "United Kingdom"
                },
                "availability_status": [
                    {
                        "region": "US",
                        "status": "in_stock"
                    },
                    {
                        "region": "EU",
                        "status": "in_stock"
                    },
                    {
                        "region": "EU_LV",
                        "status": "in_stock"
                    },
                    {
                        "region": "AU",
                        "status": "in_stock"
                    },
                    {
                        "region": "UK",
                        "status": "in_stock"
                    }
                ]
            }
        ]
    },
    "extra": []
}
```

The final variant with id `10760` is in stock in all the regions I want to sell in
and a small 8x10 sounds like a great first product for my store.

### Your first sync product and sync variant

Now we can finish this payload from before:

```
{
    "sync_product": {
        "name": "string"
    },
    "sync_variants": [
        {
            "variant_id": 1,
            "files": [
                {
                    "url": "string"
                }
            ]
        }
    ]
}
```

I'll name my product "Framed Landscape Poster" so my object will look like this:

```
{
    "sync_product": {
        "name": "Framed Landscape Poster"
    },
    "sync_variants": [
        {
            "variant_id": 10760,
            "files": [
                {
                    "url": "http://example.com/files/posters/poster_1.jpg"
                }
            ]
        }
    ]
}
```

So let's post that object to the Products endpoint:

```
curl --location --request POST 'https://api.printful.com/store/products' \
--header 'Authorization: Bearer {your_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
  "sync_product": {
    "name": "Framed Landscape Poster"
  },
    "sync_variants": [
        {
            "variant_id": 10760,
            "files": [
                {
                    "url": "http://example.com/files/posters/poster_1.jpg"
                }
            ]
        }
    ]
}'
```

And now we'll get our product back looking like this:

```
{
    "code": 200,
    "result": {
        "id": 280896090,
        "external_id": "6308d1498007d7",
        "name": "Framed Landscape Poster",
        "variants": 1,
        "synced": 1,
        "thumbnail_url": null,
        "is_ignored": false
    },
    "extra": []
}
```

And when I go back to see the products in my store

```
curl --location --request GET 'https://api.printful.com/store/products' \
--header 'Authorization: Bearer {your_token}'
```

I see the new product:

```
{
    "code": 200,
    "result": [
        {
            "id": 280896090,
            "external_id": "6308d1498007d7",
            "name": "Framed Landscape Poster",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": null,
            "is_ignored": false
        }
    ],
    "extra": [],
    "paging": {
        "total": 1,
        "offset": 0,
        "limit": 20
    }
}
```

And I can see more details about the product with this request now.

**Note**: Replace the `{your_product_id}` at the end with the `id` of your sync product, i.e.

```
curl --location --request GET 'https://api.printful.com/store/products/{your_product_id}' \
--header 'Authorization: Bearer {your_token}'
```

For me, the request will look like this, but you will have a different id:
```
curl --location --request GET 'https://api.printful.com/store/products/280896090' \
--header 'Authorization: Bearer {your_token}'
```


```
{
    "code": 200,
    "result": {
        "sync_product": {
            "id": 280896090,
            "external_id": "6308d1498007d7",
            "name": "Framed Landscape Poster",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": null,
            "is_ignored": false
        },
        "sync_variants": [
            {
                "id": 3374445206,
                "external_id": "6308d14981f197",
                "sync_product_id": 280896090,
                "name": "Framed Landscape Poster - Black / 10″×10″",
                "synced": true,
                "variant_id": 6883,
                "warehouse_product_id": null,
                "warehouse_product_variant_id": null,
                "retail_price": null,
                "sku": null,
                "currency": "USD",
                "product": {
                    "variant_id": 6883,
                    "product_id": 172,
                    "image": "https://s3-printful.stage.printful.dev/products/172/6883_1527683114.jpg",
                    "name": "Premium Luster Photo Paper Framed Poster (Black/10″×10″)"
                },
                "files": [
                    {
                        "id": 450621845,
                        "type": "default",
                        "hash": null,
                        "url": "http://example.com/files/posters/poster_1.jpg",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1661516795,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": [],
                "is_ignored": false
            }
        ]
    },
    "extra": []
}
```

### Ordering the new product

Now that I have the product in my store, I can very easily order
one of my sync variants as an item using only the `sync_variant_id`.

The payload might look something like this

``` 
{
    "recipient": {
        "name": "name",
        "address1": "address",
        "city": "city",
        "state_code": "STATE_CODE",
        "country_code": "COUNTRY_CODE",
        "zip": "2200"
    },
    "items": [
        {
            "sync_variant_id": 3374445206,
            "quantity": 1,
        }
    ]
}
```
We can make a draft order with this command:

**Note**: You will need to swap `{your_sync_variant_id}` out with your own `sync_variant_id`
```
curl --location --request POST 'https://api.printful.com/orders' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "recipient": {
        "name": "name",
        "address1": "address",
        "city": "city",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
        {
            "sync_variant_id": {your_sync_variant_id},
            "quantity": 1
        }
    ]
}
'
```

### Next steps
To learn more about making an order see:
[Make your first order through the API](#tag/Tutorials/Make-your-first-order-through-the-API).

## Ordering a Direct to Garment (DTG) and an Embroidery Product through the API

### Introduction

In this tutorial, we're going to make an order and design one of Printful's
most popular items, the **Unisex Staple T-Shirt | Bella + Canvas 3001**.
You will learn how to define file placements and product options when ordering DTG and embroidery products.

### Set up

Before anything else make sure you have created a native or "manual order platform store"
if you haven't already.

- Go to this link: https://www.printful.com/dashboard/store
- Click the "Create" button under "Manual order platform/API"
- Choose a name and click continue

As well as this you will need to create an OAuth token for this store.

- Go to the developer portal: https://developers.printful.com/login
- Sign in with your Printful account
- Go to this link: https://developers.printful.com/tokens/add-new-token
- Make sure you have set the access level to "A single store" and select your store
- Make sure you have the following scopes selected for this tutorial:
  -  `orders` ("View and manage orders of the authorized store")
- Fill in all other fields as you please
- Create your token
- You will receive an access key, you will only see the token once, store it securely


To double-check that your token works and that you have the right store selected, try running the following command
replacing `your_token` with the access token created in the developer portal.
```
curl --location --request GET 'https://api.printful.com/stores' \
--header 'Authorization: Bearer {your_token}'
```

You should see your store in the response. If you don't see the right store there, or you get an error,
double-check the token is correct and if necessary create a new token.

### Making the order
We won't go through all the details on how to make an order in this tutorial, instead we will start with the following 
JSON object, which we can use as the basis for constructing an order. 

```
{
    "recipient": {
        "name": "recipients name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
        {
            "variant_id": 4035,
            "quantity": 1
        }
    ]
}
```

The `recipient` object describes where, and to whom, I want to send the order. Inside the `items`
array I have one object with `variant_id` `4035` which is this T-Shirt:

<img alt="variant_id 4035" src="https://s3-printful.stage.printful.dev/products/71/4035_1581408351.jpg" width="300"/>

Described in JSON like this:
```
{
    "id": 4035,
    "product_id": 71,
    "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (Asphalt / 2XL)",
    "size": "2XL",
    "color": "Asphalt",
    "color_code": "#52514f",
    "color_code2": null,
    "image": "https://s3-printful.stage.printful.dev/products/71/4035_1581408351.jpg",
    "price": "12.55",
    "in_stock": true,
    "availability_regions": {
        "US": "United States",
        "EU": "Europe",
        "EU_LV": "Latvia",
        "EU_ES": "Spain",
        "AU": "Australia",
        "CA": "Canada",
        "UK": "United Kingdom"
    },
    "availability_status": [
        {
            "region": "US",
            "status": "in_stock"
        },
        {
            "region": "EU",
            "status": "in_stock"
        },
        {
            "region": "EU_LV",
            "status": "in_stock"
        },
        {
            "region": "EU_ES",
            "status": "stocked_on_demand"
        },
        {
            "region": "AU",
            "status": "in_stock"
        },
        {
            "region": "CA",
            "status": "in_stock"
        },
        {
            "region": "UK",
            "status": "in_stock"
        }
    ]
}
```
If you'd like more detail, we have a tutorial to help you
[make your first order through the API](#tag/Tutorials/Make-your-first-order-through-the-API).

### Adding placements
We already know what we want to order. But now we want to add our design to the image. I want to print the following design
on the right sleeve and the back of the T-Shirt.

![Printful logo](https://printful.com/static/images/layout/logo-printful.png)


First let's make sure that's actually possible, by making a request to the `variants` endpoint.
```
curl --location --request GET 'https://api.printful.com/products/variant/4035' \
--header 'Authorization: Bearer {your_token}'
```
**Note**: You could also do this with a request to the Products endpoint as this information will be the same for each variant of a product.
```
curl --location --request GET 'https://api.printful.com/products/71' \
--header 'Authorization: Bearer {your_token}'
```
You will see what file placements are available under `result.product.files`. As of writing this tutorial the following placements are
available:
```
...
"files": [
  {
      "id": "embroidery_chest_left",
      "type": "embroidery_chest_left",
      "title": "Left chest",
      "additional_price": "2.60"
  },
  {
      "id": "embroidery_large_center",
      "type": "embroidery_large_center",
      "title": "Large center",
      "additional_price": "3.25"
  },
  {
      "id": "embroidery_chest_center",
      "type": "embroidery_chest_center",
      "title": "Center chest",
      "additional_price": "2.60"
  },
  {
      "id": "embroidery_sleeve_left_top",
      "type": "embroidery_sleeve_left_top",
      "title": "Left sleeve top",
      "additional_price": "2.60"
  },
  {
      "id": "embroidery_sleeve_right_top",
      "type": "embroidery_sleeve_right_top",
      "title": "Right sleeve top",
      "additional_price": "2.60"
  },
  {
      "id": "default",
      "type": "front",
      "title": "Front print",
      "additional_price": null
  },
  {
      "id": "front_large",
      "type": "front_large",
      "title": "Large front print",
      "additional_price": "5.25"
  },
  {
      "id": "back",
      "type": "back",
      "title": "Back print",
      "additional_price": "5.25"
  },
  {
      "id": "label_outside",
      "type": "label_outside",
      "title": "Outside label",
      "additional_price": "2.20"
  },
  {
      "id": "label_inside",
      "type": "label_inside",
      "title": "Inside label",
      "additional_price": "2.20"
  },
  {
      "id": "sleeve_left",
      "type": "sleeve_left",
      "title": "Left sleeve",
      "additional_price": "2.20"
  },
  {
      "id": "sleeve_right",
      "type": "sleeve_right",
      "title": "Right sleeve",
      "additional_price": "2.20"
  },
  {
      "id": "preview",
      "type": "mockup",
      "title": "Mockup",
      "additional_price": null
  }
],
...
```

Thankfully "Right sleeve" (`sleeve_right`) and "Back print" (`back`) are among the available options for this product.

```
  ...
  {
      "id": "back",
      "type": "back",
      "title": "Back print",
      "additional_price": "5.25"
  },
  ...
  {
      "id": "sleeve_right",
      "type": "sleeve_right",
      "title": "Right sleeve",
      "additional_price": "2.20"
  },
  ...
```
The most important thing here is the `type` field, that's what we will use to indicate where we want our files to be placed.

### Adding the image placements to the items
Now we can add our designs to the item object. To do that we need a `files` array containing the source of our images.

In our [previous article](#tag/Tutorials/Make-your-first-order-through-the-API) on making orders, we already added a file to an item, 
and it looked like this:
```
{
    "variant_id": 4035,
    "quantity": 1,
    "files": [
        {
            "url": "https://printful.com/static/images/layout/logo-printful.png"
        }
    ]
}
```

This won't work for us though, because it doesn't specify the placement. The above payload
will use the `default` placement because no other was specified. If we use the default the image
will be printed on the `front` for this product.

To have the placement on the back instead, let's add a `type` field to the `file` object
```
{
    "type": "back",
    "url": "https://printful.com/static/images/layout/logo-printful.png"
}
```
Then we can add another object to the `files` array doing the same but with `sleeve_right` as the type so that our final item
looks like this.
```
{
    "variant_id": 4035,
    "quantity": 1,
    "files": [
      { 
        "type": "back",
        "url": "https://printful.com/static/images/layout/logo-printful.png"
      },
      { 
        "type": "sleeve_right",
        "url": "https://printful.com/static/images/layout/logo-printful.png"
      }
    ]
}
```

### Ordering the product
We can already make our order as a draft with this request:

```
curl --location --request POST 'https://api.printful.com/orders' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "recipient": {
        "name": "recipients name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
      {
          "variant_id": 4035,
          "quantity": 1,
          "files": [
            { 
              "type": "back",
              "url": "https://printful.com/static/images/layout/logo-printful.png"
            },
            { 
              "type": "sleeve_right",
              "url": "https://printful.com/static/images/layout/logo-printful.png"
            }
          ]
      }
    ]
}'
```

In the response, you'll find a link to the order in the dashboard.

```
...
"dashboard_url": "https://www.printful.com/dashboard?order_id={your_order_id}"
...
```
In the dashboard, you'll be able to view the order and see how your item looks on a mockup. You may have to wait
for the mockup to be generated.

**Note**: This can be done programmatically through the API as well, using the Mockup Generator API.

### File position
The print file on the `back` placement didn't look how I expected. I wanted the image towards the top and the middle.
So, I will update the order and pass a position object along with the item using a `PUT` request to
the `orders` endpoint.

The position object looks like this:
```
{
    "area_width": 1200,
    "area_height": 1600,
    "width": 300,
    "height": 100,
    "top": 100,
    "left": 450
}
```
This object creates an invisible background for your image, which you can use to place your image in a more specific location.
`area_width` and `area_height` are used to create the area your image will be placed on top of. `width` and `height` can
be used to resize your image. `top` and `left` refer to how far away the image should be placed from the top and left
respectively.

These numbers are relative and don't refer to centimeters, inches, or pixels. So the following `placement` object is the same as the previous one:
```
{
    "area_width": 12,
    "area_height": 16,
    "width": 3,
    "height": 1,
    "top": 1,
    "left": 4.5
}
```

This is the request I make to update the order:
```
curl --location --request PUT 'https://api.printful.com/orders/{your_order_id}' \
--header 'Authorization: Bearer {your_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "items": [
      {
          "variant_id": 4035,
          "quantity": 1,
          "files": [
            { 
              "type": "back",
              "url": "https://printful.com/static/images/layout/logo-printful.png",
              "position": {
                  "area_width": 1200,
                  "area_height": 1600,
                  "width": 300,
                  "height": 100,
                  "top": 100,
                  "left": 450
              }
            },
            { 
              "type": "sleeve_right",
              "url": "https://printful.com/static/images/layout/logo-printful.png"
            }
          ]
      }
    ]
}'
```

### Adding embroidery
Now that I have an order with direct to garment prints made, I'd also like to have my design embroidered.

For the embroidered version of the T-Shirt I'd like the design to be on the chest towards the left,
where a pocket might be on another shirt. Thankfully, we've already seen that top left is an option for this shirt

```
{
      "id": "embroidery_chest_left",
      "type": "embroidery_chest_left",
      "title": "Left chest",
      "additional_price": "2.60"
}
```

So let's try and make our previous `POST` request again, but this time with only the `embroidery_chest_left` file type:

```
curl --location --request POST 'https://api.printful.com/orders' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "recipient": {
        "name": "recipients name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
      {
          "variant_id": 4035,
          "quantity": 1,
          "files": [
            { 
              "type": "embroidery_chest_left",
              "url": "https://printful.com/static/images/layout/logo-printful.png"
            }
          ]
      }
    ]
}'
```
Unfortunately, embroidery items work a bit differently, so if we make that request we'll get this error:
```
{
    "code": 400,
    "result": "Item 0: thread_colors_chest_left option is missing or incorrect! Allowed values: #FFFFFF, #000000, #96A1A8, #A67843, #FFCC00, #E25C27, #CC3366, #CC3333, #660000, #333366, #005397, #3399FF, #6B5294, #01784E, #7BA35A",
    "error": {
        "reason": "BadRequest",
        "message": "Item 0: thread_colors_chest_left option is missing or incorrect! Allowed values: #FFFFFF, #000000, #96A1A8, #A67843, #FFCC00, #E25C27, #CC3366, #CC3333, #660000, #333366, #005397, #3399FF, #6B5294, #01784E, #7BA35A"
    }
}
```

There are only so many colors available when using embroidery, so the `thread_colors` need to be set in the request.
If you don't want to do that manually you can pass the `auto_thread_color` into the files `options` array.

```
"type": "embroidery_chest_left",
"url": "https://printful.com/static/images/layout/logo-printful.png"
"options": [
    {
        "id": "auto_thread_color",
        "value": true
    }
]

```
**Note**: The options are a property of the **file** object not the item

If that option is passed into the request, Printful will choose the colors closest to those in the file.

However, in this case, I'd like more control over the thread colors. I'd like a black and white effect,
so I need to add the option `thread_colors_chest_left` to the item along with the first two allowed values
from the error message.

```
{
  "variant_id": 4035,
  "quantity": 1,
  "files": [
    { 
      "type": "embroidery_chest_left",
      "url": "https://printful.com/static/images/layout/logo-printful.png"
    }
  ],
  "options": [
    {
      "id": "thread_colors_chest_left",
      "value": ["#000000", "#FFFFFF"]
    }
  ]
}
```
**Note**: The options are a property of the **item** object not of the file.

Now to make that order we can make a request like this:
```
curl --location --request POST 'https://api.printful.com/orders' \
--header 'Content-Type: application/json' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "recipient": {
        "name": "recipients name",
        "address1": "12 address avenue, Bankstown",
        "city": "Sydney",
        "state_code": "NSW",
        "country_code": "AU",
        "zip": "2200"
    },
    "items": [
        {
            "variant_id": 4035,
            "quantity": 1,
            "files": [
                {
                    "type": "embroidery_chest_left",
                    "url": "https://printful.com/static/images/layout/logo-printful.png"
                }
            ],
            "options": [
                {
                    "id": "thread_colors_chest_left",
                    "value": ["#000000", "#FFFFFF"]
                }
            ]
        }
    ]
}'
```

### Next steps
We've made these orders, but we haven't confirmed them yet, so they are just drafts. For more information on making orders, you can
look through the documentation [here](#tag/Orders-API) or follow our other tutorial [Make your first order through the API](#tag/Tutorials/Make-your-first-order-through-the-API).

Or if you would like these items to be more reusable as items in your store you can follow our tutorial on [creating products for your store through the API](#tag/Tutorials/Creating-products-for-your-store-through-the-API).

## Creating Mockups in the API
### Introduction
In our previous tutorials we created products that can be ordered by ourselves or our customers.
However, if we're going to order a product, or offer it to a customer, we'll want to know what
it looks like first. That's where mockups come in.

In this tutorial, we're going to create mockups for some of the products we built in previous tutorials.

### Set up

Before anything else make sure you have created a native or "manual order platform store"
if you haven't already.

- Go to this link: https://www.printful.com/dashboard/store
- Click the "Create" button under "Manual order platform/API"
- Choose a name and click create

As well as this you will need to create an OAuth token for this store.

- Go to the developer portal: https://developers.printful.com/login
- Sign in with your Printful account
- Go to this link: https://developers.printful.com/tokens/add-new-token
- Make sure you have set access level to "A single store" and select your store
- Make sure you have the following scopes selected for this tutorial:
  -  `orders` ("View and manage orders of the authorized store")
- Fill in all other fields as you please
- Create your token
- You will receive an access key, you will only see the token once, store it securely

### Deciding which variants to generate
In one of our previous tutorials, [Ordering a Direct to Garment (DTG) and an Embroidery Product through the API](#tag/Tutorials/Ordering-a-Direct-to-Garment-(DTG)-and-an-Embroidery-Product-through-the-API).
Let's make some mockups for that same T-Shirt.

When we originally made the T-Shirt, we only had the `Asphalt` color, but I'd also like to
display a `Black` and `White` variant as well. So, I find the `id` of the T-Shirt product I want,
`71`, and make a `GET` request to `https://api.printful.com/products/71`.

```
curl --location --request GET 'https://api.printful.com/products/71' \
--header 'Authorization: Bearer {your_token}'
```
This will provide you with, among other things, a list of variants. Among those variants,
you will find the Asphalt variant  `4035` we created in the previous tutorial. We will also find
a variant with the color black `4020` and a variant with the color white `4015`.

Now we need to get the files from our previous tutorial and put them into a new files array using the format
of the [Mockup Generator API](#tag/Mockup-Generator-API).

So if we had files from an order that looked like this:
```
  "files": [
    { 
      "type": "embroidery_chest_left",
      "url": "https://printful.com/static/images/layout/logo-printful.png"
      "position": {
        "area_width": 1800,
        "area_height": 2400,
        "width": 1800,
        "height": 1800,
        "top": 300,
        "left": 0
      }
    }
  ]
```

Our new files array will look like this:
``` 
  "files": [
	{ 
		"type": "embroidery_chest_left",
		"image_url": "https://printful.com/static/images/layout/logo-printful.png",
		"position": {
		  "area_width": 1800,
		  "area_height": 2400,
		  "width": 1800,
		  "height": 1800,
		  "top": 300,
		  "left": 0
		}
	}
  ]
```

To generate the mockups for these variants we can put these files and `variant_id`s in
a single `POST` request to `https://api.printful.com/mockup-generator/create-task/{product_id}`.

``` 
curl --location --request POST 'https://api.printful.com/mockup-generator/create-task/71' \
--header 'Authorization: Bearer {your_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "variant_ids": [
        4035,
        4020,
        4015
    ],
  "files": [
    { 
      "type": "embroidery_chest_left",
      "image_url": "https://printful.com/static/images/layout/logo-printful.png",
      "position": {
        "area_width": 1800,
        "area_height": 2400,
        "width": 1800,
        "height": 1800,
        "top": 300,
        "left": 0
      }
    }
  ]
}'
```

This will give you a response like this:
```
{
   "code" : 200,
   "extra" : [],
   "result" : {
      "status" : "pending",
      "task_key" : "gt-421699980"
   }
}
```
You'll notice that there are no mockups in the response, instead, you receive a `task_key`
which you can use to retrieve the actual mockup:

```
curl --location --request GET 'https://api.printful.com/mockup-generator/task?task_key=gt-421699980' \
--header 'Authorization: Bearer {your_token}'
```

```
{
   "code" : 200,
   "extra" : [],
   "result" : {
      "mockups" : [
         {
            "extra" : [
               {
                  "option" : "Front",
                  "option_group" : "Men's",
                  "title" : "Front",
                  "url" : "https://printful-upload.s3-accelerate.amazonaws.com/tmp/ca321525cfd907188c2266fb6a41cf94/unisex-staple-t-shirt-black-front-634d3ec02e655.png"
               },
               {
                  "option" : "Front",
                  "option_group" : "Flat",
                  "title" : "Front",
                  "url" : "https://printful-upload.s3-accelerate.amazonaws.com/tmp/a3560e3a7558374cfdb2bb6c53ccee18/unisex-staple-t-shirt-black-front-634d3ec02e945.png"
               }
            ],
            "mockup_url" : "https://printful-upload.s3-accelerate.amazonaws.com/tmp/e2a9e33986914a13096d60847be8a5b0/unisex-staple-t-shirt-black-zoomed-in-634d3ec02dafc.png",
            "placement" : "embroidery_chest_left",
            "variant_ids" : [
               4020
            ]
         },
         {
            ...
            "variant_ids" : [
               4035
            ]
         },
         {
            ...
            "variant_ids" : [
               4015
            ]
         }
      ],
      "printfiles" : [
         {
            "placement" : "embroidery_chest_left",
            "url" : "https://printful-upload.s3-accelerate.amazonaws.com/tmp/81aabb08df3df89bf67edbd0540fcbe1/printfile_embroidery_chest_left.png",
            "variant_ids" : [
               4020,
               4035,
               4015
            ]
         }
      ],
      "status" : "completed",
      "task_key" : "gt-421709941"
   }
}
```

And that's it, now we have three mockups for each variant, one is an image of the front of the T-Shirt with no model,
and the other two are the front of the T-Shirt with a model.

The only problem is that these default mockups only give me one kind of model or style
To change the style of my mockups I'll need to find the `options` and `option_groups` that will help create different mockup styles.

### Options and Option Groups
#### What are options and option_groups?
In the context of mockup generation `options`, generally speaking, refer to the part of the product being displayed.
`option_groups` refers to the "style" of the mockup. For example "Front" refers to an `option` but "Wrinkled" or "On Hanger"
are `option_groups`.

#### Finding options and option_groups
To find what `options` and `option_groups` are available I make a request to `mockup-generator/printfiles/{product_id}`:

```
curl --location --request GET 'https://api.printful.com/mockup-generator/printfiles/71' \
--header 'Authorization: Bearer {your_token}' 
```
Giving the following response:
```
{
   "code" : 200,
   "extra" : [],
   "result" : {
      "available_placements" : {
	      ...
      },
      "option_groups" : [
            "Couple's",
            "Flat",
            "Flat 2",
            "Flat Lifestyle",
            "Folded",
            "Halloween",
            "Holiday season",
            "Men's",
            "Men's 2",
            "Men's 3",
            "Men's 4",
            "Men's 5",
            "Men's Lifestyle",
            "Men's Lifestyle 2",
            "Men's Lifestyle 3",
            "Men's Lifestyle 4",
            "On Hanger",
            "Product details",
            "Red, white & blue",
            "Spring/summer vibes",
            "Valentine's Day",
            "Women's",
            "Women's 2",
            "Women's 3",
            "Women's 4",
            "Women's 5",
            "Women's Lifestyle",
            "Women's Lifestyle 2",
            "Women's Lifestyle 3",
            "Wrinkled"
      ],
      "options" : [
 	 ...
         "Front",
	 ...
      ],
      "printfiles" : [
         ...
      ],
      "product_id" : 71,
      "variant_printfiles" : [
	...
      ]
   }
}

```

I'd like to have mockups for both genders, so I've already created "Men's" mockups.
So, I can make the request again, specifying that I want the "Women's" `option_group`

```
curl --location --request POST 'https://api.printful.com/mockup-generator/create-task/71' \
--header 'Authorization: Bearer {your_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "variant_ids": [
        4035,
        4020,
        4015
    ],
  "files": [
    { 
      "type": "embroidery_chest_left",
      "image_url": "https://upload.wikimedia.org/wikipedia/commons/0/06/Ercole_de%27_roberti%2C_san_michele_arcangelo_louvre_01.jpg",
      "position": {
	    "area_width": 1800,
		"area_height": 2400,
		"width": 1800,
		"height": 1800,
		"top": 300,
		"left": 0
      }
    }
  ],
  "options": ["Front"],
  "option_groups": ["Women'\''s"]

}'
```

Repeating the steps from before, results in new mockups for each variant, but now with a different model.

If I also want to receive additional styles (`option_groups`) I just need to add it to the option groups.
For example, these option groups will give me mockups for "Men's 2", "Women's" and "Couple's".
```
  "option_groups": ["Men's 2", "Women's", "Couple's"]
```

#### Mismatched options
It is possible to add `options` or `option_groups` that don't produce any mockups.

For example, if I try and make the previous request with `"options": ["Back"]` I will
receive a `400` error:
```
{
   "code" : 400,
   "error" : {
      "message" : "No variants to generate. Option filters may exclude all variants or variants are not available. Alternatively, make sure that variants belong to given product id.",
      "reason" : "BadRequest"
   },
   "result" : "No variants to generate. Option filters may exclude all variants or variants are not available. Alternatively, make sure that variants belong to given product id."
}
```
This is because all the print files are on the front, so Printful can't produce a mockup for the "Back".

But if our request had a print file on the back the request would succeed for. For example:
```
curl --location --request POST 'https://api.printful.com/mockup-generator/create-task/71' \
--header 'Authorization: Bearer {your_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "variant_ids": [
        4035,
        4020,
        4015
    ],
  "files": [
    {
      "type": "back",
      "image_url": "https://upload.wikimedia.org/wikipedia/commons/0/06/Ercole_de%27_roberti%2C_san_michele_arcangelo_louvre_01.jpg",
      "position": {
              "area_width": 1800,
                "area_height": 2400,
                "width": 1800,
                "height": 1800,
                "top": 300,
                "left": 0
      }
    }
  ],
  "options": ["Back"],
  "option_groups": ["Women'\''s"]

}'
```

#### Multiple options and option_groups
It is also possible to have multiple `options`. For example, assuming we have the right variants and print files,
the following combination would produce mockups of the back and front of the product, for "Men's" "Women's".

```
  "options": ["Front", "Back"],
  "option_groups": ["Men's 2", "Women's"]
```
However, you should attempt to limit the number of `options` and `option_groups` you provide here as much as possible.
Otherwise, you may get combinations that don't match resulting in unexpected results that you won't see until after generation.

The following request will not give `400`, instead, it will produce the `"Front"` mockups and ignore the `"Back"`
mockups.

```
curl --location --request POST 'https://api.printful.com/mockup-generator/create-task/71' \
--header 'Authorization: Bearer {your_token}' \
--header 'Content-Type: application/json' \
--data-raw '{
    "variant_ids": [
        ...
    ],
  "files": [
    {
      "type": "embroidery_chest_left",
      ...
    }
  ],
  "options": ["Front", "Back"],
  "option_groups": ["Men'\''s 2", "Women'\''s"]
}'
```
Without the `400` message you may think you are producing mockups that are actually impossible to produce.
If these requests are part of a larger system, it could mean that you end up promising mockups
to your users that are actually impossible.

Additionally, mockup generation can be slow, each new option group will add new mockups
multiplied by the amount of variants and options you have. In a user facing application
generating too many mockups for users might be unacceptably slow.

**It is usually safer to make multiple requests with limited than one large request that attempts to generate a large range of different mockups**

### Next Steps
Now that you've created a basic mockup try and create some mockups for your products.
After that you can save the results somewhere on your server and use your new mockups however you like.

You could also update the `thumbnail` of one of your existing store products.

```
curl --location --request PUT 'https://api.printful.com/store/products/{sync_product_id}' \
--header 'Authorization: Bearer {your_token}' \
--data-raw '{
    "sync_product": {
        "thumbnail": "{url_of_your_new_mockup}"
    }
}'
```

If you don't have any products in your store yet, check out our tutorial on that subject,
[Creating products for your store through the API](#tag/Tutorials/Creating-products-for-your-store-through-the-API).
[Make your first order through the API](#tag/Tutorials/Make-your-first-order-through-the-API).

## Integrating with ECommerce Platforms through the API
### Introduction
In this tutorial, we will show you how to integrate with your ECommerce Sync Products through the API. 
We will retrieve our products, update sync variants, and send out notifications.

Suppose we have a new store that is selling a range of products with a design that changes every year, 
it could be for a conference or maybe for a holiday like Christmas. We have a very large range of products and at the moment our designers have to update all the products manually every year. We want to offer our designers a way to update all of these products at once through the API.


### Set up

Before anything else make sure you have created a store using one of our integrations.
You can choose a platform here: https://www.printful.com/dashboard/store/connect, and 
find additional tutorials to help you set it up.

As well as this you will need to create an OAuth token for this store.

- Go to the developer portal: https://developers.printful.com/login
- Sign in with your Printful account
- Go to this link: https://developers.printful.com/tokens/add-new-token
- Make sure you have set access level to "A single store" and select your store
- Make sure you have the following scopes selected for this tutorial:
  -  `webhooks` ("View and manage store webhooks")
  -  `sync_products` ("View and manage store products")
- Fill in all other fields as you please
- Create your token
- You will receive an access key, you will only see the token once, store it securely

### Limitations
For the most part, the ECommerce API works just like the Products API, but there are a couple of important limitations. 
These limitations are explained here: https://developers.printful.com/docs/#tag/Ecommerce-Platform-Sync-API. But, for the 
purposes of this tutorial, the most important thing is that it’s not possible to create new sync products directly through the API.

### Retrieving Products and Variants
First, let's find the products from last year's Christmas sale

```
curl --location --request GET 'https://api.printful.com/sync/products' \
--header 'Authorization: Bearer {your_token} \ 
```
```
{
    "code": 200,
    "result": [
        {
            "id": 290959495,
            "external_id": "christmas_636b6f729a37c7",
            "name": "Unisex t-shirt | Christmas",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://files.cdn.printful.com/files/thumbnail/christmas.png",
            "is_ignored": false
        },
    ...
    ],
    "extra": [],
    "paging": {
        "total": 143,
        "offset": 0,
        "limit": 20
    }
}
```

Now we will need to loop through these products to find the variants.

```
curl --location --request GET 'https://api.printful.com/sync/products/{id} \
--header 'Authorization: Bearer {your_token} \
```

For that first product we might get a list of variants that looks like this
```
{
    "code": 200,
    "result": {
        "sync_product": {
            "id": 290959495,
            "external_id": "636b6f729a37c7",
            "name": "Unisex t-shirt | Christmas",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://files.cdn.printful.com/files/6bf/6bfceb21e5fa7a854ff915ca66ca3a79_preview.png",
            "is_ignored": false
        },
        "sync_variants": [
            {
                "id": 3590218032,
                "external_id": "636b6f729a3867",
                "sync_product_id": 290959495,
                "name": "Unisex t-shirt | Christmas",
                "synced": true,
                "variant_id": 9575,
                "main_category_id": 24,
                "warehouse_product_id": 330125,
                "warehouse_product_variant_id": 394380,
                "retail_price": "37.25",
                "sku": "636B6F729A302",
                "currency": "EUR",
                "product": {
                    "variant_id": 9575,
                    "product_id": 71,
                    "image": "https://s3-printful.stage.printful.dev/products/71/9575_1581408916.jpg",
                    "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (Black Heather / XS)"
                },
           "files": [
                {
                    "id": 490457488,
                    "type": "default",
                    "hash": null,
                    "url": "https://example.com/file_for_christmas_2022.jpg",
                    "filename": null,
                    "mime_type": null,
                    "size": 0,
                    "width": null,
                    "height": null,
                    "dpi": null,
                    "status": "ok",
                    "created": 1681472128,
                    "thumbnail_url": null,
                    "preview_url": null,
                    "visible": true,
                    "is_temporary": false
                }
            ],

                ],
                "options": [
                    ...
                ],
                "is_ignored": false
            }
        ]
    },
    "extra": []
}
```

### Updating a Sync Variant
As you can see from the request above the image is for 2022, https://example.com/file_for_christmas_2022.jpg,
and we want to update so that we use the new 2023 image. For that all we need to do is send a PUT request to the endpoint

```
curl --location --request PUT 'https://api.printful.com/sync/variant/3590218032' \
--header 'Authorization: Bearer {your_token} \
--data '{
    "files": [
        {
            "type": "default",
            "url": "https://example.com/file_for_christmas_2023.jpg"
        }
    ]
}'
```

This will replace the original 2022 files with the new 2023 files.

Now, rather than changing each product individually the designers could simply 
send you the new design, or use your custom UI. You will then be able to loop 
through all the relevant variants, replacing the old file for each variant.

**Note** when looping through anything within the API remember to keep within the
[rate limits](#section/About-the-Printful-API/Rate-Limits)

### Setting Up a Webhook For Newly Synced Products
Webhooks are another opportunity for the automation of your ECommmerce website through the API. 
Suppose you want to inform your customers everytime your team creates a new Sync Product. This can be automated using webhooks.

First, you will need to listen for webhooks coming from Printful. We can’t help with setting up a server to do that,
but you can use our [webhook simulator](https://www.printful.com/api/webhook-simulator).

The webhook you will be listening for is: [**product_synced**](#operation/productSynced).
To configure the webhook make the following request

```
curl --location -X POST 'https://api.printful.com/webhooks' \
--header 'Authorization: Bearer {your_token} \
--data '{
  "url": "https://your.site/path/to/webhook-listener",
  "types": [
      "product_synced"
  ]
}'
```

Now, whenever a new product is synced to Printful, Printful will POST to the new product to that endpoint in this format:

```
{
  "type": "product_synced",
  "created": 1681473307,
  "retries": 1,
  "store": {your_store_id},
  "data": {
    "sync_product": {
      "id": 4567234,
      "external_id": "christmas_collection-1234",
      "name": "Mug | Christmas",
      "variants": 1,
      "synced": 1
    }
  }
}
```
Now, whenever you receive the webhook you can send updates to your customers about the new product that was just synced.

### Next steps
After updating your products you may also want to [generate mockups through the Mockup Generator API](#tag/Tutorials/Creating-Mockups-in-the-API).

If you have a Native Store, you should check [our tutorial on the Store Products API](#tag/Tutorials/Creating-products-for-your-store-through-the-API).


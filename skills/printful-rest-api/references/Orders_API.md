# Orders API

The Orders API is the most important part of the Printful API - it allows you to create new orders and confirm them for
fulfillment.

**Important**: Jewelry products are not supported via API.

### Order life cycle and statuses

Each order will go through different states while being processed. The following order status types indicate those
states:

<table>
    <tr>
        <td><strong>draft</strong></td>
        <td>The order is created but is not yet submitted for fulfillment. You still can edit it and confirm later.</td>
    </tr>
    <tr>
        <td><strong>inreview</strong></td>
        <td>The order is being reviewed. It's not possible to cancel the order at this point. It will be possible to cancel the order when the review process is finished.</td>
    </tr>
    <tr>
        <td><strong>pending</strong></td>
    <td>The order has been submitted for fulfillment, but is not yet accepted for fulfillment. You can still cancel the order if you need.</td>
    </tr>
    <tr>
        <td><strong>failed</strong></td>
        <td>Order was submitted for fulfillment but was returned for review because of an error (problem with address, missing printfiles, charging has failed, etc.).</td>
    </tr>
    <tr>
        <td><strong>canceled</strong></td>
        <td>The order has been canceled and can no longer be processed. If the order was charged then the amount has been returned to your credit card.</td>
    </tr>
    <tr>
        <td><strong>inprocess</strong></td>
        <td>The order is being fulfilled and can no longer be cancelled or modified. Contact customer support if there are any issues with the order at this point.</td>
    </tr>
    <tr>
        <td><strong>onhold</strong></td>
        <td>The order has encountered a problem during the fulfillment that needs to be resolved together with Printful customer service before fulfillment can continue.</td>
    </tr>
    <tr>
        <td><strong>partial</strong></td>
        <td>The order is partially fulfilled (some items are shipped already, the rest will follow)</td>
    </tr>
    <tr>
        <td><strong>fulfilled</strong></td>
        <td>All items have been shipped successfully</td>
    </tr>
    <tr>
        <td><strong>archived</strong></td>
        <td>The order has been archived and hidden from the UI</td>
    </tr>
</table>

To sum up, the API allows you to create orders with status `draft` and then move them to state `pending` (both steps can
be done with a single action). You are only charged for orders that have been confirmed. If the order encounters a
problem after it has been submitted, then it is moved to the failed state so that the problem can be fixed and the order
can be resubmitted.

### Asynchronous order cost calculation

Most of the times, when you submit an order, we'll perform the cost calculation and return it in the response.

However, we might not be able to calculate all the costs immediately, for example if the order contains a new advanced
embroidery design. If that's the case, we'll automatically put your order on hold, calculate the order costs once it's
possible, and then remove the order from hold.

Such an order will return to a draft status (even if it was created with the auto-confirm option) and will need to be
confirmed.

You can subscribe to the `order_remove_hold` event (see [Webhook API](#tag/Webhook-API)) to be notified when the order is removed from hold.

### External ID

External ID is an optional feature that allows you to link your Printful order with the Order ID from your system
without the need to store additional data on your side. External ID can be up to 32 characters long and contain digits,
Latin alphabet letters, dashes and underscores, however it is recommended to use integer numbers. Each order's External
ID must be unique within the store.

To use the External ID feature, you just add the `external_id` attribute when creating the order. Later, when you need
to access the order through the API, you can reference it by both the Order ID and by External ID (if you prefix it with
the `@` symbol).

```
GET /orders/11001  - reference by Printful Order ID
GET /orders/@988123  - reference by External ID
GET /orders/@AA123123  - reference by External ID
```

You can assign the `external_id` attribute to line items as well. In this case they have to be unique per order.

### Specifying products

There are three general ways to specify a product’s variant when creating, updating or estimating an order:

(A) **Using an existing product variant (sync variant) in your Printful store or warehouse.** To specify the existing
product please use its `sync_variant_id` or `external_variant_id`, or `warehouse_product_variant_id`.

[Example using Sync Variant ID](#tag/Examples/Orders-API-examples/Using-a-sync-variant)
[Example using External Variant ID](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID)

(B) **Using a Catalog API variant without adding a product to the store.** This method can be used when a Printful store
has no products in it. To construct a variant on-the-fly retrieve a specific `variant_id` from the
[Catalog API](#tag/Catalog-API) together with print files and an additional options.

[Example](#tag/Examples/Orders-API-examples/Using-a-catalog-variant)

(C) **Using an existing template ID.** This method can be used when a Printful store has assigned templates without the
need to create products. To create an order please use the `product_template_id` and `variant_id` that will be added to
the order.

[Example](#tag/Examples/Orders-API-examples/Using-a-product-template)

### Adding print files

There are two ways to assign a print file to the item. One is to specify the File ID if the file already exists in the
file library of the authorized store:

```
...
"files": [
    {
        "id": 12345
    },
],
...
```

The second and the most convenient method is to specify the file URL. If a file with the same URL already exists, it will be reused.

```
...
"files": [
    {
        "url": "http://example.com/t-shirts/123/front.pdf"
    },
],
...
```

### Specifying file position

You can specify the image position inside the print area by providing a position object.

<strong>Important</strong><br>
* Each print area has specific dimensions, by default Orders API will assume that your file has to stick to those limitations and not exceed them. In some cases you would want to position your file outside the print area - to be able to do so use the `limit_to_print_area` and set it to: `false`.
* `limit_to_print_area` determines if the image can cross the print area border. If `limit_to_print_area` is set to `true` then the request will result in `400 Bad Request` with "Invalid position" in `error.message` once the image crosses the print area borders. If `limit_to_print_area` is set to `false` then it will be possible to place image partially or fully outside the print area.
* The (0,0) point is always located in top left corner of the print area.

<strong>Steps</strong><br>
1.Retrieve printfile dimensions [Printfiles](#operation/getPrintfiles)
```
...
"printfiles":
    [
        {
            "printfile_id": 1,
            "width": 1800,
            "height": 2400,
            "dpi": 150,
            "fill_mode": "fit",
            "can_rotate": false
        }
    ],
...
```
2.Specify file position for specific print placement while creating an order. Use `items` -> `files` -> `position` object as in the example:
```
...
"items": [
    {
        "variant_id":4011,
        "quantity":"1",
        "files": [
            {
                "type": "front",
                "url": "http://example.com/t-shirts/123/front.pdf",
                "position": {
                    "area_width": 1800,
                    "area_height": 2400,
                    "width": 1800,
                    "height": 1800,
                    "top": 300,
                    "left": 0,
                    "limit_to_print_area": true
                }
            }
        ]
    }
]
...
```

#### Example of positioning the 450x450 image on the front placement

<table>
  <tr>
    <td> Position </td> 
    <td> Mockup </td>
    <td> Payload </td>
  </tr>
  <tr>
<td> Top left </td>
<td> <img alt="Top left mockup" src="images/position/top_left.png" width="300"/> </td>
<td>

```
  "position": {
  "area_width": 1800,
  "area_height": 2400,
  "width": 450,
  "height": 450,
  "top": 0,
  "left": 0,
  "limit_to_print_area": true
  }
```

</td>
</tr>

<tr>
<td> Top middle </td>
<td> <img alt="Top left mockup" src="images/position/top_middle.png" width="300"/> </td>
<td>

```
"position": {
"area_width": 1800,
"area_height": 2400,
"width": 450,
"height": 450,
"top": 0,
"left": 675,
"limit_to_print_area": true
}
```

</td>
</tr>

<tr>
<td> Top right </td>
<td> <img alt="Top left mockup" src="images/position/top_right.png" width="300"/> </td>
<td>

```
"position": {
"area_width": 1800,
"area_height": 2400,
"width": 450,
"height": 450,
"top": 0,
"left": 1350,
"limit_to_print_area": true
}
```

</td>
</tr>

<tr>
<td> Middle </td>
<td> <img alt="Top left mockup" src="images/position/middle.png" width="300"/> </td>
<td>

```
"position": {
"area_width": 1800,
"area_height": 2400,
"width": 450,
"height": 450,
"top": 975,
"left": 675,
"limit_to_print_area": true
}
```

</td>
</tr>

<tr>
<td> Bottom left </td>
<td> <img alt="Top left mockup" src="images/position/bottom_left.png" width="300"/> </td>
<td>

```
"position": {
"area_width": 1800,
"area_height": 2400,
"width": 450,
"height": 450,
"top": 1950,
"left": 0,
"limit_to_print_area": true
}
```

</td>
</tr>

<tr>
<td> Bottom middle </td>
<td> <img alt="Top left mockup" src="images/position/bottom_middle.png" width="300"/> </td>
<td>

```
"position": {
"area_width": 1800,
"area_height": 2400,
"width": 450,
"height": 450,
"top": 1950,
"left": 675,
"limit_to_print_area": true
}
```

</td>
</tr>

<tr>
<td> Bottom right </td>
<td> <img alt="Top left mockup" src="images/position/bottom_right.png" width="300"/> </td>
<td>

```
"position": {
"area_width": 1800,
"area_height": 2400,
"width": 450,
"height": 450,
"top": 1950,
"left": 1350,
"limit_to_print_area": true
}
```

</td>
</tr>

</table>

### Specifying multiple files per item

Each item in the order has to be linked with one or multiple files. The available file types for each product are
available from the [Catalog API](#tag/Catalog-API).

You can add one file for each type by specifying the `type` attribute. For the `default` type, this attribute can be
skipped.

```
...
"files":[
	{
		"type": "default",
		"url": "http://example.com/t-shirts/123/front.pdf"
	},
	{
		"type": "back"
		"url": "http://example.com/t-shirts/123/back.pdf"
	},
	{
		"type": "preview"
		"url": "http://example.com/t-shirts/123/preview.png"
	}
],
...
```

Remember that using additional files can increase the price of the item.

### Creating orders from a template

Orders API allows also creating orders based on the product template created in the Printful account without the need to
add the product to the Printful store.

To retrieve available templates for your account please use the
[Products Templates API](#tag/Product-Templates-API).

To create an order from a template you need to specify a variant or variants that will be added to the order. It is
possible to use multiple templates with different variants in one request. To achieve that please use the `items` object
below:

```
    ...
    "items": [
        {
            "variant_id": 4012,
            "quantity": 1,
            "product_template_id": 123456789
        },
        {
            "variant_id": 1,
            "quantity": 2,
            "product_template_id": 11235813
        },
    ]
    ...
```

**Important note**: you can only create orders from templates for variant IDs from the Catalog API.

More examples are available [here](#tag/Examples/Orders-API-examples/Using-a-product-template).

### Retail costs

Printful allows you to specify your retail costs for the order so that the packing slip for international orders can
contain your correct retail prices. To enable retail costs, each item in the order has to contain the `retail_price`
attribute. You can also specify a custom discount sum, shipping costs and taxes in the `retail_costs` object when
creating the order. If the retail costs are missing, the packing slip will contain the Printful prices instead.

### Native inside label

Printful previously allowed customers to upload a fully customized inside label. Since these labels had to contain
specific information about fabric composition, manufacturing, etc. to meet the legal requirements, users usually
encountered issues to get their labels printed.

Inside labels are printed on the inside of the garment and require the removal of the original manufacturer's tag.
They're only available for apparel with tear-away labels. An inside label must include the country of manufacturing
origin, original garment size, and material information. To use our native label template you only need to upload a
graphic (such as your brand's logo). The mandatory content will be generated and placed automatically.

```
...
"files":[
        {
            "type": "label_inside",
            "url": "http://example.com/logo/123/image.jpg",
            "options": [{
                "id": "template_type",
                "value": "native"
            }]
        },
],
...
```

Printful previously supported fully customized inside labels. These have now been deprecated. The ability to create orders with fully customized inside labels has been limited to only users who were actively using them in their stores before April 2020. This feature is no longer accessible to new users.

### Ordering embroidery products

Embroidery is a technique which uses colored threads, sewn into a product, to recreate provided design. In order to use embroidery technique you first need to check if selected product support embroidery technique.

In order to do that you need to use [Catalog API](#tag/Catalog-API) to determine if the selected product or variant contains `EMBROIDERY` technique.
```
"techniques": [
                {
                    "key": "EMBROIDERY",
                    "display_name": "Embroidery",
                    "is_default": true
                }
            ]
```
After that you need to also get list of available embroidery placements. Those are listed under `file` property with `embroidery_` prefix. You can get list of all available placements in [Placements](#tag/Common/Placements).

<details>
    <summary>Example of file property</summary>

```
"files": [
                {
                    "id": "default",
                    "type": "embroidery_front",
                    "title": "Front",
                    "additional_price": null
                },
                {
                    "id": "back",
                    "type": "embroidery_back",
                    "title": "Back",
                    "additional_price": "3.75"
                },
                {
                    "id": "left",
                    "type": "embroidery_left",
                    "title": "Left side",
                    "additional_price": "3.75"
                },
                {
                    "id": "right",
                    "type": "embroidery_right",
                    "title": "Right side",
                    "additional_price": "3.75"
                },
                {
                    "id": "preview",
                    "type": "mockup",
                    "title": "Mockup",
                    "additional_price": null
                }
            ]
```

</details>

To create an order using embroidery technique you can:

- Provide thread colors manually [See example](#tag/Common/Embroidery/Manually-defining-thread-colors)
- Use automatic thread color detection [See example](#tag/Common/Embroidery/Automatic-thread-color)

Finally, you can make an order using embroidery
technique [See example](#tag/Examples/Orders-API-examples/Using-embroidery-products). Depending on the placement that you've
used you need to specify the correct [thread color option](#tag/Common/Options).

### Packing slip

The packing slip fields can be configured at the store level and overridden for a specific order.

The packing slip settings can be found in Dashboard at **Settings > Stores > Branding > Packing slip section**.

To override the packing slip settings for the order, you can use `packing_slip` or `gift` fields.

Below you can find an example or a packing slip for a shipment with explained fields.

![packing slip](images/sample_packing_slip.png)

Field annotations:

* **(1)** Barcode unique for the shipment.
* **(2)** Store logo defined in the store settings or overridden using `packing_slip.logo_url` field. The provided image is converted to a grayscale/1-bit monochrome image.
* **(3)** The date of the shipment.
* **(4)** Packing slip number consisting of order and shipment IDs in Printful database, divided with a hyphen.
* **(5)** The country from which the shipment is made. If the recipient is in the United States, this field will be
  absent.
* **(6)** Recipient address with phone, without email address.
* **(7)** Store name. This can be overridden using `packing_slip.store_name`.
* **(8)** The address to which the shipment should be returned. By default it will be a Printful’s return address, but
  you can set your own address in the store settings (**Settings > Stores > Returns > Return address section**).
* **(9)** The customer service phone number defined in the store settings or overridden using `packing_slip.phone`
  field.
* **(10)** The customer service email address defined in the store settings or overridden using `packing_slip.email`
  field.
* **(11)** Gift message. This is only present if the `gift` field was provided in the order request.
* **(12)** The order creation date.
* **(13)** Printful Order ID which can be overridden using `packing_slip.custom_order_id` field.
* **(14)** The list of order items with quantities. The items’ display names are localized, using the recipient’s
  country and include variant information such as color and size e.g. „Unisex Staple T-Shirt | Bella + Canvas 3001 (
  Lilac / M)”.
* **(15)** The packing slip message defined in the store settings or overridden using `packing_slip.message` field.

### More Orders API examples

See the [examples section](#tag/Examples/Orders-API-examples) for more sample requests on how Orders API can be used in
different scenarios.

### Custom border color option

Stickers can have a different border color which can be set by using the `thread_colors_outline` option.
This option is available in `options` for stickers. To showcase the usage we will use the order flow
which will create order with a sticker that will have a red border color:

Endpoint `POST https://api.printful.com/orders`
<details>
    <summary>Request body</summary>

```
{
    "shipping": "STANDARD",
    "recipient": {
        "name": "John Smith",
        "address1": "19749 Dearborn St",
        "city": "Chatsworth",
        "country_code": "US",
        "state_code": "CA",
        "zip": "91311"
    },
    "items": [
        {
            "variant_id": 10163,
            "files": [
                {
                    "type": "default",
                    "url": "https://www.printful.com/static/images/layout/printful-logo.png"
                }
            ],
            "options": [
                {
                    "id": "custom_border_color",
                    "value": "#FF0000"
                }
            ]
        }
    ]
}
```

</details>

## Endpoints

### GET /orders
**Summary:** Get list of orders

Returns list of order objects from your store

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getOrders",
  "security": [
    {
      "OAuth": [
        "orders/read"
      ]
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "status",
      "schema": {
        "type": "string"
      },
      "required": false,
      "description": "Filter by order status"
    },
    {
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer"
      },
      "required": false,
      "description": "Result set offset"
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer"
      },
      "required": false,
      "description": "Number of items per page (max 100)"
    }
  ],
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "allOf": [
                  {
                    "type": "object",
                    "properties": {
                      "code": {
                        "type": "integer",
                        "description": "Response status code `200`",
                        "example": 200
                      }
                    }
                  },
                  {
                    "properties": {
                      "paging": {
                        "type": "object",
                        "title": "Paging",
                        "description": "Paging information",
                        "required": [
                          "total",
                          "offset",
                          "limit"
                        ],
                        "properties": {
                          "total": {
                            "type": "integer",
                            "example": 100,
                            "description": "Total number of items available"
                          },
                          "offset": {
                            "type": "integer",
                            "example": 10,
                            "description": "Current result set page offset"
                          },
                          "limit": {
                            "type": "integer",
                            "description": "Max number of items per page",
                            "example": 100
                          }
                        }
                      }
                    }
                  }
                ]
              },
              {
                "properties": {
                  "result": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "title": "Order",
                      "description": "Information about the Order",
                      "required": [
                        "recipient",
                        "items"
                      ],
                      "properties": {
                        "id": {
                          "description": "Order ID",
                          "type": "integer",
                          "example": 13,
                          "readOnly": true
                        },
                        "external_id": {
                          "description": "Order ID from the external system",
                          "type": "string",
                          "nullable": true,
                          "example": "4235234213"
                        },
                        "store": {
                          "description": "Store ID",
                          "type": "integer",
                          "example": 10,
                          "readOnly": true
                        },
                        "status": {
                          "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
                          "type": "string",
                          "example": "draft",
                          "readOnly": true
                        },
                        "shipping": {
                          "description": "Shipping method. Defaults to 'STANDARD'",
                          "type": "string",
                          "example": "STANDARD"
                        },
                        "shipping_service_name": {
                          "description": "Human readable shipping method name.",
                          "type": "string",
                          "example": "Flat Rate (3-4 business days after fulfillment)",
                          "readOnly": true
                        },
                        "created": {
                          "description": "Time when the order was created",
                          "type": "integer",
                          "example": 1602607640,
                          "readOnly": true
                        },
                        "updated": {
                          "description": "Time when the order was updated",
                          "type": "integer",
                          "example": 1602607640,
                          "readOnly": true
                        },
                        "recipient": {
                          "type": "object",
                          "title": "Address",
                          "description": "Information about the address",
                          "properties": {
                            "name": {
                              "description": "Full name",
                              "type": "string",
                              "example": "John Smith"
                            },
                            "company": {
                              "description": "Company name",
                              "type": "string",
                              "example": "John Smith Inc"
                            },
                            "address1": {
                              "description": "Address line 1",
                              "type": "string",
                              "example": "19749 Dearborn St"
                            },
                            "address2": {
                              "description": "Address line 2",
                              "type": "string"
                            },
                            "city": {
                              "description": "City",
                              "type": "string",
                              "example": "Chatsworth"
                            },
                            "state_code": {
                              "description": "State code",
                              "type": "string",
                              "example": "CA"
                            },
                            "state_name": {
                              "description": "State name",
                              "type": "string",
                              "example": "California"
                            },
                            "country_code": {
                              "description": "Country code",
                              "type": "string",
                              "example": "US"
                            },
                            "country_name": {
                              "description": "Country name",
                              "type": "string",
                              "example": "United States"
                            },
                            "zip": {
                              "description": "ZIP/Postal code",
                              "type": "string",
                              "example": "91311"
                            },
                            "phone": {
                              "description": "Phone number",
                              "example": "2312322334",
                              "type": "string"
                            },
                            "email": {
                              "description": "Email address",
                              "example": "firstname.secondname@domain.com",
                              "type": "string"
                            },
                            "tax_number": {
                              "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                              "type": "string",
                              "example": "123.456.789-10"
                            }
                          }
                        },
                        "items": {
                          "type": "array",
                          "description": "Array of items in the order",
                          "items": {
                            "type": "object",
                            "title": "Item",
                            "description": "Information about an item in the order",
                            "properties": {
                              "id": {
                                "description": "Line item ID",
                                "type": "integer",
                                "example": 1
                              },
                              "external_id": {
                                "description": "Line item ID from the external system",
                                "type": "string",
                                "example": "item-1"
                              },
                              "variant_id": {
                                "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                                "type": "integer",
                                "example": 1
                              },
                              "sync_variant_id": {
                                "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                                "type": "integer",
                                "example": 1
                              },
                              "external_variant_id": {
                                "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                                "type": "string",
                                "example": "variant-1"
                              },
                              "warehouse_product_variant_id": {
                                "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                                "type": "integer",
                                "example": 1
                              },
                              "product_template_id": {
                                "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                                "type": "integer",
                                "example": 1
                              },
                              "external_product_id": {
                                "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                                "type": "string",
                                "example": "template-123",
                                "writeOnly": true
                              },
                              "quantity": {
                                "description": "Number of items ordered (Limited to 1000 for one item)",
                                "type": "integer",
                                "example": 1
                              },
                              "price": {
                                "description": "Printful price of the item",
                                "type": "string",
                                "example": "13.00"
                              },
                              "retail_price": {
                                "description": "Original retail price of the item to be displayed on the packing slip",
                                "type": "string",
                                "example": "13.00"
                              },
                              "name": {
                                "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                                "type": "string",
                                "example": "Enhanced Matte Paper Poster 18\u00d724"
                              },
                              "product": {
                                "type": "object",
                                "title": "ProductVariant",
                                "description": "Short information about the Printful Product and Variant",
                                "properties": {
                                  "variant_id": {
                                    "type": "integer",
                                    "description": "Variant ID",
                                    "example": 3001
                                  },
                                  "product_id": {
                                    "type": "integer",
                                    "description": "Product ID of this variant",
                                    "example": 301
                                  },
                                  "image": {
                                    "type": "string",
                                    "description": "URL of a sample image for this variant",
                                    "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                                  },
                                  "name": {
                                    "type": "string",
                                    "description": "Display name of this variant",
                                    "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                                  }
                                }
                              },
                              "files": {
                                "type": "array",
                                "items": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "File",
                                      "description": "Information about the File",
                                      "required": [
                                        "url"
                                      ],
                                      "properties": {
                                        "type": {
                                          "type": "string",
                                          "description": "Role of the file",
                                          "example": "default"
                                        },
                                        "id": {
                                          "type": "integer",
                                          "description": "File ID",
                                          "example": 10,
                                          "readOnly": true
                                        },
                                        "url": {
                                          "type": "string",
                                          "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                                          "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                                        },
                                        "options": {
                                          "type": "array",
                                          "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                                          "items": {
                                            "type": "object",
                                            "title": "FileOption",
                                            "description": "File option",
                                            "required": [
                                              "id",
                                              "value"
                                            ],
                                            "properties": {
                                              "id": {
                                                "type": "string",
                                                "description": "Option id",
                                                "example": "template_type"
                                              },
                                              "value": {
                                                "type": "string",
                                                "description": "Option value",
                                                "example": "native"
                                              }
                                            }
                                          }
                                        },
                                        "hash": {
                                          "type": "string",
                                          "description": "MD5 checksum of the file",
                                          "example": "ea44330b887dfec278dbc4626a759547",
                                          "readOnly": true
                                        },
                                        "filename": {
                                          "type": "string",
                                          "description": "File name",
                                          "example": "shirt1.png"
                                        },
                                        "mime_type": {
                                          "type": "string",
                                          "description": "MIME type of the file",
                                          "example": "image/png",
                                          "readOnly": true
                                        },
                                        "size": {
                                          "type": "integer",
                                          "description": "Size in bytes",
                                          "example": 45582633,
                                          "readOnly": true
                                        },
                                        "width": {
                                          "type": "integer",
                                          "description": "Width in pixels",
                                          "example": 1000,
                                          "readOnly": true
                                        },
                                        "height": {
                                          "type": "integer",
                                          "description": "Height in pixels",
                                          "example": 1000,
                                          "readOnly": true
                                        },
                                        "dpi": {
                                          "type": "integer",
                                          "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                                          "example": 300,
                                          "readOnly": true
                                        },
                                        "status": {
                                          "type": "string",
                                          "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                                          "example": "ok",
                                          "readOnly": true
                                        },
                                        "created": {
                                          "type": "integer",
                                          "description": "File creation timestamp",
                                          "example": 1590051937,
                                          "readOnly": true
                                        },
                                        "thumbnail_url": {
                                          "type": "string",
                                          "description": "Small thumbnail URL",
                                          "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                          "readOnly": true
                                        },
                                        "preview_url": {
                                          "type": "string",
                                          "description": "Medium preview image URL",
                                          "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                          "readOnly": true
                                        },
                                        "visible": {
                                          "type": "boolean",
                                          "description": "Show file in the Printfile Library (default true)",
                                          "example": true
                                        },
                                        "is_temporary": {
                                          "type": "boolean",
                                          "readOnly": true,
                                          "description": "Whether it is a temporary printfile.",
                                          "example": false
                                        }
                                      }
                                    },
                                    {
                                      "type": "object",
                                      "properties": {
                                        "position": {
                                          "writeOnly": true,
                                          "allOf": [
                                            {
                                              "type": "object",
                                              "title": "GenerationTaskFilePosition",
                                              "description": "Position",
                                              "properties": {
                                                "area_width": {
                                                  "type": "integer",
                                                  "description": "Positioning area width on print area in pixels",
                                                  "example": 1800,
                                                  "nullable": true
                                                },
                                                "area_height": {
                                                  "type": "integer",
                                                  "description": "Positioning area height on print area in pixels",
                                                  "example": 2400,
                                                  "nullable": true
                                                },
                                                "width": {
                                                  "type": "integer",
                                                  "description": "Width of the image in given area in pixels",
                                                  "example": 1800,
                                                  "readOnly": false
                                                },
                                                "height": {
                                                  "type": "integer",
                                                  "description": "Height of the image in given area in pixels",
                                                  "example": 1800,
                                                  "readOnly": false
                                                },
                                                "top": {
                                                  "type": "integer",
                                                  "description": "Image top offset in given area in pixels",
                                                  "example": 300
                                                },
                                                "left": {
                                                  "type": "integer",
                                                  "description": "Image left offset in given area in pixels",
                                                  "example": 0
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "properties": {
                                                "limit_to_print_area": {
                                                  "type": "boolean",
                                                  "default": true,
                                                  "description": "Should limit printfile to print area"
                                                }
                                              }
                                            }
                                          ]
                                        }
                                      }
                                    }
                                  ]
                                },
                                "description": "Array of attached printfiles / preview images"
                              },
                              "options": {
                                "type": "array",
                                "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                                "items": {
                                  "type": "object",
                                  "title": "ItemOption",
                                  "description": "Additional option for order item",
                                  "properties": {
                                    "id": {
                                      "type": "string",
                                      "description": "Option ID",
                                      "example": "OptionKey"
                                    },
                                    "value": {
                                      "type": "string",
                                      "description": "Option value",
                                      "example": "OptionValue"
                                    }
                                  }
                                }
                              },
                              "sku": {
                                "description": "Product identifier (SKU) from the external system",
                                "type": "string",
                                "example": null
                              },
                              "discontinued": {
                                "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                                "type": "boolean",
                                "example": true
                              },
                              "out_of_stock": {
                                "description": "Whether the item is out of stock i.e. temporarily unavailable",
                                "type": "boolean",
                                "example": true
                              }
                            },
                            "anyOf": [
                              {
                                "required": [
                                  "variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "variant_id",
                                  "product_template_id"
                                ]
                              },
                              {
                                "required": [
                                  "variant_id",
                                  "external_product_id"
                                ]
                              }
                            ],
                            "not": {
                              "anyOf": [
                                {
                                  "required": [
                                    "product_template_id",
                                    "sync_variant_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "product_template_id",
                                    "external_variant_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "product_template_id",
                                    "warehouse_product_variant_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "product_template_id",
                                    "files"
                                  ]
                                },
                                {
                                  "required": [
                                    "product_template_id",
                                    "options"
                                  ]
                                },
                                {
                                  "required": [
                                    "product_template_id",
                                    "external_product_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "external_product_id",
                                    "sync_variant_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "external_product_id",
                                    "external_variant_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "external_product_id",
                                    "warehouse_product_variant_id"
                                  ]
                                },
                                {
                                  "required": [
                                    "external_product_id",
                                    "files"
                                  ]
                                },
                                {
                                  "required": [
                                    "external_product_id",
                                    "options"
                                  ]
                                },
                                {
                                  "required": [
                                    "external_product_id",
                                    "product_template_id"
                                  ]
                                }
                              ]
                            }
                          }
                        },
                        "branding_items": {
                          "type": "array",
                          "description": "Array of branding items in the order",
                          "readOnly": true,
                          "items": {
                            "$ref": "circular reference to #/components/schemas/Item"
                          }
                        },
                        "incomplete_items": {
                          "type": "array",
                          "description": "Array of incomplete items in the order",
                          "readOnly": true,
                          "items": {
                            "type": "object",
                            "title": "IncompleteItem",
                            "description": "Information about an incomplete item in the order",
                            "properties": {
                              "name": {
                                "type": "string",
                                "description": "Incomplete item name",
                                "example": "Red T-Shirt"
                              },
                              "quantity": {
                                "description": "Incompleted item quantity",
                                "type": "integer",
                                "example": 1
                              },
                              "sync_variant_id": {
                                "description": "Sync variant ID of the incompleted item.",
                                "type": "integer",
                                "example": 70
                              },
                              "external_variant_id": {
                                "description": "External variant ID of the incompleted item.",
                                "type": "string",
                                "example": "external-id"
                              },
                              "external_line_item_id": {
                                "description": "External order line item id.",
                                "type": "string",
                                "example": "external-line-item-id"
                              }
                            }
                          }
                        },
                        "costs": {
                          "allOf": [
                            {
                              "type": "object",
                              "title": "OrderCosts",
                              "description": "Order costs (Printful prices)",
                              "properties": {
                                "currency": {
                                  "type": "string",
                                  "description": "3 letter currency code",
                                  "example": "USD"
                                },
                                "subtotal": {
                                  "type": "string",
                                  "description": "Total cost of all items",
                                  "example": "10.00"
                                },
                                "discount": {
                                  "type": "string",
                                  "description": "Discount sum",
                                  "example": "0.00"
                                },
                                "shipping": {
                                  "type": "string",
                                  "description": "Shipping costs",
                                  "example": "5.00"
                                },
                                "digitization": {
                                  "type": "string",
                                  "description": "Digitization costs",
                                  "example": "0.00"
                                },
                                "additional_fee": {
                                  "type": "string",
                                  "description": "Additional fee for custom product",
                                  "example": "0.00"
                                },
                                "fulfillment_fee": {
                                  "type": "string",
                                  "description": "Custom product fulfillment fee",
                                  "example": "0.00"
                                },
                                "retail_delivery_fee": {
                                  "type": "string",
                                  "description": "Retail delivery fee",
                                  "example": "0.00"
                                },
                                "tax": {
                                  "type": "string",
                                  "description": "Sum of taxes (not included in the item price)",
                                  "example": "0.00"
                                },
                                "vat": {
                                  "type": "string",
                                  "description": "Sum of vat (not included in the item price)",
                                  "example": "0.00"
                                },
                                "total": {
                                  "type": "string",
                                  "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                                  "example": "15.00"
                                }
                              }
                            },
                            {
                              "readOnly": true
                            }
                          ]
                        },
                        "retail_costs": {
                          "type": "object",
                          "title": "OrderRetailCosts",
                          "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                          "properties": {
                            "currency": {
                              "type": "string",
                              "description": "3 letter currency code",
                              "example": "USD"
                            },
                            "subtotal": {
                              "type": "string",
                              "nullable": true,
                              "description": "Total cost of all items",
                              "example": "10.00"
                            },
                            "discount": {
                              "type": "string",
                              "nullable": true,
                              "description": "Discount sum",
                              "example": "0.00"
                            },
                            "shipping": {
                              "type": "string",
                              "nullable": true,
                              "description": "Shipping costs",
                              "example": "5.00"
                            },
                            "tax": {
                              "type": "string",
                              "nullable": true,
                              "description": "Sum of taxes (not included in the item price)",
                              "example": "0.00"
                            },
                            "vat": {
                              "type": "string",
                              "nullable": true,
                              "readOnly": true,
                              "description": "Sum of VAT (not included in the item price)",
                              "example": "0.00"
                            },
                            "total": {
                              "type": "string",
                              "nullable": true,
                              "readOnly": true,
                              "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                              "example": "15.00"
                            }
                          }
                        },
                        "pricing_breakdown": {
                          "type": "array",
                          "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                          "readOnly": true,
                          "items": {
                            "type": "object",
                            "title": "PricingBreakdown",
                            "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                            "properties": {
                              "customer_pays": {
                                "type": "string",
                                "description": "Amount customer paid",
                                "example": "3.75"
                              },
                              "printful_price": {
                                "type": "string",
                                "description": "Printful price",
                                "example": "6"
                              },
                              "profit": {
                                "type": "string",
                                "description": "Profit",
                                "example": "-2.25"
                              },
                              "currency_symbol": {
                                "type": "string",
                                "description": "Shipment tracking number",
                                "example": "USD"
                              }
                            }
                          }
                        },
                        "shipments": {
                          "type": "array",
                          "description": "Array of shipments already shipped for this order",
                          "readOnly": true,
                          "items": {
                            "type": "object",
                            "title": "OrderShipment",
                            "description": "Information about order shipment",
                            "properties": {
                              "id": {
                                "type": "integer",
                                "description": "Shipment ID",
                                "example": 10
                              },
                              "carrier": {
                                "type": "string",
                                "description": "Carrier name",
                                "example": "FEDEX"
                              },
                              "service": {
                                "type": "string",
                                "description": "Delivery service name",
                                "example": "FedEx SmartPost"
                              },
                              "tracking_number": {
                                "type": "string",
                                "description": "Shipment tracking number",
                                "example": 0
                              },
                              "tracking_url": {
                                "type": "string",
                                "description": "Shipment tracking URL",
                                "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                              },
                              "created": {
                                "type": "integer",
                                "description": "Shipping time",
                                "example": 1588716060
                              },
                              "ship_date": {
                                "type": "string",
                                "description": "Ship date",
                                "example": "2020-05-05"
                              },
                              "shipped_at": {
                                "type": "string",
                                "description": "Ship time in unix timestamp",
                                "example": 1588716060
                              },
                              "reshipment": {
                                "type": "boolean",
                                "description": "Whether this is a reshipment",
                                "example": false
                              },
                              "items": {
                                "type": "array",
                                "description": "Array of items in this shipment",
                                "items": {
                                  "type": "object",
                                  "title": "OrderShipmentItem",
                                  "description": "Information about item in an order shipment",
                                  "properties": {
                                    "item_id": {
                                      "type": "integer",
                                      "description": "Line item ID",
                                      "example": 1
                                    },
                                    "quantity": {
                                      "type": "integer",
                                      "description": "Quantity of items in this shipment",
                                      "example": 1
                                    },
                                    "picked": {
                                      "type": "integer",
                                      "enum": [
                                        0,
                                        1
                                      ],
                                      "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                                      "example": 1
                                    },
                                    "printed": {
                                      "type": "integer",
                                      "enum": [
                                        0,
                                        1
                                      ],
                                      "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                                      "example": 1
                                    }
                                  }
                                }
                              }
                            }
                          }
                        },
                        "gift": {
                          "type": "object",
                          "title": "OrderGift",
                          "description": "Optional gift message for the packing slip",
                          "properties": {
                            "subject": {
                              "type": "string",
                              "maxLength": 200,
                              "description": "Gift message title",
                              "example": "To John"
                            },
                            "message": {
                              "type": "string",
                              "maxLength": 200,
                              "description": "Gift message text",
                              "example": "Have a nice day"
                            }
                          }
                        },
                        "packing_slip": {
                          "type": "object",
                          "title": "OrderPackingSlip",
                          "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                          "anyOf": [
                            {
                              "required": [
                                "email"
                              ]
                            },
                            {
                              "required": [
                                "phone"
                              ]
                            },
                            {
                              "required": [
                                "message"
                              ]
                            },
                            {
                              "required": [
                                "custom_order_id"
                              ]
                            }
                          ],
                          "properties": {
                            "email": {
                              "type": "string",
                              "description": "Customer service email",
                              "example": "your-name@your-domain.com"
                            },
                            "phone": {
                              "type": "string",
                              "description": "Customer service phone",
                              "example": "+371 28888888"
                            },
                            "message": {
                              "type": "string",
                              "description": "Custom packing slip message",
                              "example": "Message on packing slip"
                            },
                            "logo_url": {
                              "type": "string",
                              "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                              "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                            },
                            "store_name": {
                              "type": "string",
                              "description": "Store name override for the return address",
                              "example": "Your store name"
                            },
                            "custom_order_id": {
                              "type": "string",
                              "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                              "example": "kkk2344lm"
                            }
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/orders' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Get 10 latest orders\n$orders = $pf->get('orders', ['limit' => 10]);\n"
    }
  ]
}
```
</details>

---

### POST /orders
**Summary:** Create a new order

Creates a new order and optionally submits it for fulfillment ([See examples](#tag/Examples/Orders-API-examples))

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createOrder",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "confirm",
      "schema": {
        "type": "boolean"
      },
      "required": false,
      "description": "Automatically submit the newly created order for fulfillment (skip the Draft phase)"
    },
    {
      "in": "query",
      "name": "update_existing",
      "schema": {
        "type": "boolean"
      },
      "required": false,
      "description": "Try to update existing order if an order with the specified external_id already exists"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "Order",
          "description": "Information about the Order",
          "required": [
            "recipient",
            "items"
          ],
          "properties": {
            "id": {
              "description": "Order ID",
              "type": "integer",
              "example": 13,
              "readOnly": true
            },
            "external_id": {
              "description": "Order ID from the external system",
              "type": "string",
              "nullable": true,
              "example": "4235234213"
            },
            "store": {
              "description": "Store ID",
              "type": "integer",
              "example": 10,
              "readOnly": true
            },
            "status": {
              "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
              "type": "string",
              "example": "draft",
              "readOnly": true
            },
            "shipping": {
              "description": "Shipping method. Defaults to 'STANDARD'",
              "type": "string",
              "example": "STANDARD"
            },
            "shipping_service_name": {
              "description": "Human readable shipping method name.",
              "type": "string",
              "example": "Flat Rate (3-4 business days after fulfillment)",
              "readOnly": true
            },
            "created": {
              "description": "Time when the order was created",
              "type": "integer",
              "example": 1602607640,
              "readOnly": true
            },
            "updated": {
              "description": "Time when the order was updated",
              "type": "integer",
              "example": 1602607640,
              "readOnly": true
            },
            "recipient": {
              "type": "object",
              "title": "Address",
              "description": "Information about the address",
              "properties": {
                "name": {
                  "description": "Full name",
                  "type": "string",
                  "example": "John Smith"
                },
                "company": {
                  "description": "Company name",
                  "type": "string",
                  "example": "John Smith Inc"
                },
                "address1": {
                  "description": "Address line 1",
                  "type": "string",
                  "example": "19749 Dearborn St"
                },
                "address2": {
                  "description": "Address line 2",
                  "type": "string"
                },
                "city": {
                  "description": "City",
                  "type": "string",
                  "example": "Chatsworth"
                },
                "state_code": {
                  "description": "State code",
                  "type": "string",
                  "example": "CA"
                },
                "state_name": {
                  "description": "State name",
                  "type": "string",
                  "example": "California"
                },
                "country_code": {
                  "description": "Country code",
                  "type": "string",
                  "example": "US"
                },
                "country_name": {
                  "description": "Country name",
                  "type": "string",
                  "example": "United States"
                },
                "zip": {
                  "description": "ZIP/Postal code",
                  "type": "string",
                  "example": "91311"
                },
                "phone": {
                  "description": "Phone number",
                  "example": "2312322334",
                  "type": "string"
                },
                "email": {
                  "description": "Email address",
                  "example": "firstname.secondname@domain.com",
                  "type": "string"
                },
                "tax_number": {
                  "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                  "type": "string",
                  "example": "123.456.789-10"
                }
              }
            },
            "items": {
              "type": "array",
              "description": "Array of items in the order",
              "items": {
                "type": "object",
                "title": "Item",
                "description": "Information about an item in the order",
                "properties": {
                  "id": {
                    "description": "Line item ID",
                    "type": "integer",
                    "example": 1
                  },
                  "external_id": {
                    "description": "Line item ID from the external system",
                    "type": "string",
                    "example": "item-1"
                  },
                  "variant_id": {
                    "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                    "type": "integer",
                    "example": 1
                  },
                  "sync_variant_id": {
                    "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                    "type": "integer",
                    "example": 1
                  },
                  "external_variant_id": {
                    "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                    "type": "string",
                    "example": "variant-1"
                  },
                  "warehouse_product_variant_id": {
                    "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                    "type": "integer",
                    "example": 1
                  },
                  "product_template_id": {
                    "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                    "type": "integer",
                    "example": 1
                  },
                  "external_product_id": {
                    "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                    "type": "string",
                    "example": "template-123",
                    "writeOnly": true
                  },
                  "quantity": {
                    "description": "Number of items ordered (Limited to 1000 for one item)",
                    "type": "integer",
                    "example": 1
                  },
                  "price": {
                    "description": "Printful price of the item",
                    "type": "string",
                    "example": "13.00"
                  },
                  "retail_price": {
                    "description": "Original retail price of the item to be displayed on the packing slip",
                    "type": "string",
                    "example": "13.00"
                  },
                  "name": {
                    "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                    "type": "string",
                    "example": "Enhanced Matte Paper Poster 18\u00d724"
                  },
                  "product": {
                    "type": "object",
                    "title": "ProductVariant",
                    "description": "Short information about the Printful Product and Variant",
                    "properties": {
                      "variant_id": {
                        "type": "integer",
                        "description": "Variant ID",
                        "example": 3001
                      },
                      "product_id": {
                        "type": "integer",
                        "description": "Product ID of this variant",
                        "example": 301
                      },
                      "image": {
                        "type": "string",
                        "description": "URL of a sample image for this variant",
                        "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                      },
                      "name": {
                        "type": "string",
                        "description": "Display name of this variant",
                        "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                      }
                    }
                  },
                  "files": {
                    "type": "array",
                    "items": {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "File",
                          "description": "Information about the File",
                          "required": [
                            "url"
                          ],
                          "properties": {
                            "type": {
                              "type": "string",
                              "description": "Role of the file",
                              "example": "default"
                            },
                            "id": {
                              "type": "integer",
                              "description": "File ID",
                              "example": 10,
                              "readOnly": true
                            },
                            "url": {
                              "type": "string",
                              "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                              "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "FileOption",
                                "description": "File option",
                                "required": [
                                  "id",
                                  "value"
                                ],
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option id",
                                    "example": "template_type"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "native"
                                  }
                                }
                              }
                            },
                            "hash": {
                              "type": "string",
                              "description": "MD5 checksum of the file",
                              "example": "ea44330b887dfec278dbc4626a759547",
                              "readOnly": true
                            },
                            "filename": {
                              "type": "string",
                              "description": "File name",
                              "example": "shirt1.png"
                            },
                            "mime_type": {
                              "type": "string",
                              "description": "MIME type of the file",
                              "example": "image/png",
                              "readOnly": true
                            },
                            "size": {
                              "type": "integer",
                              "description": "Size in bytes",
                              "example": 45582633,
                              "readOnly": true
                            },
                            "width": {
                              "type": "integer",
                              "description": "Width in pixels",
                              "example": 1000,
                              "readOnly": true
                            },
                            "height": {
                              "type": "integer",
                              "description": "Height in pixels",
                              "example": 1000,
                              "readOnly": true
                            },
                            "dpi": {
                              "type": "integer",
                              "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                              "example": 300,
                              "readOnly": true
                            },
                            "status": {
                              "type": "string",
                              "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                              "example": "ok",
                              "readOnly": true
                            },
                            "created": {
                              "type": "integer",
                              "description": "File creation timestamp",
                              "example": 1590051937,
                              "readOnly": true
                            },
                            "thumbnail_url": {
                              "type": "string",
                              "description": "Small thumbnail URL",
                              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                              "readOnly": true
                            },
                            "preview_url": {
                              "type": "string",
                              "description": "Medium preview image URL",
                              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                              "readOnly": true
                            },
                            "visible": {
                              "type": "boolean",
                              "description": "Show file in the Printfile Library (default true)",
                              "example": true
                            },
                            "is_temporary": {
                              "type": "boolean",
                              "readOnly": true,
                              "description": "Whether it is a temporary printfile.",
                              "example": false
                            }
                          }
                        },
                        {
                          "type": "object",
                          "properties": {
                            "position": {
                              "writeOnly": true,
                              "allOf": [
                                {
                                  "type": "object",
                                  "title": "GenerationTaskFilePosition",
                                  "description": "Position",
                                  "properties": {
                                    "area_width": {
                                      "type": "integer",
                                      "description": "Positioning area width on print area in pixels",
                                      "example": 1800,
                                      "nullable": true
                                    },
                                    "area_height": {
                                      "type": "integer",
                                      "description": "Positioning area height on print area in pixels",
                                      "example": 2400,
                                      "nullable": true
                                    },
                                    "width": {
                                      "type": "integer",
                                      "description": "Width of the image in given area in pixels",
                                      "example": 1800,
                                      "readOnly": false
                                    },
                                    "height": {
                                      "type": "integer",
                                      "description": "Height of the image in given area in pixels",
                                      "example": 1800,
                                      "readOnly": false
                                    },
                                    "top": {
                                      "type": "integer",
                                      "description": "Image top offset in given area in pixels",
                                      "example": 300
                                    },
                                    "left": {
                                      "type": "integer",
                                      "description": "Image left offset in given area in pixels",
                                      "example": 0
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "properties": {
                                    "limit_to_print_area": {
                                      "type": "boolean",
                                      "default": true,
                                      "description": "Should limit printfile to print area"
                                    }
                                  }
                                }
                              ]
                            }
                          }
                        }
                      ]
                    },
                    "description": "Array of attached printfiles / preview images"
                  },
                  "options": {
                    "type": "array",
                    "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                    "items": {
                      "type": "object",
                      "title": "ItemOption",
                      "description": "Additional option for order item",
                      "properties": {
                        "id": {
                          "type": "string",
                          "description": "Option ID",
                          "example": "OptionKey"
                        },
                        "value": {
                          "type": "string",
                          "description": "Option value",
                          "example": "OptionValue"
                        }
                      }
                    }
                  },
                  "sku": {
                    "description": "Product identifier (SKU) from the external system",
                    "type": "string",
                    "example": null
                  },
                  "discontinued": {
                    "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                    "type": "boolean",
                    "example": true
                  },
                  "out_of_stock": {
                    "description": "Whether the item is out of stock i.e. temporarily unavailable",
                    "type": "boolean",
                    "example": true
                  }
                },
                "anyOf": [
                  {
                    "required": [
                      "variant_id"
                    ]
                  },
                  {
                    "required": [
                      "sync_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "external_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "warehouse_product_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "variant_id",
                      "product_template_id"
                    ]
                  },
                  {
                    "required": [
                      "variant_id",
                      "external_product_id"
                    ]
                  }
                ],
                "not": {
                  "anyOf": [
                    {
                      "required": [
                        "product_template_id",
                        "sync_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "external_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "warehouse_product_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "files"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "options"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "external_product_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "sync_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "external_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "warehouse_product_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "files"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "options"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "product_template_id"
                      ]
                    }
                  ]
                }
              }
            },
            "branding_items": {
              "type": "array",
              "description": "Array of branding items in the order",
              "readOnly": true,
              "items": {
                "$ref": "circular reference to #/components/schemas/Item"
              }
            },
            "incomplete_items": {
              "type": "array",
              "description": "Array of incomplete items in the order",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "IncompleteItem",
                "description": "Information about an incomplete item in the order",
                "properties": {
                  "name": {
                    "type": "string",
                    "description": "Incomplete item name",
                    "example": "Red T-Shirt"
                  },
                  "quantity": {
                    "description": "Incompleted item quantity",
                    "type": "integer",
                    "example": 1
                  },
                  "sync_variant_id": {
                    "description": "Sync variant ID of the incompleted item.",
                    "type": "integer",
                    "example": 70
                  },
                  "external_variant_id": {
                    "description": "External variant ID of the incompleted item.",
                    "type": "string",
                    "example": "external-id"
                  },
                  "external_line_item_id": {
                    "description": "External order line item id.",
                    "type": "string",
                    "example": "external-line-item-id"
                  }
                }
              }
            },
            "costs": {
              "allOf": [
                {
                  "type": "object",
                  "title": "OrderCosts",
                  "description": "Order costs (Printful prices)",
                  "properties": {
                    "currency": {
                      "type": "string",
                      "description": "3 letter currency code",
                      "example": "USD"
                    },
                    "subtotal": {
                      "type": "string",
                      "description": "Total cost of all items",
                      "example": "10.00"
                    },
                    "discount": {
                      "type": "string",
                      "description": "Discount sum",
                      "example": "0.00"
                    },
                    "shipping": {
                      "type": "string",
                      "description": "Shipping costs",
                      "example": "5.00"
                    },
                    "digitization": {
                      "type": "string",
                      "description": "Digitization costs",
                      "example": "0.00"
                    },
                    "additional_fee": {
                      "type": "string",
                      "description": "Additional fee for custom product",
                      "example": "0.00"
                    },
                    "fulfillment_fee": {
                      "type": "string",
                      "description": "Custom product fulfillment fee",
                      "example": "0.00"
                    },
                    "retail_delivery_fee": {
                      "type": "string",
                      "description": "Retail delivery fee",
                      "example": "0.00"
                    },
                    "tax": {
                      "type": "string",
                      "description": "Sum of taxes (not included in the item price)",
                      "example": "0.00"
                    },
                    "vat": {
                      "type": "string",
                      "description": "Sum of vat (not included in the item price)",
                      "example": "0.00"
                    },
                    "total": {
                      "type": "string",
                      "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                      "example": "15.00"
                    }
                  }
                },
                {
                  "readOnly": true
                }
              ]
            },
            "retail_costs": {
              "type": "object",
              "title": "OrderRetailCosts",
              "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
              "properties": {
                "currency": {
                  "type": "string",
                  "description": "3 letter currency code",
                  "example": "USD"
                },
                "subtotal": {
                  "type": "string",
                  "nullable": true,
                  "description": "Total cost of all items",
                  "example": "10.00"
                },
                "discount": {
                  "type": "string",
                  "nullable": true,
                  "description": "Discount sum",
                  "example": "0.00"
                },
                "shipping": {
                  "type": "string",
                  "nullable": true,
                  "description": "Shipping costs",
                  "example": "5.00"
                },
                "tax": {
                  "type": "string",
                  "nullable": true,
                  "description": "Sum of taxes (not included in the item price)",
                  "example": "0.00"
                },
                "vat": {
                  "type": "string",
                  "nullable": true,
                  "readOnly": true,
                  "description": "Sum of VAT (not included in the item price)",
                  "example": "0.00"
                },
                "total": {
                  "type": "string",
                  "nullable": true,
                  "readOnly": true,
                  "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                  "example": "15.00"
                }
              }
            },
            "pricing_breakdown": {
              "type": "array",
              "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "PricingBreakdown",
                "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                "properties": {
                  "customer_pays": {
                    "type": "string",
                    "description": "Amount customer paid",
                    "example": "3.75"
                  },
                  "printful_price": {
                    "type": "string",
                    "description": "Printful price",
                    "example": "6"
                  },
                  "profit": {
                    "type": "string",
                    "description": "Profit",
                    "example": "-2.25"
                  },
                  "currency_symbol": {
                    "type": "string",
                    "description": "Shipment tracking number",
                    "example": "USD"
                  }
                }
              }
            },
            "shipments": {
              "type": "array",
              "description": "Array of shipments already shipped for this order",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "OrderShipment",
                "description": "Information about order shipment",
                "properties": {
                  "id": {
                    "type": "integer",
                    "description": "Shipment ID",
                    "example": 10
                  },
                  "carrier": {
                    "type": "string",
                    "description": "Carrier name",
                    "example": "FEDEX"
                  },
                  "service": {
                    "type": "string",
                    "description": "Delivery service name",
                    "example": "FedEx SmartPost"
                  },
                  "tracking_number": {
                    "type": "string",
                    "description": "Shipment tracking number",
                    "example": 0
                  },
                  "tracking_url": {
                    "type": "string",
                    "description": "Shipment tracking URL",
                    "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                  },
                  "created": {
                    "type": "integer",
                    "description": "Shipping time",
                    "example": 1588716060
                  },
                  "ship_date": {
                    "type": "string",
                    "description": "Ship date",
                    "example": "2020-05-05"
                  },
                  "shipped_at": {
                    "type": "string",
                    "description": "Ship time in unix timestamp",
                    "example": 1588716060
                  },
                  "reshipment": {
                    "type": "boolean",
                    "description": "Whether this is a reshipment",
                    "example": false
                  },
                  "items": {
                    "type": "array",
                    "description": "Array of items in this shipment",
                    "items": {
                      "type": "object",
                      "title": "OrderShipmentItem",
                      "description": "Information about item in an order shipment",
                      "properties": {
                        "item_id": {
                          "type": "integer",
                          "description": "Line item ID",
                          "example": 1
                        },
                        "quantity": {
                          "type": "integer",
                          "description": "Quantity of items in this shipment",
                          "example": 1
                        },
                        "picked": {
                          "type": "integer",
                          "enum": [
                            0,
                            1
                          ],
                          "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                          "example": 1
                        },
                        "printed": {
                          "type": "integer",
                          "enum": [
                            0,
                            1
                          ],
                          "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                          "example": 1
                        }
                      }
                    }
                  }
                }
              }
            },
            "gift": {
              "type": "object",
              "title": "OrderGift",
              "description": "Optional gift message for the packing slip",
              "properties": {
                "subject": {
                  "type": "string",
                  "maxLength": 200,
                  "description": "Gift message title",
                  "example": "To John"
                },
                "message": {
                  "type": "string",
                  "maxLength": 200,
                  "description": "Gift message text",
                  "example": "Have a nice day"
                }
              }
            },
            "packing_slip": {
              "type": "object",
              "title": "OrderPackingSlip",
              "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
              "anyOf": [
                {
                  "required": [
                    "email"
                  ]
                },
                {
                  "required": [
                    "phone"
                  ]
                },
                {
                  "required": [
                    "message"
                  ]
                },
                {
                  "required": [
                    "custom_order_id"
                  ]
                }
              ],
              "properties": {
                "email": {
                  "type": "string",
                  "description": "Customer service email",
                  "example": "your-name@your-domain.com"
                },
                "phone": {
                  "type": "string",
                  "description": "Customer service phone",
                  "example": "+371 28888888"
                },
                "message": {
                  "type": "string",
                  "description": "Custom packing slip message",
                  "example": "Message on packing slip"
                },
                "logo_url": {
                  "type": "string",
                  "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                  "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                },
                "store_name": {
                  "type": "string",
                  "description": "Store name override for the return address",
                  "example": "Your store name"
                },
                "custom_order_id": {
                  "type": "string",
                  "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                  "example": "kkk2344lm"
                }
              }
            }
          }
        }
      }
    }
  },
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "code": {
                    "type": "integer",
                    "description": "Response status code `200`",
                    "example": 200
                  }
                }
              },
              {
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "Order",
                    "description": "Information about the Order",
                    "required": [
                      "recipient",
                      "items"
                    ],
                    "properties": {
                      "id": {
                        "description": "Order ID",
                        "type": "integer",
                        "example": 13,
                        "readOnly": true
                      },
                      "external_id": {
                        "description": "Order ID from the external system",
                        "type": "string",
                        "nullable": true,
                        "example": "4235234213"
                      },
                      "store": {
                        "description": "Store ID",
                        "type": "integer",
                        "example": 10,
                        "readOnly": true
                      },
                      "status": {
                        "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
                        "type": "string",
                        "example": "draft",
                        "readOnly": true
                      },
                      "shipping": {
                        "description": "Shipping method. Defaults to 'STANDARD'",
                        "type": "string",
                        "example": "STANDARD"
                      },
                      "shipping_service_name": {
                        "description": "Human readable shipping method name.",
                        "type": "string",
                        "example": "Flat Rate (3-4 business days after fulfillment)",
                        "readOnly": true
                      },
                      "created": {
                        "description": "Time when the order was created",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "updated": {
                        "description": "Time when the order was updated",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "recipient": {
                        "type": "object",
                        "title": "Address",
                        "description": "Information about the address",
                        "properties": {
                          "name": {
                            "description": "Full name",
                            "type": "string",
                            "example": "John Smith"
                          },
                          "company": {
                            "description": "Company name",
                            "type": "string",
                            "example": "John Smith Inc"
                          },
                          "address1": {
                            "description": "Address line 1",
                            "type": "string",
                            "example": "19749 Dearborn St"
                          },
                          "address2": {
                            "description": "Address line 2",
                            "type": "string"
                          },
                          "city": {
                            "description": "City",
                            "type": "string",
                            "example": "Chatsworth"
                          },
                          "state_code": {
                            "description": "State code",
                            "type": "string",
                            "example": "CA"
                          },
                          "state_name": {
                            "description": "State name",
                            "type": "string",
                            "example": "California"
                          },
                          "country_code": {
                            "description": "Country code",
                            "type": "string",
                            "example": "US"
                          },
                          "country_name": {
                            "description": "Country name",
                            "type": "string",
                            "example": "United States"
                          },
                          "zip": {
                            "description": "ZIP/Postal code",
                            "type": "string",
                            "example": "91311"
                          },
                          "phone": {
                            "description": "Phone number",
                            "example": "2312322334",
                            "type": "string"
                          },
                          "email": {
                            "description": "Email address",
                            "example": "firstname.secondname@domain.com",
                            "type": "string"
                          },
                          "tax_number": {
                            "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                            "type": "string",
                            "example": "123.456.789-10"
                          }
                        }
                      },
                      "items": {
                        "type": "array",
                        "description": "Array of items in the order",
                        "items": {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about an item in the order",
                          "properties": {
                            "id": {
                              "description": "Line item ID",
                              "type": "integer",
                              "example": 1
                            },
                            "external_id": {
                              "description": "Line item ID from the external system",
                              "type": "string",
                              "example": "item-1"
                            },
                            "variant_id": {
                              "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                              "type": "integer",
                              "example": 1
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                              "type": "string",
                              "example": "variant-1"
                            },
                            "warehouse_product_variant_id": {
                              "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                              "type": "integer",
                              "example": 1
                            },
                            "product_template_id": {
                              "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "integer",
                              "example": 1
                            },
                            "external_product_id": {
                              "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "string",
                              "example": "template-123",
                              "writeOnly": true
                            },
                            "quantity": {
                              "description": "Number of items ordered (Limited to 1000 for one item)",
                              "type": "integer",
                              "example": 1
                            },
                            "price": {
                              "description": "Printful price of the item",
                              "type": "string",
                              "example": "13.00"
                            },
                            "retail_price": {
                              "description": "Original retail price of the item to be displayed on the packing slip",
                              "type": "string",
                              "example": "13.00"
                            },
                            "name": {
                              "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                              "type": "string",
                              "example": "Enhanced Matte Paper Poster 18\u00d724"
                            },
                            "product": {
                              "type": "object",
                              "title": "ProductVariant",
                              "description": "Short information about the Printful Product and Variant",
                              "properties": {
                                "variant_id": {
                                  "type": "integer",
                                  "description": "Variant ID",
                                  "example": 3001
                                },
                                "product_id": {
                                  "type": "integer",
                                  "description": "Product ID of this variant",
                                  "example": 301
                                },
                                "image": {
                                  "type": "string",
                                  "description": "URL of a sample image for this variant",
                                  "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                                },
                                "name": {
                                  "type": "string",
                                  "description": "Display name of this variant",
                                  "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                                }
                              }
                            },
                            "files": {
                              "type": "array",
                              "items": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "File",
                                    "description": "Information about the File",
                                    "required": [
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "type": "string",
                                        "description": "Role of the file",
                                        "example": "default"
                                      },
                                      "id": {
                                        "type": "integer",
                                        "description": "File ID",
                                        "example": 10,
                                        "readOnly": true
                                      },
                                      "url": {
                                        "type": "string",
                                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                                      },
                                      "options": {
                                        "type": "array",
                                        "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                                        "items": {
                                          "type": "object",
                                          "title": "FileOption",
                                          "description": "File option",
                                          "required": [
                                            "id",
                                            "value"
                                          ],
                                          "properties": {
                                            "id": {
                                              "type": "string",
                                              "description": "Option id",
                                              "example": "template_type"
                                            },
                                            "value": {
                                              "type": "string",
                                              "description": "Option value",
                                              "example": "native"
                                            }
                                          }
                                        }
                                      },
                                      "hash": {
                                        "type": "string",
                                        "description": "MD5 checksum of the file",
                                        "example": "ea44330b887dfec278dbc4626a759547",
                                        "readOnly": true
                                      },
                                      "filename": {
                                        "type": "string",
                                        "description": "File name",
                                        "example": "shirt1.png"
                                      },
                                      "mime_type": {
                                        "type": "string",
                                        "description": "MIME type of the file",
                                        "example": "image/png",
                                        "readOnly": true
                                      },
                                      "size": {
                                        "type": "integer",
                                        "description": "Size in bytes",
                                        "example": 45582633,
                                        "readOnly": true
                                      },
                                      "width": {
                                        "type": "integer",
                                        "description": "Width in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "height": {
                                        "type": "integer",
                                        "description": "Height in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "dpi": {
                                        "type": "integer",
                                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                                        "example": 300,
                                        "readOnly": true
                                      },
                                      "status": {
                                        "type": "string",
                                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                                        "example": "ok",
                                        "readOnly": true
                                      },
                                      "created": {
                                        "type": "integer",
                                        "description": "File creation timestamp",
                                        "example": 1590051937,
                                        "readOnly": true
                                      },
                                      "thumbnail_url": {
                                        "type": "string",
                                        "description": "Small thumbnail URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "preview_url": {
                                        "type": "string",
                                        "description": "Medium preview image URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "visible": {
                                        "type": "boolean",
                                        "description": "Show file in the Printfile Library (default true)",
                                        "example": true
                                      },
                                      "is_temporary": {
                                        "type": "boolean",
                                        "readOnly": true,
                                        "description": "Whether it is a temporary printfile.",
                                        "example": false
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "properties": {
                                      "position": {
                                        "writeOnly": true,
                                        "allOf": [
                                          {
                                            "type": "object",
                                            "title": "GenerationTaskFilePosition",
                                            "description": "Position",
                                            "properties": {
                                              "area_width": {
                                                "type": "integer",
                                                "description": "Positioning area width on print area in pixels",
                                                "example": 1800,
                                                "nullable": true
                                              },
                                              "area_height": {
                                                "type": "integer",
                                                "description": "Positioning area height on print area in pixels",
                                                "example": 2400,
                                                "nullable": true
                                              },
                                              "width": {
                                                "type": "integer",
                                                "description": "Width of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "height": {
                                                "type": "integer",
                                                "description": "Height of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "top": {
                                                "type": "integer",
                                                "description": "Image top offset in given area in pixels",
                                                "example": 300
                                              },
                                              "left": {
                                                "type": "integer",
                                                "description": "Image left offset in given area in pixels",
                                                "example": 0
                                              }
                                            }
                                          },
                                          {
                                            "type": "object",
                                            "properties": {
                                              "limit_to_print_area": {
                                                "type": "boolean",
                                                "default": true,
                                                "description": "Should limit printfile to print area"
                                              }
                                            }
                                          }
                                        ]
                                      }
                                    }
                                  }
                                ]
                              },
                              "description": "Array of attached printfiles / preview images"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "ItemOption",
                                "description": "Additional option for order item",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option ID",
                                    "example": "OptionKey"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "OptionValue"
                                  }
                                }
                              }
                            },
                            "sku": {
                              "description": "Product identifier (SKU) from the external system",
                              "type": "string",
                              "example": null
                            },
                            "discontinued": {
                              "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                              "type": "boolean",
                              "example": true
                            },
                            "out_of_stock": {
                              "description": "Whether the item is out of stock i.e. temporarily unavailable",
                              "type": "boolean",
                              "example": true
                            }
                          },
                          "anyOf": [
                            {
                              "required": [
                                "variant_id"
                              ]
                            },
                            {
                              "required": [
                                "sync_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "external_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "warehouse_product_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "product_template_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "external_product_id"
                              ]
                            }
                          ],
                          "not": {
                            "anyOf": [
                              {
                                "required": [
                                  "product_template_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_product_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "product_template_id"
                                ]
                              }
                            ]
                          }
                        }
                      },
                      "branding_items": {
                        "type": "array",
                        "description": "Array of branding items in the order",
                        "readOnly": true,
                        "items": {
                          "$ref": "circular reference to #/components/schemas/Item"
                        }
                      },
                      "incomplete_items": {
                        "type": "array",
                        "description": "Array of incomplete items in the order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "IncompleteItem",
                          "description": "Information about an incomplete item in the order",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Incomplete item name",
                              "example": "Red T-Shirt"
                            },
                            "quantity": {
                              "description": "Incompleted item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the incompleted item.",
                              "type": "integer",
                              "example": 70
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the incompleted item.",
                              "type": "string",
                              "example": "external-id"
                            },
                            "external_line_item_id": {
                              "description": "External order line item id.",
                              "type": "string",
                              "example": "external-line-item-id"
                            }
                          }
                        }
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "OrderCosts",
                            "description": "Order costs (Printful prices)",
                            "properties": {
                              "currency": {
                                "type": "string",
                                "description": "3 letter currency code",
                                "example": "USD"
                              },
                              "subtotal": {
                                "type": "string",
                                "description": "Total cost of all items",
                                "example": "10.00"
                              },
                              "discount": {
                                "type": "string",
                                "description": "Discount sum",
                                "example": "0.00"
                              },
                              "shipping": {
                                "type": "string",
                                "description": "Shipping costs",
                                "example": "5.00"
                              },
                              "digitization": {
                                "type": "string",
                                "description": "Digitization costs",
                                "example": "0.00"
                              },
                              "additional_fee": {
                                "type": "string",
                                "description": "Additional fee for custom product",
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "type": "string",
                                "description": "Custom product fulfillment fee",
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "type": "string",
                                "description": "Retail delivery fee",
                                "example": "0.00"
                              },
                              "tax": {
                                "type": "string",
                                "description": "Sum of taxes (not included in the item price)",
                                "example": "0.00"
                              },
                              "vat": {
                                "type": "string",
                                "description": "Sum of vat (not included in the item price)",
                                "example": "0.00"
                              },
                              "total": {
                                "type": "string",
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                                "example": "15.00"
                              }
                            }
                          },
                          {
                            "readOnly": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "type": "object",
                        "title": "OrderRetailCosts",
                        "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "string",
                            "nullable": true,
                            "description": "Total cost of all items",
                            "example": "10.00"
                          },
                          "discount": {
                            "type": "string",
                            "nullable": true,
                            "description": "Discount sum",
                            "example": "0.00"
                          },
                          "shipping": {
                            "type": "string",
                            "nullable": true,
                            "description": "Shipping costs",
                            "example": "5.00"
                          },
                          "tax": {
                            "type": "string",
                            "nullable": true,
                            "description": "Sum of taxes (not included in the item price)",
                            "example": "0.00"
                          },
                          "vat": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Sum of VAT (not included in the item price)",
                            "example": "0.00"
                          },
                          "total": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": "15.00"
                          }
                        }
                      },
                      "pricing_breakdown": {
                        "type": "array",
                        "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "PricingBreakdown",
                          "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                          "properties": {
                            "customer_pays": {
                              "type": "string",
                              "description": "Amount customer paid",
                              "example": "3.75"
                            },
                            "printful_price": {
                              "type": "string",
                              "description": "Printful price",
                              "example": "6"
                            },
                            "profit": {
                              "type": "string",
                              "description": "Profit",
                              "example": "-2.25"
                            },
                            "currency_symbol": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": "USD"
                            }
                          }
                        }
                      },
                      "shipments": {
                        "type": "array",
                        "description": "Array of shipments already shipped for this order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "OrderShipment",
                          "description": "Information about order shipment",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Shipment ID",
                              "example": 10
                            },
                            "carrier": {
                              "type": "string",
                              "description": "Carrier name",
                              "example": "FEDEX"
                            },
                            "service": {
                              "type": "string",
                              "description": "Delivery service name",
                              "example": "FedEx SmartPost"
                            },
                            "tracking_number": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": 0
                            },
                            "tracking_url": {
                              "type": "string",
                              "description": "Shipment tracking URL",
                              "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                            },
                            "created": {
                              "type": "integer",
                              "description": "Shipping time",
                              "example": 1588716060
                            },
                            "ship_date": {
                              "type": "string",
                              "description": "Ship date",
                              "example": "2020-05-05"
                            },
                            "shipped_at": {
                              "type": "string",
                              "description": "Ship time in unix timestamp",
                              "example": 1588716060
                            },
                            "reshipment": {
                              "type": "boolean",
                              "description": "Whether this is a reshipment",
                              "example": false
                            },
                            "items": {
                              "type": "array",
                              "description": "Array of items in this shipment",
                              "items": {
                                "type": "object",
                                "title": "OrderShipmentItem",
                                "description": "Information about item in an order shipment",
                                "properties": {
                                  "item_id": {
                                    "type": "integer",
                                    "description": "Line item ID",
                                    "example": 1
                                  },
                                  "quantity": {
                                    "type": "integer",
                                    "description": "Quantity of items in this shipment",
                                    "example": 1
                                  },
                                  "picked": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                                    "example": 1
                                  },
                                  "printed": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                                    "example": 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "gift": {
                        "type": "object",
                        "title": "OrderGift",
                        "description": "Optional gift message for the packing slip",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message title",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message text",
                            "example": "Have a nice day"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "OrderPackingSlip",
                        "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                        "anyOf": [
                          {
                            "required": [
                              "email"
                            ]
                          },
                          {
                            "required": [
                              "phone"
                            ]
                          },
                          {
                            "required": [
                              "message"
                            ]
                          },
                          {
                            "required": [
                              "custom_order_id"
                            ]
                          }
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "your-name@your-domain.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+371 28888888"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "Message on packing slip"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "Your store name"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "kkk2344lm"
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "400": {
      "description": "Bad Request",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `400`",
                "example": 400
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Missing required parameters"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Missing required parameters"
                  }
                }
              }
            }
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/orders' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"recipient\": {\n        \"name\": \"John Doe\",\n        \"address1\": \"19749 Dearborn St\",\n        \"city\": \"Chatsworth\",\n        \"state_code\": \"CA\",\n        \"country_code\": \"US\",\n        \"zip\": \"91311\"\n    },\n    \"items\": [\n        {\n            \"variant_id\": 8630,\n            \"quantity\": 1,\n            \"files\": [\n                {\n                    \"type\": \"front\",\n                    \"url\": \"https://picsum.photos/200\"\n                }\n            ]\n        }\n    ],\n    \"packing_slip\": {\n        \"email\": \"john.doe@printful.com\",\n        \"phone\": \"288-888-8888\",\n        \"message\": \"Custom packing slip\",\n        \"logo_url\": \"https://i.picsum.photos/id/817/2000/2000.jpg\"\n    }\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n$order = $pf->post('orders', [\n    'recipient' => [\n        'name' => 'John Doe',\n        'address1' => '172 W Street Ave #105',\n        'city' => 'Burbank',\n        'state_code' => 'CA',\n        'country_code' => 'US',\n        'zip' => '91502',\n    ],\n    'items' => [\n        [\n            'variant_id' => 1, //Small poster\n            'name' => 'Leggings', //Display name\n            'retail_price' => '19.99', //Retail price for packing slip\n            'quantity' => 1,\n            'files' => [\n                [\n                    'url' => 'http://example.com/files/posters/poster_1.jpg',\n                ],\n            ],\n        ],\n        [\n            'variant_id' => 1118,\n            'quantity' => 2,\n            'name' => 'Grand Canyon T-Shirt', //Display name\n            'retail_price' => '29.99', //Retail price for packing slip\n            'files' => [\n                [ // Front print\n                    'url' => 'http://example.com/files/tshirts/shirt_front.png',\n                ],\n                [ // Back print\n                    'type' => 'back',\n                    'url' => 'http://example.com/files/tshirts/shirt_back.png',\n                ],\n                [ // Mockup image\n                    'type' => 'preview',\n                    'url' => 'http://example.com/files/tshirts/shirt_mockup.jpg',\n                ],\n            ],\n            'options' => [ // Additional options\n                [\n                    'id' => 'remove_labels',\n                    'value' => true,\n                ],\n            ],\n        ],\n    ],\n]);\n\nvar_export($order);"
    }
  ]
}
```
</details>

---

### GET /orders/{id}
**Summary:** Get order data

Returns order data by ID or External ID.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getOrderById",
  "security": [
    {
      "OAuth": [
        "orders/read"
      ]
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "oneOf": [
          {
            "type": "string"
          },
          {
            "type": "integer"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or External ID (if prefixed with @)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "code": {
                    "type": "integer",
                    "description": "Response status code `200`",
                    "example": 200
                  }
                }
              },
              {
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "Order",
                    "description": "Information about the Order",
                    "required": [
                      "recipient",
                      "items"
                    ],
                    "properties": {
                      "id": {
                        "description": "Order ID",
                        "type": "integer",
                        "example": 13,
                        "readOnly": true
                      },
                      "external_id": {
                        "description": "Order ID from the external system",
                        "type": "string",
                        "nullable": true,
                        "example": "4235234213"
                      },
                      "store": {
                        "description": "Store ID",
                        "type": "integer",
                        "example": 10,
                        "readOnly": true
                      },
                      "status": {
                        "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
                        "type": "string",
                        "example": "draft",
                        "readOnly": true
                      },
                      "shipping": {
                        "description": "Shipping method. Defaults to 'STANDARD'",
                        "type": "string",
                        "example": "STANDARD"
                      },
                      "shipping_service_name": {
                        "description": "Human readable shipping method name.",
                        "type": "string",
                        "example": "Flat Rate (3-4 business days after fulfillment)",
                        "readOnly": true
                      },
                      "created": {
                        "description": "Time when the order was created",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "updated": {
                        "description": "Time when the order was updated",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "recipient": {
                        "type": "object",
                        "title": "Address",
                        "description": "Information about the address",
                        "properties": {
                          "name": {
                            "description": "Full name",
                            "type": "string",
                            "example": "John Smith"
                          },
                          "company": {
                            "description": "Company name",
                            "type": "string",
                            "example": "John Smith Inc"
                          },
                          "address1": {
                            "description": "Address line 1",
                            "type": "string",
                            "example": "19749 Dearborn St"
                          },
                          "address2": {
                            "description": "Address line 2",
                            "type": "string"
                          },
                          "city": {
                            "description": "City",
                            "type": "string",
                            "example": "Chatsworth"
                          },
                          "state_code": {
                            "description": "State code",
                            "type": "string",
                            "example": "CA"
                          },
                          "state_name": {
                            "description": "State name",
                            "type": "string",
                            "example": "California"
                          },
                          "country_code": {
                            "description": "Country code",
                            "type": "string",
                            "example": "US"
                          },
                          "country_name": {
                            "description": "Country name",
                            "type": "string",
                            "example": "United States"
                          },
                          "zip": {
                            "description": "ZIP/Postal code",
                            "type": "string",
                            "example": "91311"
                          },
                          "phone": {
                            "description": "Phone number",
                            "example": "2312322334",
                            "type": "string"
                          },
                          "email": {
                            "description": "Email address",
                            "example": "firstname.secondname@domain.com",
                            "type": "string"
                          },
                          "tax_number": {
                            "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                            "type": "string",
                            "example": "123.456.789-10"
                          }
                        }
                      },
                      "items": {
                        "type": "array",
                        "description": "Array of items in the order",
                        "items": {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about an item in the order",
                          "properties": {
                            "id": {
                              "description": "Line item ID",
                              "type": "integer",
                              "example": 1
                            },
                            "external_id": {
                              "description": "Line item ID from the external system",
                              "type": "string",
                              "example": "item-1"
                            },
                            "variant_id": {
                              "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                              "type": "integer",
                              "example": 1
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                              "type": "string",
                              "example": "variant-1"
                            },
                            "warehouse_product_variant_id": {
                              "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                              "type": "integer",
                              "example": 1
                            },
                            "product_template_id": {
                              "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "integer",
                              "example": 1
                            },
                            "external_product_id": {
                              "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "string",
                              "example": "template-123",
                              "writeOnly": true
                            },
                            "quantity": {
                              "description": "Number of items ordered (Limited to 1000 for one item)",
                              "type": "integer",
                              "example": 1
                            },
                            "price": {
                              "description": "Printful price of the item",
                              "type": "string",
                              "example": "13.00"
                            },
                            "retail_price": {
                              "description": "Original retail price of the item to be displayed on the packing slip",
                              "type": "string",
                              "example": "13.00"
                            },
                            "name": {
                              "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                              "type": "string",
                              "example": "Enhanced Matte Paper Poster 18\u00d724"
                            },
                            "product": {
                              "type": "object",
                              "title": "ProductVariant",
                              "description": "Short information about the Printful Product and Variant",
                              "properties": {
                                "variant_id": {
                                  "type": "integer",
                                  "description": "Variant ID",
                                  "example": 3001
                                },
                                "product_id": {
                                  "type": "integer",
                                  "description": "Product ID of this variant",
                                  "example": 301
                                },
                                "image": {
                                  "type": "string",
                                  "description": "URL of a sample image for this variant",
                                  "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                                },
                                "name": {
                                  "type": "string",
                                  "description": "Display name of this variant",
                                  "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                                }
                              }
                            },
                            "files": {
                              "type": "array",
                              "items": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "File",
                                    "description": "Information about the File",
                                    "required": [
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "type": "string",
                                        "description": "Role of the file",
                                        "example": "default"
                                      },
                                      "id": {
                                        "type": "integer",
                                        "description": "File ID",
                                        "example": 10,
                                        "readOnly": true
                                      },
                                      "url": {
                                        "type": "string",
                                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                                      },
                                      "options": {
                                        "type": "array",
                                        "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                                        "items": {
                                          "type": "object",
                                          "title": "FileOption",
                                          "description": "File option",
                                          "required": [
                                            "id",
                                            "value"
                                          ],
                                          "properties": {
                                            "id": {
                                              "type": "string",
                                              "description": "Option id",
                                              "example": "template_type"
                                            },
                                            "value": {
                                              "type": "string",
                                              "description": "Option value",
                                              "example": "native"
                                            }
                                          }
                                        }
                                      },
                                      "hash": {
                                        "type": "string",
                                        "description": "MD5 checksum of the file",
                                        "example": "ea44330b887dfec278dbc4626a759547",
                                        "readOnly": true
                                      },
                                      "filename": {
                                        "type": "string",
                                        "description": "File name",
                                        "example": "shirt1.png"
                                      },
                                      "mime_type": {
                                        "type": "string",
                                        "description": "MIME type of the file",
                                        "example": "image/png",
                                        "readOnly": true
                                      },
                                      "size": {
                                        "type": "integer",
                                        "description": "Size in bytes",
                                        "example": 45582633,
                                        "readOnly": true
                                      },
                                      "width": {
                                        "type": "integer",
                                        "description": "Width in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "height": {
                                        "type": "integer",
                                        "description": "Height in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "dpi": {
                                        "type": "integer",
                                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                                        "example": 300,
                                        "readOnly": true
                                      },
                                      "status": {
                                        "type": "string",
                                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                                        "example": "ok",
                                        "readOnly": true
                                      },
                                      "created": {
                                        "type": "integer",
                                        "description": "File creation timestamp",
                                        "example": 1590051937,
                                        "readOnly": true
                                      },
                                      "thumbnail_url": {
                                        "type": "string",
                                        "description": "Small thumbnail URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "preview_url": {
                                        "type": "string",
                                        "description": "Medium preview image URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "visible": {
                                        "type": "boolean",
                                        "description": "Show file in the Printfile Library (default true)",
                                        "example": true
                                      },
                                      "is_temporary": {
                                        "type": "boolean",
                                        "readOnly": true,
                                        "description": "Whether it is a temporary printfile.",
                                        "example": false
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "properties": {
                                      "position": {
                                        "writeOnly": true,
                                        "allOf": [
                                          {
                                            "type": "object",
                                            "title": "GenerationTaskFilePosition",
                                            "description": "Position",
                                            "properties": {
                                              "area_width": {
                                                "type": "integer",
                                                "description": "Positioning area width on print area in pixels",
                                                "example": 1800,
                                                "nullable": true
                                              },
                                              "area_height": {
                                                "type": "integer",
                                                "description": "Positioning area height on print area in pixels",
                                                "example": 2400,
                                                "nullable": true
                                              },
                                              "width": {
                                                "type": "integer",
                                                "description": "Width of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "height": {
                                                "type": "integer",
                                                "description": "Height of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "top": {
                                                "type": "integer",
                                                "description": "Image top offset in given area in pixels",
                                                "example": 300
                                              },
                                              "left": {
                                                "type": "integer",
                                                "description": "Image left offset in given area in pixels",
                                                "example": 0
                                              }
                                            }
                                          },
                                          {
                                            "type": "object",
                                            "properties": {
                                              "limit_to_print_area": {
                                                "type": "boolean",
                                                "default": true,
                                                "description": "Should limit printfile to print area"
                                              }
                                            }
                                          }
                                        ]
                                      }
                                    }
                                  }
                                ]
                              },
                              "description": "Array of attached printfiles / preview images"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "ItemOption",
                                "description": "Additional option for order item",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option ID",
                                    "example": "OptionKey"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "OptionValue"
                                  }
                                }
                              }
                            },
                            "sku": {
                              "description": "Product identifier (SKU) from the external system",
                              "type": "string",
                              "example": null
                            },
                            "discontinued": {
                              "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                              "type": "boolean",
                              "example": true
                            },
                            "out_of_stock": {
                              "description": "Whether the item is out of stock i.e. temporarily unavailable",
                              "type": "boolean",
                              "example": true
                            }
                          },
                          "anyOf": [
                            {
                              "required": [
                                "variant_id"
                              ]
                            },
                            {
                              "required": [
                                "sync_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "external_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "warehouse_product_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "product_template_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "external_product_id"
                              ]
                            }
                          ],
                          "not": {
                            "anyOf": [
                              {
                                "required": [
                                  "product_template_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_product_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "product_template_id"
                                ]
                              }
                            ]
                          }
                        }
                      },
                      "branding_items": {
                        "type": "array",
                        "description": "Array of branding items in the order",
                        "readOnly": true,
                        "items": {
                          "$ref": "circular reference to #/components/schemas/Item"
                        }
                      },
                      "incomplete_items": {
                        "type": "array",
                        "description": "Array of incomplete items in the order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "IncompleteItem",
                          "description": "Information about an incomplete item in the order",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Incomplete item name",
                              "example": "Red T-Shirt"
                            },
                            "quantity": {
                              "description": "Incompleted item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the incompleted item.",
                              "type": "integer",
                              "example": 70
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the incompleted item.",
                              "type": "string",
                              "example": "external-id"
                            },
                            "external_line_item_id": {
                              "description": "External order line item id.",
                              "type": "string",
                              "example": "external-line-item-id"
                            }
                          }
                        }
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "OrderCosts",
                            "description": "Order costs (Printful prices)",
                            "properties": {
                              "currency": {
                                "type": "string",
                                "description": "3 letter currency code",
                                "example": "USD"
                              },
                              "subtotal": {
                                "type": "string",
                                "description": "Total cost of all items",
                                "example": "10.00"
                              },
                              "discount": {
                                "type": "string",
                                "description": "Discount sum",
                                "example": "0.00"
                              },
                              "shipping": {
                                "type": "string",
                                "description": "Shipping costs",
                                "example": "5.00"
                              },
                              "digitization": {
                                "type": "string",
                                "description": "Digitization costs",
                                "example": "0.00"
                              },
                              "additional_fee": {
                                "type": "string",
                                "description": "Additional fee for custom product",
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "type": "string",
                                "description": "Custom product fulfillment fee",
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "type": "string",
                                "description": "Retail delivery fee",
                                "example": "0.00"
                              },
                              "tax": {
                                "type": "string",
                                "description": "Sum of taxes (not included in the item price)",
                                "example": "0.00"
                              },
                              "vat": {
                                "type": "string",
                                "description": "Sum of vat (not included in the item price)",
                                "example": "0.00"
                              },
                              "total": {
                                "type": "string",
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                                "example": "15.00"
                              }
                            }
                          },
                          {
                            "readOnly": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "type": "object",
                        "title": "OrderRetailCosts",
                        "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "string",
                            "nullable": true,
                            "description": "Total cost of all items",
                            "example": "10.00"
                          },
                          "discount": {
                            "type": "string",
                            "nullable": true,
                            "description": "Discount sum",
                            "example": "0.00"
                          },
                          "shipping": {
                            "type": "string",
                            "nullable": true,
                            "description": "Shipping costs",
                            "example": "5.00"
                          },
                          "tax": {
                            "type": "string",
                            "nullable": true,
                            "description": "Sum of taxes (not included in the item price)",
                            "example": "0.00"
                          },
                          "vat": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Sum of VAT (not included in the item price)",
                            "example": "0.00"
                          },
                          "total": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": "15.00"
                          }
                        }
                      },
                      "pricing_breakdown": {
                        "type": "array",
                        "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "PricingBreakdown",
                          "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                          "properties": {
                            "customer_pays": {
                              "type": "string",
                              "description": "Amount customer paid",
                              "example": "3.75"
                            },
                            "printful_price": {
                              "type": "string",
                              "description": "Printful price",
                              "example": "6"
                            },
                            "profit": {
                              "type": "string",
                              "description": "Profit",
                              "example": "-2.25"
                            },
                            "currency_symbol": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": "USD"
                            }
                          }
                        }
                      },
                      "shipments": {
                        "type": "array",
                        "description": "Array of shipments already shipped for this order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "OrderShipment",
                          "description": "Information about order shipment",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Shipment ID",
                              "example": 10
                            },
                            "carrier": {
                              "type": "string",
                              "description": "Carrier name",
                              "example": "FEDEX"
                            },
                            "service": {
                              "type": "string",
                              "description": "Delivery service name",
                              "example": "FedEx SmartPost"
                            },
                            "tracking_number": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": 0
                            },
                            "tracking_url": {
                              "type": "string",
                              "description": "Shipment tracking URL",
                              "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                            },
                            "created": {
                              "type": "integer",
                              "description": "Shipping time",
                              "example": 1588716060
                            },
                            "ship_date": {
                              "type": "string",
                              "description": "Ship date",
                              "example": "2020-05-05"
                            },
                            "shipped_at": {
                              "type": "string",
                              "description": "Ship time in unix timestamp",
                              "example": 1588716060
                            },
                            "reshipment": {
                              "type": "boolean",
                              "description": "Whether this is a reshipment",
                              "example": false
                            },
                            "items": {
                              "type": "array",
                              "description": "Array of items in this shipment",
                              "items": {
                                "type": "object",
                                "title": "OrderShipmentItem",
                                "description": "Information about item in an order shipment",
                                "properties": {
                                  "item_id": {
                                    "type": "integer",
                                    "description": "Line item ID",
                                    "example": 1
                                  },
                                  "quantity": {
                                    "type": "integer",
                                    "description": "Quantity of items in this shipment",
                                    "example": 1
                                  },
                                  "picked": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                                    "example": 1
                                  },
                                  "printed": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                                    "example": 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "gift": {
                        "type": "object",
                        "title": "OrderGift",
                        "description": "Optional gift message for the packing slip",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message title",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message text",
                            "example": "Have a nice day"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "OrderPackingSlip",
                        "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                        "anyOf": [
                          {
                            "required": [
                              "email"
                            ]
                          },
                          {
                            "required": [
                              "phone"
                            ]
                          },
                          {
                            "required": [
                              "message"
                            ]
                          },
                          {
                            "required": [
                              "custom_order_id"
                            ]
                          }
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "your-name@your-domain.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+371 28888888"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "Message on packing slip"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "Your store name"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "kkk2344lm"
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    },
    "404": {
      "description": "Not found",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `404`",
                "example": 404
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Not found"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "NotFound"
                  },
                  "message": {
                    "type": "string",
                    "example": "Not found"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/orders/{order_id}' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Get order with id 12345\n$order = $pf->get('orders/12345');"
    }
  ]
}
```
</details>

---

### DELETE /orders/{id}
**Summary:** Cancel an order

Cancels pending order or draft. Charged amount is returned to the store owner's credit card.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "cancelOrderById",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "oneOf": [
          {
            "type": "string"
          },
          {
            "type": "integer"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or External ID (if prefixed with @)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "code": {
                    "type": "integer",
                    "description": "Response status code `200`",
                    "example": 200
                  }
                }
              },
              {
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "Order",
                    "description": "Information about the Order",
                    "required": [
                      "recipient",
                      "items"
                    ],
                    "properties": {
                      "id": {
                        "description": "Order ID",
                        "type": "integer",
                        "example": 13,
                        "readOnly": true
                      },
                      "external_id": {
                        "description": "Order ID from the external system",
                        "type": "string",
                        "nullable": true,
                        "example": "4235234213"
                      },
                      "store": {
                        "description": "Store ID",
                        "type": "integer",
                        "example": 10,
                        "readOnly": true
                      },
                      "status": {
                        "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
                        "type": "string",
                        "example": "draft",
                        "readOnly": true
                      },
                      "shipping": {
                        "description": "Shipping method. Defaults to 'STANDARD'",
                        "type": "string",
                        "example": "STANDARD"
                      },
                      "shipping_service_name": {
                        "description": "Human readable shipping method name.",
                        "type": "string",
                        "example": "Flat Rate (3-4 business days after fulfillment)",
                        "readOnly": true
                      },
                      "created": {
                        "description": "Time when the order was created",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "updated": {
                        "description": "Time when the order was updated",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "recipient": {
                        "type": "object",
                        "title": "Address",
                        "description": "Information about the address",
                        "properties": {
                          "name": {
                            "description": "Full name",
                            "type": "string",
                            "example": "John Smith"
                          },
                          "company": {
                            "description": "Company name",
                            "type": "string",
                            "example": "John Smith Inc"
                          },
                          "address1": {
                            "description": "Address line 1",
                            "type": "string",
                            "example": "19749 Dearborn St"
                          },
                          "address2": {
                            "description": "Address line 2",
                            "type": "string"
                          },
                          "city": {
                            "description": "City",
                            "type": "string",
                            "example": "Chatsworth"
                          },
                          "state_code": {
                            "description": "State code",
                            "type": "string",
                            "example": "CA"
                          },
                          "state_name": {
                            "description": "State name",
                            "type": "string",
                            "example": "California"
                          },
                          "country_code": {
                            "description": "Country code",
                            "type": "string",
                            "example": "US"
                          },
                          "country_name": {
                            "description": "Country name",
                            "type": "string",
                            "example": "United States"
                          },
                          "zip": {
                            "description": "ZIP/Postal code",
                            "type": "string",
                            "example": "91311"
                          },
                          "phone": {
                            "description": "Phone number",
                            "example": "2312322334",
                            "type": "string"
                          },
                          "email": {
                            "description": "Email address",
                            "example": "firstname.secondname@domain.com",
                            "type": "string"
                          },
                          "tax_number": {
                            "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                            "type": "string",
                            "example": "123.456.789-10"
                          }
                        }
                      },
                      "items": {
                        "type": "array",
                        "description": "Array of items in the order",
                        "items": {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about an item in the order",
                          "properties": {
                            "id": {
                              "description": "Line item ID",
                              "type": "integer",
                              "example": 1
                            },
                            "external_id": {
                              "description": "Line item ID from the external system",
                              "type": "string",
                              "example": "item-1"
                            },
                            "variant_id": {
                              "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                              "type": "integer",
                              "example": 1
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                              "type": "string",
                              "example": "variant-1"
                            },
                            "warehouse_product_variant_id": {
                              "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                              "type": "integer",
                              "example": 1
                            },
                            "product_template_id": {
                              "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "integer",
                              "example": 1
                            },
                            "external_product_id": {
                              "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "string",
                              "example": "template-123",
                              "writeOnly": true
                            },
                            "quantity": {
                              "description": "Number of items ordered (Limited to 1000 for one item)",
                              "type": "integer",
                              "example": 1
                            },
                            "price": {
                              "description": "Printful price of the item",
                              "type": "string",
                              "example": "13.00"
                            },
                            "retail_price": {
                              "description": "Original retail price of the item to be displayed on the packing slip",
                              "type": "string",
                              "example": "13.00"
                            },
                            "name": {
                              "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                              "type": "string",
                              "example": "Enhanced Matte Paper Poster 18\u00d724"
                            },
                            "product": {
                              "type": "object",
                              "title": "ProductVariant",
                              "description": "Short information about the Printful Product and Variant",
                              "properties": {
                                "variant_id": {
                                  "type": "integer",
                                  "description": "Variant ID",
                                  "example": 3001
                                },
                                "product_id": {
                                  "type": "integer",
                                  "description": "Product ID of this variant",
                                  "example": 301
                                },
                                "image": {
                                  "type": "string",
                                  "description": "URL of a sample image for this variant",
                                  "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                                },
                                "name": {
                                  "type": "string",
                                  "description": "Display name of this variant",
                                  "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                                }
                              }
                            },
                            "files": {
                              "type": "array",
                              "items": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "File",
                                    "description": "Information about the File",
                                    "required": [
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "type": "string",
                                        "description": "Role of the file",
                                        "example": "default"
                                      },
                                      "id": {
                                        "type": "integer",
                                        "description": "File ID",
                                        "example": 10,
                                        "readOnly": true
                                      },
                                      "url": {
                                        "type": "string",
                                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                                      },
                                      "options": {
                                        "type": "array",
                                        "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                                        "items": {
                                          "type": "object",
                                          "title": "FileOption",
                                          "description": "File option",
                                          "required": [
                                            "id",
                                            "value"
                                          ],
                                          "properties": {
                                            "id": {
                                              "type": "string",
                                              "description": "Option id",
                                              "example": "template_type"
                                            },
                                            "value": {
                                              "type": "string",
                                              "description": "Option value",
                                              "example": "native"
                                            }
                                          }
                                        }
                                      },
                                      "hash": {
                                        "type": "string",
                                        "description": "MD5 checksum of the file",
                                        "example": "ea44330b887dfec278dbc4626a759547",
                                        "readOnly": true
                                      },
                                      "filename": {
                                        "type": "string",
                                        "description": "File name",
                                        "example": "shirt1.png"
                                      },
                                      "mime_type": {
                                        "type": "string",
                                        "description": "MIME type of the file",
                                        "example": "image/png",
                                        "readOnly": true
                                      },
                                      "size": {
                                        "type": "integer",
                                        "description": "Size in bytes",
                                        "example": 45582633,
                                        "readOnly": true
                                      },
                                      "width": {
                                        "type": "integer",
                                        "description": "Width in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "height": {
                                        "type": "integer",
                                        "description": "Height in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "dpi": {
                                        "type": "integer",
                                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                                        "example": 300,
                                        "readOnly": true
                                      },
                                      "status": {
                                        "type": "string",
                                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                                        "example": "ok",
                                        "readOnly": true
                                      },
                                      "created": {
                                        "type": "integer",
                                        "description": "File creation timestamp",
                                        "example": 1590051937,
                                        "readOnly": true
                                      },
                                      "thumbnail_url": {
                                        "type": "string",
                                        "description": "Small thumbnail URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "preview_url": {
                                        "type": "string",
                                        "description": "Medium preview image URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "visible": {
                                        "type": "boolean",
                                        "description": "Show file in the Printfile Library (default true)",
                                        "example": true
                                      },
                                      "is_temporary": {
                                        "type": "boolean",
                                        "readOnly": true,
                                        "description": "Whether it is a temporary printfile.",
                                        "example": false
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "properties": {
                                      "position": {
                                        "writeOnly": true,
                                        "allOf": [
                                          {
                                            "type": "object",
                                            "title": "GenerationTaskFilePosition",
                                            "description": "Position",
                                            "properties": {
                                              "area_width": {
                                                "type": "integer",
                                                "description": "Positioning area width on print area in pixels",
                                                "example": 1800,
                                                "nullable": true
                                              },
                                              "area_height": {
                                                "type": "integer",
                                                "description": "Positioning area height on print area in pixels",
                                                "example": 2400,
                                                "nullable": true
                                              },
                                              "width": {
                                                "type": "integer",
                                                "description": "Width of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "height": {
                                                "type": "integer",
                                                "description": "Height of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "top": {
                                                "type": "integer",
                                                "description": "Image top offset in given area in pixels",
                                                "example": 300
                                              },
                                              "left": {
                                                "type": "integer",
                                                "description": "Image left offset in given area in pixels",
                                                "example": 0
                                              }
                                            }
                                          },
                                          {
                                            "type": "object",
                                            "properties": {
                                              "limit_to_print_area": {
                                                "type": "boolean",
                                                "default": true,
                                                "description": "Should limit printfile to print area"
                                              }
                                            }
                                          }
                                        ]
                                      }
                                    }
                                  }
                                ]
                              },
                              "description": "Array of attached printfiles / preview images"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "ItemOption",
                                "description": "Additional option for order item",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option ID",
                                    "example": "OptionKey"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "OptionValue"
                                  }
                                }
                              }
                            },
                            "sku": {
                              "description": "Product identifier (SKU) from the external system",
                              "type": "string",
                              "example": null
                            },
                            "discontinued": {
                              "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                              "type": "boolean",
                              "example": true
                            },
                            "out_of_stock": {
                              "description": "Whether the item is out of stock i.e. temporarily unavailable",
                              "type": "boolean",
                              "example": true
                            }
                          },
                          "anyOf": [
                            {
                              "required": [
                                "variant_id"
                              ]
                            },
                            {
                              "required": [
                                "sync_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "external_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "warehouse_product_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "product_template_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "external_product_id"
                              ]
                            }
                          ],
                          "not": {
                            "anyOf": [
                              {
                                "required": [
                                  "product_template_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_product_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "product_template_id"
                                ]
                              }
                            ]
                          }
                        }
                      },
                      "branding_items": {
                        "type": "array",
                        "description": "Array of branding items in the order",
                        "readOnly": true,
                        "items": {
                          "$ref": "circular reference to #/components/schemas/Item"
                        }
                      },
                      "incomplete_items": {
                        "type": "array",
                        "description": "Array of incomplete items in the order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "IncompleteItem",
                          "description": "Information about an incomplete item in the order",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Incomplete item name",
                              "example": "Red T-Shirt"
                            },
                            "quantity": {
                              "description": "Incompleted item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the incompleted item.",
                              "type": "integer",
                              "example": 70
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the incompleted item.",
                              "type": "string",
                              "example": "external-id"
                            },
                            "external_line_item_id": {
                              "description": "External order line item id.",
                              "type": "string",
                              "example": "external-line-item-id"
                            }
                          }
                        }
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "OrderCosts",
                            "description": "Order costs (Printful prices)",
                            "properties": {
                              "currency": {
                                "type": "string",
                                "description": "3 letter currency code",
                                "example": "USD"
                              },
                              "subtotal": {
                                "type": "string",
                                "description": "Total cost of all items",
                                "example": "10.00"
                              },
                              "discount": {
                                "type": "string",
                                "description": "Discount sum",
                                "example": "0.00"
                              },
                              "shipping": {
                                "type": "string",
                                "description": "Shipping costs",
                                "example": "5.00"
                              },
                              "digitization": {
                                "type": "string",
                                "description": "Digitization costs",
                                "example": "0.00"
                              },
                              "additional_fee": {
                                "type": "string",
                                "description": "Additional fee for custom product",
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "type": "string",
                                "description": "Custom product fulfillment fee",
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "type": "string",
                                "description": "Retail delivery fee",
                                "example": "0.00"
                              },
                              "tax": {
                                "type": "string",
                                "description": "Sum of taxes (not included in the item price)",
                                "example": "0.00"
                              },
                              "vat": {
                                "type": "string",
                                "description": "Sum of vat (not included in the item price)",
                                "example": "0.00"
                              },
                              "total": {
                                "type": "string",
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                                "example": "15.00"
                              }
                            }
                          },
                          {
                            "readOnly": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "type": "object",
                        "title": "OrderRetailCosts",
                        "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "string",
                            "nullable": true,
                            "description": "Total cost of all items",
                            "example": "10.00"
                          },
                          "discount": {
                            "type": "string",
                            "nullable": true,
                            "description": "Discount sum",
                            "example": "0.00"
                          },
                          "shipping": {
                            "type": "string",
                            "nullable": true,
                            "description": "Shipping costs",
                            "example": "5.00"
                          },
                          "tax": {
                            "type": "string",
                            "nullable": true,
                            "description": "Sum of taxes (not included in the item price)",
                            "example": "0.00"
                          },
                          "vat": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Sum of VAT (not included in the item price)",
                            "example": "0.00"
                          },
                          "total": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": "15.00"
                          }
                        }
                      },
                      "pricing_breakdown": {
                        "type": "array",
                        "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "PricingBreakdown",
                          "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                          "properties": {
                            "customer_pays": {
                              "type": "string",
                              "description": "Amount customer paid",
                              "example": "3.75"
                            },
                            "printful_price": {
                              "type": "string",
                              "description": "Printful price",
                              "example": "6"
                            },
                            "profit": {
                              "type": "string",
                              "description": "Profit",
                              "example": "-2.25"
                            },
                            "currency_symbol": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": "USD"
                            }
                          }
                        }
                      },
                      "shipments": {
                        "type": "array",
                        "description": "Array of shipments already shipped for this order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "OrderShipment",
                          "description": "Information about order shipment",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Shipment ID",
                              "example": 10
                            },
                            "carrier": {
                              "type": "string",
                              "description": "Carrier name",
                              "example": "FEDEX"
                            },
                            "service": {
                              "type": "string",
                              "description": "Delivery service name",
                              "example": "FedEx SmartPost"
                            },
                            "tracking_number": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": 0
                            },
                            "tracking_url": {
                              "type": "string",
                              "description": "Shipment tracking URL",
                              "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                            },
                            "created": {
                              "type": "integer",
                              "description": "Shipping time",
                              "example": 1588716060
                            },
                            "ship_date": {
                              "type": "string",
                              "description": "Ship date",
                              "example": "2020-05-05"
                            },
                            "shipped_at": {
                              "type": "string",
                              "description": "Ship time in unix timestamp",
                              "example": 1588716060
                            },
                            "reshipment": {
                              "type": "boolean",
                              "description": "Whether this is a reshipment",
                              "example": false
                            },
                            "items": {
                              "type": "array",
                              "description": "Array of items in this shipment",
                              "items": {
                                "type": "object",
                                "title": "OrderShipmentItem",
                                "description": "Information about item in an order shipment",
                                "properties": {
                                  "item_id": {
                                    "type": "integer",
                                    "description": "Line item ID",
                                    "example": 1
                                  },
                                  "quantity": {
                                    "type": "integer",
                                    "description": "Quantity of items in this shipment",
                                    "example": 1
                                  },
                                  "picked": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                                    "example": 1
                                  },
                                  "printed": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                                    "example": 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "gift": {
                        "type": "object",
                        "title": "OrderGift",
                        "description": "Optional gift message for the packing slip",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message title",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message text",
                            "example": "Have a nice day"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "OrderPackingSlip",
                        "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                        "anyOf": [
                          {
                            "required": [
                              "email"
                            ]
                          },
                          {
                            "required": [
                              "phone"
                            ]
                          },
                          {
                            "required": [
                              "message"
                            ]
                          },
                          {
                            "required": [
                              "custom_order_id"
                            ]
                          }
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "your-name@your-domain.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+371 28888888"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "Message on packing slip"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "Your store name"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "kkk2344lm"
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    },
    "404": {
      "description": "Not found",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `404`",
                "example": 404
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Not found"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "NotFound"
                  },
                  "message": {
                    "type": "string",
                    "example": "Not found"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request DELETE 'https://api.printful.com/orders/{order_id}' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Cancel order with id 12345\n$order = $pf->delete('orders/12345');"
    }
  ]
}
```
</details>

---

### PUT /orders/{id}
**Summary:** Update order data

 Updates unsubmitted order and optionally submits it for the fulfillment.

 Note that you need to post only the fields that need to be changed, not all required fields.

 If items array is given in the update data, the items will be:

 a) updated, if the update data contains the item id or external_id parameter that alreay exists

 b) deleted, if the request doesn't contain the item with previously existing id

 c) created as new if the id is not given or does not already exist 

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "updateOrderById",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "oneOf": [
          {
            "type": "string"
          },
          {
            "type": "integer"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or External ID (if prefixed with @)"
    },
    {
      "in": "query",
      "name": "confirm",
      "schema": {
        "type": "boolean"
      },
      "required": false,
      "description": "Automatically submit the newly created order for fulfillment (skip the Draft phase)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "Order",
          "description": "Information about the Order",
          "required": [
            "recipient",
            "items"
          ],
          "properties": {
            "id": {
              "description": "Order ID",
              "type": "integer",
              "example": 13,
              "readOnly": true
            },
            "external_id": {
              "description": "Order ID from the external system",
              "type": "string",
              "nullable": true,
              "example": "4235234213"
            },
            "store": {
              "description": "Store ID",
              "type": "integer",
              "example": 10,
              "readOnly": true
            },
            "status": {
              "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
              "type": "string",
              "example": "draft",
              "readOnly": true
            },
            "shipping": {
              "description": "Shipping method. Defaults to 'STANDARD'",
              "type": "string",
              "example": "STANDARD"
            },
            "shipping_service_name": {
              "description": "Human readable shipping method name.",
              "type": "string",
              "example": "Flat Rate (3-4 business days after fulfillment)",
              "readOnly": true
            },
            "created": {
              "description": "Time when the order was created",
              "type": "integer",
              "example": 1602607640,
              "readOnly": true
            },
            "updated": {
              "description": "Time when the order was updated",
              "type": "integer",
              "example": 1602607640,
              "readOnly": true
            },
            "recipient": {
              "type": "object",
              "title": "Address",
              "description": "Information about the address",
              "properties": {
                "name": {
                  "description": "Full name",
                  "type": "string",
                  "example": "John Smith"
                },
                "company": {
                  "description": "Company name",
                  "type": "string",
                  "example": "John Smith Inc"
                },
                "address1": {
                  "description": "Address line 1",
                  "type": "string",
                  "example": "19749 Dearborn St"
                },
                "address2": {
                  "description": "Address line 2",
                  "type": "string"
                },
                "city": {
                  "description": "City",
                  "type": "string",
                  "example": "Chatsworth"
                },
                "state_code": {
                  "description": "State code",
                  "type": "string",
                  "example": "CA"
                },
                "state_name": {
                  "description": "State name",
                  "type": "string",
                  "example": "California"
                },
                "country_code": {
                  "description": "Country code",
                  "type": "string",
                  "example": "US"
                },
                "country_name": {
                  "description": "Country name",
                  "type": "string",
                  "example": "United States"
                },
                "zip": {
                  "description": "ZIP/Postal code",
                  "type": "string",
                  "example": "91311"
                },
                "phone": {
                  "description": "Phone number",
                  "example": "2312322334",
                  "type": "string"
                },
                "email": {
                  "description": "Email address",
                  "example": "firstname.secondname@domain.com",
                  "type": "string"
                },
                "tax_number": {
                  "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                  "type": "string",
                  "example": "123.456.789-10"
                }
              }
            },
            "items": {
              "type": "array",
              "description": "Array of items in the order",
              "items": {
                "type": "object",
                "title": "Item",
                "description": "Information about an item in the order",
                "properties": {
                  "id": {
                    "description": "Line item ID",
                    "type": "integer",
                    "example": 1
                  },
                  "external_id": {
                    "description": "Line item ID from the external system",
                    "type": "string",
                    "example": "item-1"
                  },
                  "variant_id": {
                    "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                    "type": "integer",
                    "example": 1
                  },
                  "sync_variant_id": {
                    "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                    "type": "integer",
                    "example": 1
                  },
                  "external_variant_id": {
                    "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                    "type": "string",
                    "example": "variant-1"
                  },
                  "warehouse_product_variant_id": {
                    "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                    "type": "integer",
                    "example": 1
                  },
                  "product_template_id": {
                    "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                    "type": "integer",
                    "example": 1
                  },
                  "external_product_id": {
                    "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                    "type": "string",
                    "example": "template-123",
                    "writeOnly": true
                  },
                  "quantity": {
                    "description": "Number of items ordered (Limited to 1000 for one item)",
                    "type": "integer",
                    "example": 1
                  },
                  "price": {
                    "description": "Printful price of the item",
                    "type": "string",
                    "example": "13.00"
                  },
                  "retail_price": {
                    "description": "Original retail price of the item to be displayed on the packing slip",
                    "type": "string",
                    "example": "13.00"
                  },
                  "name": {
                    "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                    "type": "string",
                    "example": "Enhanced Matte Paper Poster 18\u00d724"
                  },
                  "product": {
                    "type": "object",
                    "title": "ProductVariant",
                    "description": "Short information about the Printful Product and Variant",
                    "properties": {
                      "variant_id": {
                        "type": "integer",
                        "description": "Variant ID",
                        "example": 3001
                      },
                      "product_id": {
                        "type": "integer",
                        "description": "Product ID of this variant",
                        "example": 301
                      },
                      "image": {
                        "type": "string",
                        "description": "URL of a sample image for this variant",
                        "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                      },
                      "name": {
                        "type": "string",
                        "description": "Display name of this variant",
                        "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                      }
                    }
                  },
                  "files": {
                    "type": "array",
                    "items": {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "File",
                          "description": "Information about the File",
                          "required": [
                            "url"
                          ],
                          "properties": {
                            "type": {
                              "type": "string",
                              "description": "Role of the file",
                              "example": "default"
                            },
                            "id": {
                              "type": "integer",
                              "description": "File ID",
                              "example": 10,
                              "readOnly": true
                            },
                            "url": {
                              "type": "string",
                              "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                              "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "FileOption",
                                "description": "File option",
                                "required": [
                                  "id",
                                  "value"
                                ],
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option id",
                                    "example": "template_type"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "native"
                                  }
                                }
                              }
                            },
                            "hash": {
                              "type": "string",
                              "description": "MD5 checksum of the file",
                              "example": "ea44330b887dfec278dbc4626a759547",
                              "readOnly": true
                            },
                            "filename": {
                              "type": "string",
                              "description": "File name",
                              "example": "shirt1.png"
                            },
                            "mime_type": {
                              "type": "string",
                              "description": "MIME type of the file",
                              "example": "image/png",
                              "readOnly": true
                            },
                            "size": {
                              "type": "integer",
                              "description": "Size in bytes",
                              "example": 45582633,
                              "readOnly": true
                            },
                            "width": {
                              "type": "integer",
                              "description": "Width in pixels",
                              "example": 1000,
                              "readOnly": true
                            },
                            "height": {
                              "type": "integer",
                              "description": "Height in pixels",
                              "example": 1000,
                              "readOnly": true
                            },
                            "dpi": {
                              "type": "integer",
                              "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                              "example": 300,
                              "readOnly": true
                            },
                            "status": {
                              "type": "string",
                              "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                              "example": "ok",
                              "readOnly": true
                            },
                            "created": {
                              "type": "integer",
                              "description": "File creation timestamp",
                              "example": 1590051937,
                              "readOnly": true
                            },
                            "thumbnail_url": {
                              "type": "string",
                              "description": "Small thumbnail URL",
                              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                              "readOnly": true
                            },
                            "preview_url": {
                              "type": "string",
                              "description": "Medium preview image URL",
                              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                              "readOnly": true
                            },
                            "visible": {
                              "type": "boolean",
                              "description": "Show file in the Printfile Library (default true)",
                              "example": true
                            },
                            "is_temporary": {
                              "type": "boolean",
                              "readOnly": true,
                              "description": "Whether it is a temporary printfile.",
                              "example": false
                            }
                          }
                        },
                        {
                          "type": "object",
                          "properties": {
                            "position": {
                              "writeOnly": true,
                              "allOf": [
                                {
                                  "type": "object",
                                  "title": "GenerationTaskFilePosition",
                                  "description": "Position",
                                  "properties": {
                                    "area_width": {
                                      "type": "integer",
                                      "description": "Positioning area width on print area in pixels",
                                      "example": 1800,
                                      "nullable": true
                                    },
                                    "area_height": {
                                      "type": "integer",
                                      "description": "Positioning area height on print area in pixels",
                                      "example": 2400,
                                      "nullable": true
                                    },
                                    "width": {
                                      "type": "integer",
                                      "description": "Width of the image in given area in pixels",
                                      "example": 1800,
                                      "readOnly": false
                                    },
                                    "height": {
                                      "type": "integer",
                                      "description": "Height of the image in given area in pixels",
                                      "example": 1800,
                                      "readOnly": false
                                    },
                                    "top": {
                                      "type": "integer",
                                      "description": "Image top offset in given area in pixels",
                                      "example": 300
                                    },
                                    "left": {
                                      "type": "integer",
                                      "description": "Image left offset in given area in pixels",
                                      "example": 0
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "properties": {
                                    "limit_to_print_area": {
                                      "type": "boolean",
                                      "default": true,
                                      "description": "Should limit printfile to print area"
                                    }
                                  }
                                }
                              ]
                            }
                          }
                        }
                      ]
                    },
                    "description": "Array of attached printfiles / preview images"
                  },
                  "options": {
                    "type": "array",
                    "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                    "items": {
                      "type": "object",
                      "title": "ItemOption",
                      "description": "Additional option for order item",
                      "properties": {
                        "id": {
                          "type": "string",
                          "description": "Option ID",
                          "example": "OptionKey"
                        },
                        "value": {
                          "type": "string",
                          "description": "Option value",
                          "example": "OptionValue"
                        }
                      }
                    }
                  },
                  "sku": {
                    "description": "Product identifier (SKU) from the external system",
                    "type": "string",
                    "example": null
                  },
                  "discontinued": {
                    "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                    "type": "boolean",
                    "example": true
                  },
                  "out_of_stock": {
                    "description": "Whether the item is out of stock i.e. temporarily unavailable",
                    "type": "boolean",
                    "example": true
                  }
                },
                "anyOf": [
                  {
                    "required": [
                      "variant_id"
                    ]
                  },
                  {
                    "required": [
                      "sync_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "external_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "warehouse_product_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "variant_id",
                      "product_template_id"
                    ]
                  },
                  {
                    "required": [
                      "variant_id",
                      "external_product_id"
                    ]
                  }
                ],
                "not": {
                  "anyOf": [
                    {
                      "required": [
                        "product_template_id",
                        "sync_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "external_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "warehouse_product_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "files"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "options"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "external_product_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "sync_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "external_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "warehouse_product_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "files"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "options"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "product_template_id"
                      ]
                    }
                  ]
                }
              }
            },
            "branding_items": {
              "type": "array",
              "description": "Array of branding items in the order",
              "readOnly": true,
              "items": {
                "$ref": "circular reference to #/components/schemas/Item"
              }
            },
            "incomplete_items": {
              "type": "array",
              "description": "Array of incomplete items in the order",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "IncompleteItem",
                "description": "Information about an incomplete item in the order",
                "properties": {
                  "name": {
                    "type": "string",
                    "description": "Incomplete item name",
                    "example": "Red T-Shirt"
                  },
                  "quantity": {
                    "description": "Incompleted item quantity",
                    "type": "integer",
                    "example": 1
                  },
                  "sync_variant_id": {
                    "description": "Sync variant ID of the incompleted item.",
                    "type": "integer",
                    "example": 70
                  },
                  "external_variant_id": {
                    "description": "External variant ID of the incompleted item.",
                    "type": "string",
                    "example": "external-id"
                  },
                  "external_line_item_id": {
                    "description": "External order line item id.",
                    "type": "string",
                    "example": "external-line-item-id"
                  }
                }
              }
            },
            "costs": {
              "allOf": [
                {
                  "type": "object",
                  "title": "OrderCosts",
                  "description": "Order costs (Printful prices)",
                  "properties": {
                    "currency": {
                      "type": "string",
                      "description": "3 letter currency code",
                      "example": "USD"
                    },
                    "subtotal": {
                      "type": "string",
                      "description": "Total cost of all items",
                      "example": "10.00"
                    },
                    "discount": {
                      "type": "string",
                      "description": "Discount sum",
                      "example": "0.00"
                    },
                    "shipping": {
                      "type": "string",
                      "description": "Shipping costs",
                      "example": "5.00"
                    },
                    "digitization": {
                      "type": "string",
                      "description": "Digitization costs",
                      "example": "0.00"
                    },
                    "additional_fee": {
                      "type": "string",
                      "description": "Additional fee for custom product",
                      "example": "0.00"
                    },
                    "fulfillment_fee": {
                      "type": "string",
                      "description": "Custom product fulfillment fee",
                      "example": "0.00"
                    },
                    "retail_delivery_fee": {
                      "type": "string",
                      "description": "Retail delivery fee",
                      "example": "0.00"
                    },
                    "tax": {
                      "type": "string",
                      "description": "Sum of taxes (not included in the item price)",
                      "example": "0.00"
                    },
                    "vat": {
                      "type": "string",
                      "description": "Sum of vat (not included in the item price)",
                      "example": "0.00"
                    },
                    "total": {
                      "type": "string",
                      "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                      "example": "15.00"
                    }
                  }
                },
                {
                  "readOnly": true
                }
              ]
            },
            "retail_costs": {
              "type": "object",
              "title": "OrderRetailCosts",
              "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
              "properties": {
                "currency": {
                  "type": "string",
                  "description": "3 letter currency code",
                  "example": "USD"
                },
                "subtotal": {
                  "type": "string",
                  "nullable": true,
                  "description": "Total cost of all items",
                  "example": "10.00"
                },
                "discount": {
                  "type": "string",
                  "nullable": true,
                  "description": "Discount sum",
                  "example": "0.00"
                },
                "shipping": {
                  "type": "string",
                  "nullable": true,
                  "description": "Shipping costs",
                  "example": "5.00"
                },
                "tax": {
                  "type": "string",
                  "nullable": true,
                  "description": "Sum of taxes (not included in the item price)",
                  "example": "0.00"
                },
                "vat": {
                  "type": "string",
                  "nullable": true,
                  "readOnly": true,
                  "description": "Sum of VAT (not included in the item price)",
                  "example": "0.00"
                },
                "total": {
                  "type": "string",
                  "nullable": true,
                  "readOnly": true,
                  "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                  "example": "15.00"
                }
              }
            },
            "pricing_breakdown": {
              "type": "array",
              "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "PricingBreakdown",
                "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                "properties": {
                  "customer_pays": {
                    "type": "string",
                    "description": "Amount customer paid",
                    "example": "3.75"
                  },
                  "printful_price": {
                    "type": "string",
                    "description": "Printful price",
                    "example": "6"
                  },
                  "profit": {
                    "type": "string",
                    "description": "Profit",
                    "example": "-2.25"
                  },
                  "currency_symbol": {
                    "type": "string",
                    "description": "Shipment tracking number",
                    "example": "USD"
                  }
                }
              }
            },
            "shipments": {
              "type": "array",
              "description": "Array of shipments already shipped for this order",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "OrderShipment",
                "description": "Information about order shipment",
                "properties": {
                  "id": {
                    "type": "integer",
                    "description": "Shipment ID",
                    "example": 10
                  },
                  "carrier": {
                    "type": "string",
                    "description": "Carrier name",
                    "example": "FEDEX"
                  },
                  "service": {
                    "type": "string",
                    "description": "Delivery service name",
                    "example": "FedEx SmartPost"
                  },
                  "tracking_number": {
                    "type": "string",
                    "description": "Shipment tracking number",
                    "example": 0
                  },
                  "tracking_url": {
                    "type": "string",
                    "description": "Shipment tracking URL",
                    "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                  },
                  "created": {
                    "type": "integer",
                    "description": "Shipping time",
                    "example": 1588716060
                  },
                  "ship_date": {
                    "type": "string",
                    "description": "Ship date",
                    "example": "2020-05-05"
                  },
                  "shipped_at": {
                    "type": "string",
                    "description": "Ship time in unix timestamp",
                    "example": 1588716060
                  },
                  "reshipment": {
                    "type": "boolean",
                    "description": "Whether this is a reshipment",
                    "example": false
                  },
                  "items": {
                    "type": "array",
                    "description": "Array of items in this shipment",
                    "items": {
                      "type": "object",
                      "title": "OrderShipmentItem",
                      "description": "Information about item in an order shipment",
                      "properties": {
                        "item_id": {
                          "type": "integer",
                          "description": "Line item ID",
                          "example": 1
                        },
                        "quantity": {
                          "type": "integer",
                          "description": "Quantity of items in this shipment",
                          "example": 1
                        },
                        "picked": {
                          "type": "integer",
                          "enum": [
                            0,
                            1
                          ],
                          "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                          "example": 1
                        },
                        "printed": {
                          "type": "integer",
                          "enum": [
                            0,
                            1
                          ],
                          "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                          "example": 1
                        }
                      }
                    }
                  }
                }
              }
            },
            "gift": {
              "type": "object",
              "title": "OrderGift",
              "description": "Optional gift message for the packing slip",
              "properties": {
                "subject": {
                  "type": "string",
                  "maxLength": 200,
                  "description": "Gift message title",
                  "example": "To John"
                },
                "message": {
                  "type": "string",
                  "maxLength": 200,
                  "description": "Gift message text",
                  "example": "Have a nice day"
                }
              }
            },
            "packing_slip": {
              "type": "object",
              "title": "OrderPackingSlip",
              "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
              "anyOf": [
                {
                  "required": [
                    "email"
                  ]
                },
                {
                  "required": [
                    "phone"
                  ]
                },
                {
                  "required": [
                    "message"
                  ]
                },
                {
                  "required": [
                    "custom_order_id"
                  ]
                }
              ],
              "properties": {
                "email": {
                  "type": "string",
                  "description": "Customer service email",
                  "example": "your-name@your-domain.com"
                },
                "phone": {
                  "type": "string",
                  "description": "Customer service phone",
                  "example": "+371 28888888"
                },
                "message": {
                  "type": "string",
                  "description": "Custom packing slip message",
                  "example": "Message on packing slip"
                },
                "logo_url": {
                  "type": "string",
                  "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                  "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                },
                "store_name": {
                  "type": "string",
                  "description": "Store name override for the return address",
                  "example": "Your store name"
                },
                "custom_order_id": {
                  "type": "string",
                  "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                  "example": "kkk2344lm"
                }
              }
            }
          }
        }
      }
    }
  },
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "code": {
                    "type": "integer",
                    "description": "Response status code `200`",
                    "example": 200
                  }
                }
              },
              {
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "Order",
                    "description": "Information about the Order",
                    "required": [
                      "recipient",
                      "items"
                    ],
                    "properties": {
                      "id": {
                        "description": "Order ID",
                        "type": "integer",
                        "example": 13,
                        "readOnly": true
                      },
                      "external_id": {
                        "description": "Order ID from the external system",
                        "type": "string",
                        "nullable": true,
                        "example": "4235234213"
                      },
                      "store": {
                        "description": "Store ID",
                        "type": "integer",
                        "example": 10,
                        "readOnly": true
                      },
                      "status": {
                        "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
                        "type": "string",
                        "example": "draft",
                        "readOnly": true
                      },
                      "shipping": {
                        "description": "Shipping method. Defaults to 'STANDARD'",
                        "type": "string",
                        "example": "STANDARD"
                      },
                      "shipping_service_name": {
                        "description": "Human readable shipping method name.",
                        "type": "string",
                        "example": "Flat Rate (3-4 business days after fulfillment)",
                        "readOnly": true
                      },
                      "created": {
                        "description": "Time when the order was created",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "updated": {
                        "description": "Time when the order was updated",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "recipient": {
                        "type": "object",
                        "title": "Address",
                        "description": "Information about the address",
                        "properties": {
                          "name": {
                            "description": "Full name",
                            "type": "string",
                            "example": "John Smith"
                          },
                          "company": {
                            "description": "Company name",
                            "type": "string",
                            "example": "John Smith Inc"
                          },
                          "address1": {
                            "description": "Address line 1",
                            "type": "string",
                            "example": "19749 Dearborn St"
                          },
                          "address2": {
                            "description": "Address line 2",
                            "type": "string"
                          },
                          "city": {
                            "description": "City",
                            "type": "string",
                            "example": "Chatsworth"
                          },
                          "state_code": {
                            "description": "State code",
                            "type": "string",
                            "example": "CA"
                          },
                          "state_name": {
                            "description": "State name",
                            "type": "string",
                            "example": "California"
                          },
                          "country_code": {
                            "description": "Country code",
                            "type": "string",
                            "example": "US"
                          },
                          "country_name": {
                            "description": "Country name",
                            "type": "string",
                            "example": "United States"
                          },
                          "zip": {
                            "description": "ZIP/Postal code",
                            "type": "string",
                            "example": "91311"
                          },
                          "phone": {
                            "description": "Phone number",
                            "example": "2312322334",
                            "type": "string"
                          },
                          "email": {
                            "description": "Email address",
                            "example": "firstname.secondname@domain.com",
                            "type": "string"
                          },
                          "tax_number": {
                            "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                            "type": "string",
                            "example": "123.456.789-10"
                          }
                        }
                      },
                      "items": {
                        "type": "array",
                        "description": "Array of items in the order",
                        "items": {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about an item in the order",
                          "properties": {
                            "id": {
                              "description": "Line item ID",
                              "type": "integer",
                              "example": 1
                            },
                            "external_id": {
                              "description": "Line item ID from the external system",
                              "type": "string",
                              "example": "item-1"
                            },
                            "variant_id": {
                              "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                              "type": "integer",
                              "example": 1
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                              "type": "string",
                              "example": "variant-1"
                            },
                            "warehouse_product_variant_id": {
                              "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                              "type": "integer",
                              "example": 1
                            },
                            "product_template_id": {
                              "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "integer",
                              "example": 1
                            },
                            "external_product_id": {
                              "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "string",
                              "example": "template-123",
                              "writeOnly": true
                            },
                            "quantity": {
                              "description": "Number of items ordered (Limited to 1000 for one item)",
                              "type": "integer",
                              "example": 1
                            },
                            "price": {
                              "description": "Printful price of the item",
                              "type": "string",
                              "example": "13.00"
                            },
                            "retail_price": {
                              "description": "Original retail price of the item to be displayed on the packing slip",
                              "type": "string",
                              "example": "13.00"
                            },
                            "name": {
                              "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                              "type": "string",
                              "example": "Enhanced Matte Paper Poster 18\u00d724"
                            },
                            "product": {
                              "type": "object",
                              "title": "ProductVariant",
                              "description": "Short information about the Printful Product and Variant",
                              "properties": {
                                "variant_id": {
                                  "type": "integer",
                                  "description": "Variant ID",
                                  "example": 3001
                                },
                                "product_id": {
                                  "type": "integer",
                                  "description": "Product ID of this variant",
                                  "example": 301
                                },
                                "image": {
                                  "type": "string",
                                  "description": "URL of a sample image for this variant",
                                  "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                                },
                                "name": {
                                  "type": "string",
                                  "description": "Display name of this variant",
                                  "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                                }
                              }
                            },
                            "files": {
                              "type": "array",
                              "items": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "File",
                                    "description": "Information about the File",
                                    "required": [
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "type": "string",
                                        "description": "Role of the file",
                                        "example": "default"
                                      },
                                      "id": {
                                        "type": "integer",
                                        "description": "File ID",
                                        "example": 10,
                                        "readOnly": true
                                      },
                                      "url": {
                                        "type": "string",
                                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                                      },
                                      "options": {
                                        "type": "array",
                                        "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                                        "items": {
                                          "type": "object",
                                          "title": "FileOption",
                                          "description": "File option",
                                          "required": [
                                            "id",
                                            "value"
                                          ],
                                          "properties": {
                                            "id": {
                                              "type": "string",
                                              "description": "Option id",
                                              "example": "template_type"
                                            },
                                            "value": {
                                              "type": "string",
                                              "description": "Option value",
                                              "example": "native"
                                            }
                                          }
                                        }
                                      },
                                      "hash": {
                                        "type": "string",
                                        "description": "MD5 checksum of the file",
                                        "example": "ea44330b887dfec278dbc4626a759547",
                                        "readOnly": true
                                      },
                                      "filename": {
                                        "type": "string",
                                        "description": "File name",
                                        "example": "shirt1.png"
                                      },
                                      "mime_type": {
                                        "type": "string",
                                        "description": "MIME type of the file",
                                        "example": "image/png",
                                        "readOnly": true
                                      },
                                      "size": {
                                        "type": "integer",
                                        "description": "Size in bytes",
                                        "example": 45582633,
                                        "readOnly": true
                                      },
                                      "width": {
                                        "type": "integer",
                                        "description": "Width in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "height": {
                                        "type": "integer",
                                        "description": "Height in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "dpi": {
                                        "type": "integer",
                                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                                        "example": 300,
                                        "readOnly": true
                                      },
                                      "status": {
                                        "type": "string",
                                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                                        "example": "ok",
                                        "readOnly": true
                                      },
                                      "created": {
                                        "type": "integer",
                                        "description": "File creation timestamp",
                                        "example": 1590051937,
                                        "readOnly": true
                                      },
                                      "thumbnail_url": {
                                        "type": "string",
                                        "description": "Small thumbnail URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "preview_url": {
                                        "type": "string",
                                        "description": "Medium preview image URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "visible": {
                                        "type": "boolean",
                                        "description": "Show file in the Printfile Library (default true)",
                                        "example": true
                                      },
                                      "is_temporary": {
                                        "type": "boolean",
                                        "readOnly": true,
                                        "description": "Whether it is a temporary printfile.",
                                        "example": false
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "properties": {
                                      "position": {
                                        "writeOnly": true,
                                        "allOf": [
                                          {
                                            "type": "object",
                                            "title": "GenerationTaskFilePosition",
                                            "description": "Position",
                                            "properties": {
                                              "area_width": {
                                                "type": "integer",
                                                "description": "Positioning area width on print area in pixels",
                                                "example": 1800,
                                                "nullable": true
                                              },
                                              "area_height": {
                                                "type": "integer",
                                                "description": "Positioning area height on print area in pixels",
                                                "example": 2400,
                                                "nullable": true
                                              },
                                              "width": {
                                                "type": "integer",
                                                "description": "Width of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "height": {
                                                "type": "integer",
                                                "description": "Height of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "top": {
                                                "type": "integer",
                                                "description": "Image top offset in given area in pixels",
                                                "example": 300
                                              },
                                              "left": {
                                                "type": "integer",
                                                "description": "Image left offset in given area in pixels",
                                                "example": 0
                                              }
                                            }
                                          },
                                          {
                                            "type": "object",
                                            "properties": {
                                              "limit_to_print_area": {
                                                "type": "boolean",
                                                "default": true,
                                                "description": "Should limit printfile to print area"
                                              }
                                            }
                                          }
                                        ]
                                      }
                                    }
                                  }
                                ]
                              },
                              "description": "Array of attached printfiles / preview images"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "ItemOption",
                                "description": "Additional option for order item",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option ID",
                                    "example": "OptionKey"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "OptionValue"
                                  }
                                }
                              }
                            },
                            "sku": {
                              "description": "Product identifier (SKU) from the external system",
                              "type": "string",
                              "example": null
                            },
                            "discontinued": {
                              "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                              "type": "boolean",
                              "example": true
                            },
                            "out_of_stock": {
                              "description": "Whether the item is out of stock i.e. temporarily unavailable",
                              "type": "boolean",
                              "example": true
                            }
                          },
                          "anyOf": [
                            {
                              "required": [
                                "variant_id"
                              ]
                            },
                            {
                              "required": [
                                "sync_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "external_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "warehouse_product_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "product_template_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "external_product_id"
                              ]
                            }
                          ],
                          "not": {
                            "anyOf": [
                              {
                                "required": [
                                  "product_template_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_product_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "product_template_id"
                                ]
                              }
                            ]
                          }
                        }
                      },
                      "branding_items": {
                        "type": "array",
                        "description": "Array of branding items in the order",
                        "readOnly": true,
                        "items": {
                          "$ref": "circular reference to #/components/schemas/Item"
                        }
                      },
                      "incomplete_items": {
                        "type": "array",
                        "description": "Array of incomplete items in the order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "IncompleteItem",
                          "description": "Information about an incomplete item in the order",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Incomplete item name",
                              "example": "Red T-Shirt"
                            },
                            "quantity": {
                              "description": "Incompleted item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the incompleted item.",
                              "type": "integer",
                              "example": 70
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the incompleted item.",
                              "type": "string",
                              "example": "external-id"
                            },
                            "external_line_item_id": {
                              "description": "External order line item id.",
                              "type": "string",
                              "example": "external-line-item-id"
                            }
                          }
                        }
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "OrderCosts",
                            "description": "Order costs (Printful prices)",
                            "properties": {
                              "currency": {
                                "type": "string",
                                "description": "3 letter currency code",
                                "example": "USD"
                              },
                              "subtotal": {
                                "type": "string",
                                "description": "Total cost of all items",
                                "example": "10.00"
                              },
                              "discount": {
                                "type": "string",
                                "description": "Discount sum",
                                "example": "0.00"
                              },
                              "shipping": {
                                "type": "string",
                                "description": "Shipping costs",
                                "example": "5.00"
                              },
                              "digitization": {
                                "type": "string",
                                "description": "Digitization costs",
                                "example": "0.00"
                              },
                              "additional_fee": {
                                "type": "string",
                                "description": "Additional fee for custom product",
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "type": "string",
                                "description": "Custom product fulfillment fee",
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "type": "string",
                                "description": "Retail delivery fee",
                                "example": "0.00"
                              },
                              "tax": {
                                "type": "string",
                                "description": "Sum of taxes (not included in the item price)",
                                "example": "0.00"
                              },
                              "vat": {
                                "type": "string",
                                "description": "Sum of vat (not included in the item price)",
                                "example": "0.00"
                              },
                              "total": {
                                "type": "string",
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                                "example": "15.00"
                              }
                            }
                          },
                          {
                            "readOnly": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "type": "object",
                        "title": "OrderRetailCosts",
                        "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "string",
                            "nullable": true,
                            "description": "Total cost of all items",
                            "example": "10.00"
                          },
                          "discount": {
                            "type": "string",
                            "nullable": true,
                            "description": "Discount sum",
                            "example": "0.00"
                          },
                          "shipping": {
                            "type": "string",
                            "nullable": true,
                            "description": "Shipping costs",
                            "example": "5.00"
                          },
                          "tax": {
                            "type": "string",
                            "nullable": true,
                            "description": "Sum of taxes (not included in the item price)",
                            "example": "0.00"
                          },
                          "vat": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Sum of VAT (not included in the item price)",
                            "example": "0.00"
                          },
                          "total": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": "15.00"
                          }
                        }
                      },
                      "pricing_breakdown": {
                        "type": "array",
                        "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "PricingBreakdown",
                          "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                          "properties": {
                            "customer_pays": {
                              "type": "string",
                              "description": "Amount customer paid",
                              "example": "3.75"
                            },
                            "printful_price": {
                              "type": "string",
                              "description": "Printful price",
                              "example": "6"
                            },
                            "profit": {
                              "type": "string",
                              "description": "Profit",
                              "example": "-2.25"
                            },
                            "currency_symbol": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": "USD"
                            }
                          }
                        }
                      },
                      "shipments": {
                        "type": "array",
                        "description": "Array of shipments already shipped for this order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "OrderShipment",
                          "description": "Information about order shipment",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Shipment ID",
                              "example": 10
                            },
                            "carrier": {
                              "type": "string",
                              "description": "Carrier name",
                              "example": "FEDEX"
                            },
                            "service": {
                              "type": "string",
                              "description": "Delivery service name",
                              "example": "FedEx SmartPost"
                            },
                            "tracking_number": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": 0
                            },
                            "tracking_url": {
                              "type": "string",
                              "description": "Shipment tracking URL",
                              "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                            },
                            "created": {
                              "type": "integer",
                              "description": "Shipping time",
                              "example": 1588716060
                            },
                            "ship_date": {
                              "type": "string",
                              "description": "Ship date",
                              "example": "2020-05-05"
                            },
                            "shipped_at": {
                              "type": "string",
                              "description": "Ship time in unix timestamp",
                              "example": 1588716060
                            },
                            "reshipment": {
                              "type": "boolean",
                              "description": "Whether this is a reshipment",
                              "example": false
                            },
                            "items": {
                              "type": "array",
                              "description": "Array of items in this shipment",
                              "items": {
                                "type": "object",
                                "title": "OrderShipmentItem",
                                "description": "Information about item in an order shipment",
                                "properties": {
                                  "item_id": {
                                    "type": "integer",
                                    "description": "Line item ID",
                                    "example": 1
                                  },
                                  "quantity": {
                                    "type": "integer",
                                    "description": "Quantity of items in this shipment",
                                    "example": 1
                                  },
                                  "picked": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                                    "example": 1
                                  },
                                  "printed": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                                    "example": 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "gift": {
                        "type": "object",
                        "title": "OrderGift",
                        "description": "Optional gift message for the packing slip",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message title",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message text",
                            "example": "Have a nice day"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "OrderPackingSlip",
                        "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                        "anyOf": [
                          {
                            "required": [
                              "email"
                            ]
                          },
                          {
                            "required": [
                              "phone"
                            ]
                          },
                          {
                            "required": [
                              "message"
                            ]
                          },
                          {
                            "required": [
                              "custom_order_id"
                            ]
                          }
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "your-name@your-domain.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+371 28888888"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "Message on packing slip"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "Your store name"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "kkk2344lm"
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "400": {
      "description": "Bad Request",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `400`",
                "example": 400
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Missing required parameters"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Missing required parameters"
                  }
                }
              }
            }
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    },
    "404": {
      "description": "Not found",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `404`",
                "example": 404
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Not found"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "NotFound"
                  },
                  "message": {
                    "type": "string",
                    "example": "Not found"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request PUT 'https://api.printful.com/orders/{order_id}' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"items\": [\n        {\n            \"id\": 26030055,\n            \"files\": [\n                {\n                    \"url\": \"https://picsum.photos/200/200?random=2\"\n                }\n            ]\n        }\n    ]\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Change recipient address for order 12345\n$order = $pf->put('orders/12345', [\n    'recipient' => [\n        'name' => 'John Doe',\n        'address1' => '172 W Street Ave #105',\n        'city' => 'Burbank',\n        'state_code' => 'CA',\n        'country_code' => 'US',\n        'zip' => '91502',\n    ],\n]);\n"
    }
  ]
}
```
</details>

---

### POST /orders/{id}/confirm
**Summary:** Confirm draft for fulfillment

Approves for fulfillment an order that was saved as a draft. Store owner's credit card is charged when the order is submitted for fulfillment.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "confirmOrderById",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "oneOf": [
          {
            "type": "string"
          },
          {
            "type": "integer"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or External ID (if prefixed with @)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "code": {
                    "type": "integer",
                    "description": "Response status code `200`",
                    "example": 200
                  }
                }
              },
              {
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "Order",
                    "description": "Information about the Order",
                    "required": [
                      "recipient",
                      "items"
                    ],
                    "properties": {
                      "id": {
                        "description": "Order ID",
                        "type": "integer",
                        "example": 13,
                        "readOnly": true
                      },
                      "external_id": {
                        "description": "Order ID from the external system",
                        "type": "string",
                        "nullable": true,
                        "example": "4235234213"
                      },
                      "store": {
                        "description": "Store ID",
                        "type": "integer",
                        "example": 10,
                        "readOnly": true
                      },
                      "status": {
                        "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
                        "type": "string",
                        "example": "draft",
                        "readOnly": true
                      },
                      "shipping": {
                        "description": "Shipping method. Defaults to 'STANDARD'",
                        "type": "string",
                        "example": "STANDARD"
                      },
                      "shipping_service_name": {
                        "description": "Human readable shipping method name.",
                        "type": "string",
                        "example": "Flat Rate (3-4 business days after fulfillment)",
                        "readOnly": true
                      },
                      "created": {
                        "description": "Time when the order was created",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "updated": {
                        "description": "Time when the order was updated",
                        "type": "integer",
                        "example": 1602607640,
                        "readOnly": true
                      },
                      "recipient": {
                        "type": "object",
                        "title": "Address",
                        "description": "Information about the address",
                        "properties": {
                          "name": {
                            "description": "Full name",
                            "type": "string",
                            "example": "John Smith"
                          },
                          "company": {
                            "description": "Company name",
                            "type": "string",
                            "example": "John Smith Inc"
                          },
                          "address1": {
                            "description": "Address line 1",
                            "type": "string",
                            "example": "19749 Dearborn St"
                          },
                          "address2": {
                            "description": "Address line 2",
                            "type": "string"
                          },
                          "city": {
                            "description": "City",
                            "type": "string",
                            "example": "Chatsworth"
                          },
                          "state_code": {
                            "description": "State code",
                            "type": "string",
                            "example": "CA"
                          },
                          "state_name": {
                            "description": "State name",
                            "type": "string",
                            "example": "California"
                          },
                          "country_code": {
                            "description": "Country code",
                            "type": "string",
                            "example": "US"
                          },
                          "country_name": {
                            "description": "Country name",
                            "type": "string",
                            "example": "United States"
                          },
                          "zip": {
                            "description": "ZIP/Postal code",
                            "type": "string",
                            "example": "91311"
                          },
                          "phone": {
                            "description": "Phone number",
                            "example": "2312322334",
                            "type": "string"
                          },
                          "email": {
                            "description": "Email address",
                            "example": "firstname.secondname@domain.com",
                            "type": "string"
                          },
                          "tax_number": {
                            "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                            "type": "string",
                            "example": "123.456.789-10"
                          }
                        }
                      },
                      "items": {
                        "type": "array",
                        "description": "Array of items in the order",
                        "items": {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about an item in the order",
                          "properties": {
                            "id": {
                              "description": "Line item ID",
                              "type": "integer",
                              "example": 1
                            },
                            "external_id": {
                              "description": "Line item ID from the external system",
                              "type": "string",
                              "example": "item-1"
                            },
                            "variant_id": {
                              "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                              "type": "integer",
                              "example": 1
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                              "type": "string",
                              "example": "variant-1"
                            },
                            "warehouse_product_variant_id": {
                              "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                              "type": "integer",
                              "example": 1
                            },
                            "product_template_id": {
                              "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "integer",
                              "example": 1
                            },
                            "external_product_id": {
                              "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                              "type": "string",
                              "example": "template-123",
                              "writeOnly": true
                            },
                            "quantity": {
                              "description": "Number of items ordered (Limited to 1000 for one item)",
                              "type": "integer",
                              "example": 1
                            },
                            "price": {
                              "description": "Printful price of the item",
                              "type": "string",
                              "example": "13.00"
                            },
                            "retail_price": {
                              "description": "Original retail price of the item to be displayed on the packing slip",
                              "type": "string",
                              "example": "13.00"
                            },
                            "name": {
                              "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                              "type": "string",
                              "example": "Enhanced Matte Paper Poster 18\u00d724"
                            },
                            "product": {
                              "type": "object",
                              "title": "ProductVariant",
                              "description": "Short information about the Printful Product and Variant",
                              "properties": {
                                "variant_id": {
                                  "type": "integer",
                                  "description": "Variant ID",
                                  "example": 3001
                                },
                                "product_id": {
                                  "type": "integer",
                                  "description": "Product ID of this variant",
                                  "example": 301
                                },
                                "image": {
                                  "type": "string",
                                  "description": "URL of a sample image for this variant",
                                  "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                                },
                                "name": {
                                  "type": "string",
                                  "description": "Display name of this variant",
                                  "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                                }
                              }
                            },
                            "files": {
                              "type": "array",
                              "items": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "File",
                                    "description": "Information about the File",
                                    "required": [
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "type": "string",
                                        "description": "Role of the file",
                                        "example": "default"
                                      },
                                      "id": {
                                        "type": "integer",
                                        "description": "File ID",
                                        "example": 10,
                                        "readOnly": true
                                      },
                                      "url": {
                                        "type": "string",
                                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                                      },
                                      "options": {
                                        "type": "array",
                                        "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                                        "items": {
                                          "type": "object",
                                          "title": "FileOption",
                                          "description": "File option",
                                          "required": [
                                            "id",
                                            "value"
                                          ],
                                          "properties": {
                                            "id": {
                                              "type": "string",
                                              "description": "Option id",
                                              "example": "template_type"
                                            },
                                            "value": {
                                              "type": "string",
                                              "description": "Option value",
                                              "example": "native"
                                            }
                                          }
                                        }
                                      },
                                      "hash": {
                                        "type": "string",
                                        "description": "MD5 checksum of the file",
                                        "example": "ea44330b887dfec278dbc4626a759547",
                                        "readOnly": true
                                      },
                                      "filename": {
                                        "type": "string",
                                        "description": "File name",
                                        "example": "shirt1.png"
                                      },
                                      "mime_type": {
                                        "type": "string",
                                        "description": "MIME type of the file",
                                        "example": "image/png",
                                        "readOnly": true
                                      },
                                      "size": {
                                        "type": "integer",
                                        "description": "Size in bytes",
                                        "example": 45582633,
                                        "readOnly": true
                                      },
                                      "width": {
                                        "type": "integer",
                                        "description": "Width in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "height": {
                                        "type": "integer",
                                        "description": "Height in pixels",
                                        "example": 1000,
                                        "readOnly": true
                                      },
                                      "dpi": {
                                        "type": "integer",
                                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                                        "example": 300,
                                        "readOnly": true
                                      },
                                      "status": {
                                        "type": "string",
                                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                                        "example": "ok",
                                        "readOnly": true
                                      },
                                      "created": {
                                        "type": "integer",
                                        "description": "File creation timestamp",
                                        "example": 1590051937,
                                        "readOnly": true
                                      },
                                      "thumbnail_url": {
                                        "type": "string",
                                        "description": "Small thumbnail URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "preview_url": {
                                        "type": "string",
                                        "description": "Medium preview image URL",
                                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                                        "readOnly": true
                                      },
                                      "visible": {
                                        "type": "boolean",
                                        "description": "Show file in the Printfile Library (default true)",
                                        "example": true
                                      },
                                      "is_temporary": {
                                        "type": "boolean",
                                        "readOnly": true,
                                        "description": "Whether it is a temporary printfile.",
                                        "example": false
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "properties": {
                                      "position": {
                                        "writeOnly": true,
                                        "allOf": [
                                          {
                                            "type": "object",
                                            "title": "GenerationTaskFilePosition",
                                            "description": "Position",
                                            "properties": {
                                              "area_width": {
                                                "type": "integer",
                                                "description": "Positioning area width on print area in pixels",
                                                "example": 1800,
                                                "nullable": true
                                              },
                                              "area_height": {
                                                "type": "integer",
                                                "description": "Positioning area height on print area in pixels",
                                                "example": 2400,
                                                "nullable": true
                                              },
                                              "width": {
                                                "type": "integer",
                                                "description": "Width of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "height": {
                                                "type": "integer",
                                                "description": "Height of the image in given area in pixels",
                                                "example": 1800,
                                                "readOnly": false
                                              },
                                              "top": {
                                                "type": "integer",
                                                "description": "Image top offset in given area in pixels",
                                                "example": 300
                                              },
                                              "left": {
                                                "type": "integer",
                                                "description": "Image left offset in given area in pixels",
                                                "example": 0
                                              }
                                            }
                                          },
                                          {
                                            "type": "object",
                                            "properties": {
                                              "limit_to_print_area": {
                                                "type": "boolean",
                                                "default": true,
                                                "description": "Should limit printfile to print area"
                                              }
                                            }
                                          }
                                        ]
                                      }
                                    }
                                  }
                                ]
                              },
                              "description": "Array of attached printfiles / preview images"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "ItemOption",
                                "description": "Additional option for order item",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option ID",
                                    "example": "OptionKey"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "OptionValue"
                                  }
                                }
                              }
                            },
                            "sku": {
                              "description": "Product identifier (SKU) from the external system",
                              "type": "string",
                              "example": null
                            },
                            "discontinued": {
                              "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                              "type": "boolean",
                              "example": true
                            },
                            "out_of_stock": {
                              "description": "Whether the item is out of stock i.e. temporarily unavailable",
                              "type": "boolean",
                              "example": true
                            }
                          },
                          "anyOf": [
                            {
                              "required": [
                                "variant_id"
                              ]
                            },
                            {
                              "required": [
                                "sync_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "external_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "warehouse_product_variant_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "product_template_id"
                              ]
                            },
                            {
                              "required": [
                                "variant_id",
                                "external_product_id"
                              ]
                            }
                          ],
                          "not": {
                            "anyOf": [
                              {
                                "required": [
                                  "product_template_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "product_template_id",
                                  "external_product_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "sync_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "external_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "warehouse_product_variant_id"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "files"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "options"
                                ]
                              },
                              {
                                "required": [
                                  "external_product_id",
                                  "product_template_id"
                                ]
                              }
                            ]
                          }
                        }
                      },
                      "branding_items": {
                        "type": "array",
                        "description": "Array of branding items in the order",
                        "readOnly": true,
                        "items": {
                          "$ref": "circular reference to #/components/schemas/Item"
                        }
                      },
                      "incomplete_items": {
                        "type": "array",
                        "description": "Array of incomplete items in the order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "IncompleteItem",
                          "description": "Information about an incomplete item in the order",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Incomplete item name",
                              "example": "Red T-Shirt"
                            },
                            "quantity": {
                              "description": "Incompleted item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "sync_variant_id": {
                              "description": "Sync variant ID of the incompleted item.",
                              "type": "integer",
                              "example": 70
                            },
                            "external_variant_id": {
                              "description": "External variant ID of the incompleted item.",
                              "type": "string",
                              "example": "external-id"
                            },
                            "external_line_item_id": {
                              "description": "External order line item id.",
                              "type": "string",
                              "example": "external-line-item-id"
                            }
                          }
                        }
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "OrderCosts",
                            "description": "Order costs (Printful prices)",
                            "properties": {
                              "currency": {
                                "type": "string",
                                "description": "3 letter currency code",
                                "example": "USD"
                              },
                              "subtotal": {
                                "type": "string",
                                "description": "Total cost of all items",
                                "example": "10.00"
                              },
                              "discount": {
                                "type": "string",
                                "description": "Discount sum",
                                "example": "0.00"
                              },
                              "shipping": {
                                "type": "string",
                                "description": "Shipping costs",
                                "example": "5.00"
                              },
                              "digitization": {
                                "type": "string",
                                "description": "Digitization costs",
                                "example": "0.00"
                              },
                              "additional_fee": {
                                "type": "string",
                                "description": "Additional fee for custom product",
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "type": "string",
                                "description": "Custom product fulfillment fee",
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "type": "string",
                                "description": "Retail delivery fee",
                                "example": "0.00"
                              },
                              "tax": {
                                "type": "string",
                                "description": "Sum of taxes (not included in the item price)",
                                "example": "0.00"
                              },
                              "vat": {
                                "type": "string",
                                "description": "Sum of vat (not included in the item price)",
                                "example": "0.00"
                              },
                              "total": {
                                "type": "string",
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                                "example": "15.00"
                              }
                            }
                          },
                          {
                            "readOnly": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "type": "object",
                        "title": "OrderRetailCosts",
                        "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "string",
                            "nullable": true,
                            "description": "Total cost of all items",
                            "example": "10.00"
                          },
                          "discount": {
                            "type": "string",
                            "nullable": true,
                            "description": "Discount sum",
                            "example": "0.00"
                          },
                          "shipping": {
                            "type": "string",
                            "nullable": true,
                            "description": "Shipping costs",
                            "example": "5.00"
                          },
                          "tax": {
                            "type": "string",
                            "nullable": true,
                            "description": "Sum of taxes (not included in the item price)",
                            "example": "0.00"
                          },
                          "vat": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Sum of VAT (not included in the item price)",
                            "example": "0.00"
                          },
                          "total": {
                            "type": "string",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": "15.00"
                          }
                        }
                      },
                      "pricing_breakdown": {
                        "type": "array",
                        "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "PricingBreakdown",
                          "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                          "properties": {
                            "customer_pays": {
                              "type": "string",
                              "description": "Amount customer paid",
                              "example": "3.75"
                            },
                            "printful_price": {
                              "type": "string",
                              "description": "Printful price",
                              "example": "6"
                            },
                            "profit": {
                              "type": "string",
                              "description": "Profit",
                              "example": "-2.25"
                            },
                            "currency_symbol": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": "USD"
                            }
                          }
                        }
                      },
                      "shipments": {
                        "type": "array",
                        "description": "Array of shipments already shipped for this order",
                        "readOnly": true,
                        "items": {
                          "type": "object",
                          "title": "OrderShipment",
                          "description": "Information about order shipment",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Shipment ID",
                              "example": 10
                            },
                            "carrier": {
                              "type": "string",
                              "description": "Carrier name",
                              "example": "FEDEX"
                            },
                            "service": {
                              "type": "string",
                              "description": "Delivery service name",
                              "example": "FedEx SmartPost"
                            },
                            "tracking_number": {
                              "type": "string",
                              "description": "Shipment tracking number",
                              "example": 0
                            },
                            "tracking_url": {
                              "type": "string",
                              "description": "Shipment tracking URL",
                              "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                            },
                            "created": {
                              "type": "integer",
                              "description": "Shipping time",
                              "example": 1588716060
                            },
                            "ship_date": {
                              "type": "string",
                              "description": "Ship date",
                              "example": "2020-05-05"
                            },
                            "shipped_at": {
                              "type": "string",
                              "description": "Ship time in unix timestamp",
                              "example": 1588716060
                            },
                            "reshipment": {
                              "type": "boolean",
                              "description": "Whether this is a reshipment",
                              "example": false
                            },
                            "items": {
                              "type": "array",
                              "description": "Array of items in this shipment",
                              "items": {
                                "type": "object",
                                "title": "OrderShipmentItem",
                                "description": "Information about item in an order shipment",
                                "properties": {
                                  "item_id": {
                                    "type": "integer",
                                    "description": "Line item ID",
                                    "example": 1
                                  },
                                  "quantity": {
                                    "type": "integer",
                                    "description": "Quantity of items in this shipment",
                                    "example": 1
                                  },
                                  "picked": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                                    "example": 1
                                  },
                                  "printed": {
                                    "type": "integer",
                                    "enum": [
                                      0,
                                      1
                                    ],
                                    "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                                    "example": 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "gift": {
                        "type": "object",
                        "title": "OrderGift",
                        "description": "Optional gift message for the packing slip",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message title",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "maxLength": 200,
                            "description": "Gift message text",
                            "example": "Have a nice day"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "OrderPackingSlip",
                        "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                        "anyOf": [
                          {
                            "required": [
                              "email"
                            ]
                          },
                          {
                            "required": [
                              "phone"
                            ]
                          },
                          {
                            "required": [
                              "message"
                            ]
                          },
                          {
                            "required": [
                              "custom_order_id"
                            ]
                          }
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "your-name@your-domain.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+371 28888888"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "Message on packing slip"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "Your store name"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "kkk2344lm"
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    },
    "404": {
      "description": "Not found",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `404`",
                "example": 404
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Not found"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "NotFound"
                  },
                  "message": {
                    "type": "string",
                    "example": "Not found"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/orders/{order_id}/confirm' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Confirm order with id 12345\n$order = $pf->post('orders/12345/confirm');\n"
    }
  ]
}
```
</details>

---

### POST /orders/estimate-costs
**Summary:** Estimate order costs

Calculates the estimated order costs including item costs, print costs (back prints, inside labels etc.), shipping and taxes

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "estimateOrderCosts",
  "security": [
    {
      "OAuth": [
        "orders/read"
      ]
    }
  ],
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "Order",
          "description": "Information about the Order",
          "required": [
            "recipient",
            "items"
          ],
          "properties": {
            "id": {
              "description": "Order ID",
              "type": "integer",
              "example": 13,
              "readOnly": true
            },
            "external_id": {
              "description": "Order ID from the external system",
              "type": "string",
              "nullable": true,
              "example": "4235234213"
            },
            "store": {
              "description": "Store ID",
              "type": "integer",
              "example": 10,
              "readOnly": true
            },
            "status": {
              "description": " Order status:<br /> **draft** - order is not submitted for fulfillment<br /> **failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br /> **inreview** - order is being reviewed<br /> **pending** - order has been submitted for fulfillment<br /> **canceled** - order is canceled<br /> **onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service **inprocess** - order is being fulfilled and is no longer cancellable<br /> **partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br /> **fulfilled** - all items are shipped<br /> ",
              "type": "string",
              "example": "draft",
              "readOnly": true
            },
            "shipping": {
              "description": "Shipping method. Defaults to 'STANDARD'",
              "type": "string",
              "example": "STANDARD"
            },
            "shipping_service_name": {
              "description": "Human readable shipping method name.",
              "type": "string",
              "example": "Flat Rate (3-4 business days after fulfillment)",
              "readOnly": true
            },
            "created": {
              "description": "Time when the order was created",
              "type": "integer",
              "example": 1602607640,
              "readOnly": true
            },
            "updated": {
              "description": "Time when the order was updated",
              "type": "integer",
              "example": 1602607640,
              "readOnly": true
            },
            "recipient": {
              "type": "object",
              "title": "Address",
              "description": "Information about the address",
              "properties": {
                "name": {
                  "description": "Full name",
                  "type": "string",
                  "example": "John Smith"
                },
                "company": {
                  "description": "Company name",
                  "type": "string",
                  "example": "John Smith Inc"
                },
                "address1": {
                  "description": "Address line 1",
                  "type": "string",
                  "example": "19749 Dearborn St"
                },
                "address2": {
                  "description": "Address line 2",
                  "type": "string"
                },
                "city": {
                  "description": "City",
                  "type": "string",
                  "example": "Chatsworth"
                },
                "state_code": {
                  "description": "State code",
                  "type": "string",
                  "example": "CA"
                },
                "state_name": {
                  "description": "State name",
                  "type": "string",
                  "example": "California"
                },
                "country_code": {
                  "description": "Country code",
                  "type": "string",
                  "example": "US"
                },
                "country_name": {
                  "description": "Country name",
                  "type": "string",
                  "example": "United States"
                },
                "zip": {
                  "description": "ZIP/Postal code",
                  "type": "string",
                  "example": "91311"
                },
                "phone": {
                  "description": "Phone number",
                  "example": "2312322334",
                  "type": "string"
                },
                "email": {
                  "description": "Email address",
                  "example": "firstname.secondname@domain.com",
                  "type": "string"
                },
                "tax_number": {
                  "description": "TAX number (`optional`, but in case of Brazil country this field becomes `required` and will be used as CPF/CNPJ number)<br> CPF format is 000.000.000-00 (14 characters);<br> CNPJ format is 00.000.000/0000-00 (18 characters).",
                  "type": "string",
                  "example": "123.456.789-10"
                }
              }
            },
            "items": {
              "type": "array",
              "description": "Array of items in the order",
              "items": {
                "type": "object",
                "title": "Item",
                "description": "Information about an item in the order",
                "properties": {
                  "id": {
                    "description": "Line item ID",
                    "type": "integer",
                    "example": 1
                  },
                  "external_id": {
                    "description": "Line item ID from the external system",
                    "type": "string",
                    "example": "item-1"
                  },
                  "variant_id": {
                    "description": "Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API)",
                    "type": "integer",
                    "example": 1
                  },
                  "sync_variant_id": {
                    "description": "Sync variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-a-sync-variant).",
                    "type": "integer",
                    "example": 1
                  },
                  "external_variant_id": {
                    "description": "External variant ID of the item ordered. [Example](#tag/Examples/Orders-API-examples/Using-sync-variant-with-external-ID).",
                    "type": "string",
                    "example": "variant-1"
                  },
                  "warehouse_product_variant_id": {
                    "description": "Warehousing product variant ID of the item ordered. See Warehouse Products API",
                    "type": "integer",
                    "example": 1
                  },
                  "product_template_id": {
                    "description": "The ID of a Product Template to generate the printfiles from. The `variant_id` field must be passed as well.\nCan't be combined with following fields: `sync_variant_id`, `external_variant_id`, `warehouse_product_variant_id`,\n`files`, `options`, `external_product_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                    "type": "integer",
                    "example": 1
                  },
                  "external_product_id": {
                    "description": "The External Product ID associated with a Product Template to generate the printfiles from.\nThe `variant_id` field must be passed as well. Can't be combined with following fields: `sync_variant_id`,\n`external_variant_id`, `warehouse_product_variant_id`, `files`, `options`, `product_template_id`. [Examples](#tag/Examples/Orders-API-examples/Using-a-product-template).\n",
                    "type": "string",
                    "example": "template-123",
                    "writeOnly": true
                  },
                  "quantity": {
                    "description": "Number of items ordered (Limited to 1000 for one item)",
                    "type": "integer",
                    "example": 1
                  },
                  "price": {
                    "description": "Printful price of the item",
                    "type": "string",
                    "example": "13.00"
                  },
                  "retail_price": {
                    "description": "Original retail price of the item to be displayed on the packing slip",
                    "type": "string",
                    "example": "13.00"
                  },
                  "name": {
                    "description": "Display name of the item. If not given, a name from the Printful system will be displayed on the packing slip",
                    "type": "string",
                    "example": "Enhanced Matte Paper Poster 18\u00d724"
                  },
                  "product": {
                    "type": "object",
                    "title": "ProductVariant",
                    "description": "Short information about the Printful Product and Variant",
                    "properties": {
                      "variant_id": {
                        "type": "integer",
                        "description": "Variant ID",
                        "example": 3001
                      },
                      "product_id": {
                        "type": "integer",
                        "description": "Product ID of this variant",
                        "example": 301
                      },
                      "image": {
                        "type": "string",
                        "description": "URL of a sample image for this variant",
                        "example": "https://files.cdn.printful.com/products/71/5309_1581412541.jpg"
                      },
                      "name": {
                        "type": "string",
                        "description": "Display name of this variant",
                        "example": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / 4XL)"
                      }
                    }
                  },
                  "files": {
                    "type": "array",
                    "items": {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "File",
                          "description": "Information about the File",
                          "required": [
                            "url"
                          ],
                          "properties": {
                            "type": {
                              "type": "string",
                              "description": "Role of the file",
                              "example": "default"
                            },
                            "id": {
                              "type": "integer",
                              "description": "File ID",
                              "example": 10,
                              "readOnly": true
                            },
                            "url": {
                              "type": "string",
                              "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                              "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "FileOption",
                                "description": "File option",
                                "required": [
                                  "id",
                                  "value"
                                ],
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option id",
                                    "example": "template_type"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "native"
                                  }
                                }
                              }
                            },
                            "hash": {
                              "type": "string",
                              "description": "MD5 checksum of the file",
                              "example": "ea44330b887dfec278dbc4626a759547",
                              "readOnly": true
                            },
                            "filename": {
                              "type": "string",
                              "description": "File name",
                              "example": "shirt1.png"
                            },
                            "mime_type": {
                              "type": "string",
                              "description": "MIME type of the file",
                              "example": "image/png",
                              "readOnly": true
                            },
                            "size": {
                              "type": "integer",
                              "description": "Size in bytes",
                              "example": 45582633,
                              "readOnly": true
                            },
                            "width": {
                              "type": "integer",
                              "description": "Width in pixels",
                              "example": 1000,
                              "readOnly": true
                            },
                            "height": {
                              "type": "integer",
                              "description": "Height in pixels",
                              "example": 1000,
                              "readOnly": true
                            },
                            "dpi": {
                              "type": "integer",
                              "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                              "example": 300,
                              "readOnly": true
                            },
                            "status": {
                              "type": "string",
                              "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                              "example": "ok",
                              "readOnly": true
                            },
                            "created": {
                              "type": "integer",
                              "description": "File creation timestamp",
                              "example": 1590051937,
                              "readOnly": true
                            },
                            "thumbnail_url": {
                              "type": "string",
                              "description": "Small thumbnail URL",
                              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                              "readOnly": true
                            },
                            "preview_url": {
                              "type": "string",
                              "description": "Medium preview image URL",
                              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                              "readOnly": true
                            },
                            "visible": {
                              "type": "boolean",
                              "description": "Show file in the Printfile Library (default true)",
                              "example": true
                            },
                            "is_temporary": {
                              "type": "boolean",
                              "readOnly": true,
                              "description": "Whether it is a temporary printfile.",
                              "example": false
                            }
                          }
                        },
                        {
                          "type": "object",
                          "properties": {
                            "position": {
                              "writeOnly": true,
                              "allOf": [
                                {
                                  "type": "object",
                                  "title": "GenerationTaskFilePosition",
                                  "description": "Position",
                                  "properties": {
                                    "area_width": {
                                      "type": "integer",
                                      "description": "Positioning area width on print area in pixels",
                                      "example": 1800,
                                      "nullable": true
                                    },
                                    "area_height": {
                                      "type": "integer",
                                      "description": "Positioning area height on print area in pixels",
                                      "example": 2400,
                                      "nullable": true
                                    },
                                    "width": {
                                      "type": "integer",
                                      "description": "Width of the image in given area in pixels",
                                      "example": 1800,
                                      "readOnly": false
                                    },
                                    "height": {
                                      "type": "integer",
                                      "description": "Height of the image in given area in pixels",
                                      "example": 1800,
                                      "readOnly": false
                                    },
                                    "top": {
                                      "type": "integer",
                                      "description": "Image top offset in given area in pixels",
                                      "example": 300
                                    },
                                    "left": {
                                      "type": "integer",
                                      "description": "Image left offset in given area in pixels",
                                      "example": 0
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "properties": {
                                    "limit_to_print_area": {
                                      "type": "boolean",
                                      "default": true,
                                      "description": "Should limit printfile to print area"
                                    }
                                  }
                                }
                              ]
                            }
                          }
                        }
                      ]
                    },
                    "description": "Array of attached printfiles / preview images"
                  },
                  "options": {
                    "type": "array",
                    "description": "Array of additional options for this product [See examples](#tag/Common/Options)",
                    "items": {
                      "type": "object",
                      "title": "ItemOption",
                      "description": "Additional option for order item",
                      "properties": {
                        "id": {
                          "type": "string",
                          "description": "Option ID",
                          "example": "OptionKey"
                        },
                        "value": {
                          "type": "string",
                          "description": "Option value",
                          "example": "OptionValue"
                        }
                      }
                    }
                  },
                  "sku": {
                    "description": "Product identifier (SKU) from the external system",
                    "type": "string",
                    "example": null
                  },
                  "discontinued": {
                    "description": "Whether the item belongs to discontinued product i.e. it's permanently unavailable",
                    "type": "boolean",
                    "example": true
                  },
                  "out_of_stock": {
                    "description": "Whether the item is out of stock i.e. temporarily unavailable",
                    "type": "boolean",
                    "example": true
                  }
                },
                "anyOf": [
                  {
                    "required": [
                      "variant_id"
                    ]
                  },
                  {
                    "required": [
                      "sync_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "external_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "warehouse_product_variant_id"
                    ]
                  },
                  {
                    "required": [
                      "variant_id",
                      "product_template_id"
                    ]
                  },
                  {
                    "required": [
                      "variant_id",
                      "external_product_id"
                    ]
                  }
                ],
                "not": {
                  "anyOf": [
                    {
                      "required": [
                        "product_template_id",
                        "sync_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "external_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "warehouse_product_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "files"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "options"
                      ]
                    },
                    {
                      "required": [
                        "product_template_id",
                        "external_product_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "sync_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "external_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "warehouse_product_variant_id"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "files"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "options"
                      ]
                    },
                    {
                      "required": [
                        "external_product_id",
                        "product_template_id"
                      ]
                    }
                  ]
                }
              }
            },
            "branding_items": {
              "type": "array",
              "description": "Array of branding items in the order",
              "readOnly": true,
              "items": {
                "$ref": "circular reference to #/components/schemas/Item"
              }
            },
            "incomplete_items": {
              "type": "array",
              "description": "Array of incomplete items in the order",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "IncompleteItem",
                "description": "Information about an incomplete item in the order",
                "properties": {
                  "name": {
                    "type": "string",
                    "description": "Incomplete item name",
                    "example": "Red T-Shirt"
                  },
                  "quantity": {
                    "description": "Incompleted item quantity",
                    "type": "integer",
                    "example": 1
                  },
                  "sync_variant_id": {
                    "description": "Sync variant ID of the incompleted item.",
                    "type": "integer",
                    "example": 70
                  },
                  "external_variant_id": {
                    "description": "External variant ID of the incompleted item.",
                    "type": "string",
                    "example": "external-id"
                  },
                  "external_line_item_id": {
                    "description": "External order line item id.",
                    "type": "string",
                    "example": "external-line-item-id"
                  }
                }
              }
            },
            "costs": {
              "allOf": [
                {
                  "type": "object",
                  "title": "OrderCosts",
                  "description": "Order costs (Printful prices)",
                  "properties": {
                    "currency": {
                      "type": "string",
                      "description": "3 letter currency code",
                      "example": "USD"
                    },
                    "subtotal": {
                      "type": "string",
                      "description": "Total cost of all items",
                      "example": "10.00"
                    },
                    "discount": {
                      "type": "string",
                      "description": "Discount sum",
                      "example": "0.00"
                    },
                    "shipping": {
                      "type": "string",
                      "description": "Shipping costs",
                      "example": "5.00"
                    },
                    "digitization": {
                      "type": "string",
                      "description": "Digitization costs",
                      "example": "0.00"
                    },
                    "additional_fee": {
                      "type": "string",
                      "description": "Additional fee for custom product",
                      "example": "0.00"
                    },
                    "fulfillment_fee": {
                      "type": "string",
                      "description": "Custom product fulfillment fee",
                      "example": "0.00"
                    },
                    "retail_delivery_fee": {
                      "type": "string",
                      "description": "Retail delivery fee",
                      "example": "0.00"
                    },
                    "tax": {
                      "type": "string",
                      "description": "Sum of taxes (not included in the item price)",
                      "example": "0.00"
                    },
                    "vat": {
                      "type": "string",
                      "description": "Sum of vat (not included in the item price)",
                      "example": "0.00"
                    },
                    "total": {
                      "type": "string",
                      "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                      "example": "15.00"
                    }
                  }
                },
                {
                  "readOnly": true
                }
              ]
            },
            "retail_costs": {
              "type": "object",
              "title": "OrderRetailCosts",
              "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
              "properties": {
                "currency": {
                  "type": "string",
                  "description": "3 letter currency code",
                  "example": "USD"
                },
                "subtotal": {
                  "type": "string",
                  "nullable": true,
                  "description": "Total cost of all items",
                  "example": "10.00"
                },
                "discount": {
                  "type": "string",
                  "nullable": true,
                  "description": "Discount sum",
                  "example": "0.00"
                },
                "shipping": {
                  "type": "string",
                  "nullable": true,
                  "description": "Shipping costs",
                  "example": "5.00"
                },
                "tax": {
                  "type": "string",
                  "nullable": true,
                  "description": "Sum of taxes (not included in the item price)",
                  "example": "0.00"
                },
                "vat": {
                  "type": "string",
                  "nullable": true,
                  "readOnly": true,
                  "description": "Sum of VAT (not included in the item price)",
                  "example": "0.00"
                },
                "total": {
                  "type": "string",
                  "nullable": true,
                  "readOnly": true,
                  "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                  "example": "15.00"
                }
              }
            },
            "pricing_breakdown": {
              "type": "array",
              "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "PricingBreakdown",
                "description": "Difference between order price and retail costs. Will be shown only if order is completed.",
                "properties": {
                  "customer_pays": {
                    "type": "string",
                    "description": "Amount customer paid",
                    "example": "3.75"
                  },
                  "printful_price": {
                    "type": "string",
                    "description": "Printful price",
                    "example": "6"
                  },
                  "profit": {
                    "type": "string",
                    "description": "Profit",
                    "example": "-2.25"
                  },
                  "currency_symbol": {
                    "type": "string",
                    "description": "Shipment tracking number",
                    "example": "USD"
                  }
                }
              }
            },
            "shipments": {
              "type": "array",
              "description": "Array of shipments already shipped for this order",
              "readOnly": true,
              "items": {
                "type": "object",
                "title": "OrderShipment",
                "description": "Information about order shipment",
                "properties": {
                  "id": {
                    "type": "integer",
                    "description": "Shipment ID",
                    "example": 10
                  },
                  "carrier": {
                    "type": "string",
                    "description": "Carrier name",
                    "example": "FEDEX"
                  },
                  "service": {
                    "type": "string",
                    "description": "Delivery service name",
                    "example": "FedEx SmartPost"
                  },
                  "tracking_number": {
                    "type": "string",
                    "description": "Shipment tracking number",
                    "example": 0
                  },
                  "tracking_url": {
                    "type": "string",
                    "description": "Shipment tracking URL",
                    "example": "https://www.fedex.com/fedextrack/?tracknumbers=0000000000"
                  },
                  "created": {
                    "type": "integer",
                    "description": "Shipping time",
                    "example": 1588716060
                  },
                  "ship_date": {
                    "type": "string",
                    "description": "Ship date",
                    "example": "2020-05-05"
                  },
                  "shipped_at": {
                    "type": "string",
                    "description": "Ship time in unix timestamp",
                    "example": 1588716060
                  },
                  "reshipment": {
                    "type": "boolean",
                    "description": "Whether this is a reshipment",
                    "example": false
                  },
                  "items": {
                    "type": "array",
                    "description": "Array of items in this shipment",
                    "items": {
                      "type": "object",
                      "title": "OrderShipmentItem",
                      "description": "Information about item in an order shipment",
                      "properties": {
                        "item_id": {
                          "type": "integer",
                          "description": "Line item ID",
                          "example": 1
                        },
                        "quantity": {
                          "type": "integer",
                          "description": "Quantity of items in this shipment",
                          "example": 1
                        },
                        "picked": {
                          "type": "integer",
                          "enum": [
                            0,
                            1
                          ],
                          "description": "A boolean indicating that the pickup stage of this item's fulfillment has been completed",
                          "example": 1
                        },
                        "printed": {
                          "type": "integer",
                          "enum": [
                            0,
                            1
                          ],
                          "description": "A boolean indicting that the item has been printed, sublimated or sewed.",
                          "example": 1
                        }
                      }
                    }
                  }
                }
              }
            },
            "gift": {
              "type": "object",
              "title": "OrderGift",
              "description": "Optional gift message for the packing slip",
              "properties": {
                "subject": {
                  "type": "string",
                  "maxLength": 200,
                  "description": "Gift message title",
                  "example": "To John"
                },
                "message": {
                  "type": "string",
                  "maxLength": 200,
                  "description": "Gift message text",
                  "example": "Have a nice day"
                }
              }
            },
            "packing_slip": {
              "type": "object",
              "title": "OrderPackingSlip",
              "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
              "anyOf": [
                {
                  "required": [
                    "email"
                  ]
                },
                {
                  "required": [
                    "phone"
                  ]
                },
                {
                  "required": [
                    "message"
                  ]
                },
                {
                  "required": [
                    "custom_order_id"
                  ]
                }
              ],
              "properties": {
                "email": {
                  "type": "string",
                  "description": "Customer service email",
                  "example": "your-name@your-domain.com"
                },
                "phone": {
                  "type": "string",
                  "description": "Customer service phone",
                  "example": "+371 28888888"
                },
                "message": {
                  "type": "string",
                  "description": "Custom packing slip message",
                  "example": "Message on packing slip"
                },
                "logo_url": {
                  "type": "string",
                  "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                  "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                },
                "store_name": {
                  "type": "string",
                  "description": "Store name override for the return address",
                  "example": "Your store name"
                },
                "custom_order_id": {
                  "type": "string",
                  "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                  "example": "kkk2344lm"
                }
              }
            }
          }
        }
      }
    }
  },
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "code": {
                    "type": "integer",
                    "description": "Response status code `200`",
                    "example": 200
                  }
                }
              },
              {
                "properties": {
                  "result": {
                    "properties": {
                      "costs": {
                        "type": "object",
                        "title": "OrderEstimateCosts",
                        "description": "Order costs (Printful prices)",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "number",
                            "description": "Total cost of all items",
                            "example": 10
                          },
                          "discount": {
                            "type": "number",
                            "description": "Discount sum",
                            "example": 0
                          },
                          "shipping": {
                            "type": "number",
                            "description": "Shipping costs",
                            "example": 5
                          },
                          "digitization": {
                            "type": "string",
                            "description": "Digitization costs",
                            "example": 0
                          },
                          "additional_fee": {
                            "type": "number",
                            "description": "Additional fee for custom product",
                            "example": 0
                          },
                          "fulfillment_fee": {
                            "type": "number",
                            "description": "Custom product fulfillment fee",
                            "example": 0
                          },
                          "tax": {
                            "type": "number",
                            "description": "Sum of taxes (not included in the item price)",
                            "example": 0
                          },
                          "vat": {
                            "type": "number",
                            "description": "Sum of vat (not included in the item price)",
                            "example": 0
                          },
                          "total": {
                            "type": "number",
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": 15
                          }
                        }
                      },
                      "retail_costs": {
                        "type": "object",
                        "title": "OrderEstimateRetailCosts",
                        "description": "Retail costs that are to be displayed on the packing slip for international shipments. Retail costs are used only if every item in order contains the `retail_price` attribute.",
                        "properties": {
                          "currency": {
                            "type": "string",
                            "description": "3 letter currency code",
                            "example": "USD"
                          },
                          "subtotal": {
                            "type": "number",
                            "nullable": true,
                            "description": "Total cost of all items",
                            "example": 10
                          },
                          "discount": {
                            "type": "number",
                            "nullable": true,
                            "description": "Discount sum",
                            "example": 0
                          },
                          "shipping": {
                            "type": "number",
                            "nullable": true,
                            "description": "Shipping costs",
                            "example": 5
                          },
                          "tax": {
                            "type": "number",
                            "nullable": true,
                            "description": "Sum of taxes (not included in the item price)",
                            "example": 0
                          },
                          "vat": {
                            "type": "number",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Sum of VAT (not included in the item price)",
                            "example": 0
                          },
                          "total": {
                            "type": "number",
                            "nullable": true,
                            "readOnly": true,
                            "description": "Grand Total (subtotal-discount+tax+vat+shipping)",
                            "example": 15
                          }
                        }
                      }
                    }
                  }
                }
              }
            ]
          }
        }
      }
    },
    "400": {
      "description": "Bad Request",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `400`",
                "example": 400
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Missing required parameters"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Missing required parameters"
                  }
                }
              }
            }
          }
        }
      }
    },
    "401": {
      "description": "Unauthorized",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `401`",
                "example": 401
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "BadRequest"
                  },
                  "message": {
                    "type": "string",
                    "example": "Malformed Authorization header."
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/orders/estimate-costs' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"recipient\": {\n        \"name\": \"John Doe\",\n        \"address1\": \"19749 Dearborn St\",\n        \"city\": \"Chatsworth\",\n        \"state_code\": \"CA\",\n        \"country_code\": \"US\",\n        \"zip\": \"91311\"\n    },\n    \"items\": [\n        {\n            \"variant_id\": 8630,\n            \"quantity\": 1,\n            \"files\": [\n                {\n                    \"url\": \"https://picsum.photos/200\"\n                }\n            ]\n        }\n    ]\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n$costs = $pf->post('orders/estimate-costs', [\n    'recipient' => [\n        'name' => 'John Doe',\n        'address1' => '172 W Street Ave #105',\n        'city' => 'Burbank',\n        'state_code' => 'CA',\n        'country_code' => 'US',\n        'zip' => '91502',\n    ],\n    'items' => [\n        [\n            'variant_id' => 1, //Small poster\n            'name' => 'Niagara Falls poster', //Display name\n            'retail_price' => '19.99', //Retail price for packing slip\n            'quantity' => 1,\n            'files' => [\n                [\n                    'url' => 'http://example.com/files/posters/poster_1.jpg',\n                ],\n            ],\n        ],\n    ],\n]);\n\nvar_export($costs);\n"
    }
  ]
}
```
</details>

---


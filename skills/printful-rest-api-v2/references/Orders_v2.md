# Orders v2

## Order Building

You can create a draft order without order items but you won’t be able to confirm it until at least one order item are added to that order.
With this approach, you can create a draft order with all necessary recipient data and add items to it as you go (similar to a shopping cart)
by using [Create Order Item](#operation/createItemByOrderId). After every order item addition, the order cost will be re-calculated.

A minimal DRAFT order can be created with just the address:

```
POST https://api.printful.com/v2/orders
Authorization: Bearer {{bearer_token}}
{
  "recipient": {
    "address1": "19749 Dearborn St",
    "city": "Chatsworth",
    "state_code": "CA",
    "state_name": "California",
    "country_code": "US",
    "country_name": "United States",
    "zip": "91311"
  }
}
```

After the order is created the order items can be added one by one.

```
POST https://api.printful.com/v2/orders/{{order_id}}/order-items
Authorization: Bearer {{bearer_token}}
{
    "catalog_variant_id": 4011,
    "source": "catalog",
    "quantity": 1,
    "placements": [
        {
            "placement": "front",
            "technique": "dtg",
            "layers": [
                {
                    "type": "file",
                    "url": "https://www.printful.com/static/images/layout/printful-logo.png"
                }
            ]
        }
    ]
}
```

It is also possible to create an order with all order items in a single request.

```
POST https://api.printful.com/v2/orders
Authorization: Bearer {{bearer_token}}
{
  "recipient": {
    "address1": "19749 Dearborn St",
    "city": "Chatsworth",
    "state_code": "CA",
    "state_name": "California",
    "country_code": "US",
    "country_name": "United States",
    "zip": "91311"
  },
  "order_items": [
    {
      "catalog_variant_id": 4011,
      "source": "catalog",
      "quantity": 1,
      "placements": [
        {
          "placement": "front",
          "technique": "dtg",
          "layers": [
            {
              "type": "file",
              "url": "https://www.printful.com/static/images/layout/printful-logo.png"
            }
          ]
        }
      ]
    }
  ]
}
```

## Order life cycle and statuses

<table>
    <tr>
        <td><strong>draft</strong></td>
        <td>At this stage, the order has been created but has not yet been submitted for fulfillment. It can be freely edited, changed, or deleted at this stage. The order will not charged or fulfilled until it has been confirmed.</td>
    </tr>
    <tr>
        <td><strong>inreview</strong></td>
        <td>The order is being reviewed. It's not possible to cancel the order at this point. It will be possible to cancel the order when the review process is finished.</td>
    </tr>
    <tr>
        <td><strong>pending</strong></td>
    <td>The order has been submitted for fulfillment and it will be charged in this stage, but is not yet accepted for fulfillment. The order can be cancelled at this stage so long as it has not been charged yet.</td>
    </tr>
    <tr>
        <td><strong>failed</strong></td>
        <td>Order was submitted for fulfillment but it had some errors preventing it from being fulfilled (problem with the address, missing printfiles, failed to charge the wallet, etc.).</td>
    </tr>
    <tr>
        <td><strong>canceled</strong></td>
        <td>The order has been canceled and can no longer be processed. If the order was already charged then the cost of the order will be returned to your store.</td>
    </tr>
    <tr>
        <td><strong>inprocess</strong></td>
        <td>The order is being fulfilled and can no longer be canceled or modified. Contact customer support if there are any issues with the order at this point.</td>
    </tr>
    <tr>
        <td><strong>onhold</strong></td>
        <td>The order has encountered a problem during the fulfillment that needs to be resolved together with Printful customer service before fulfillment can continue.</td>
    </tr>
    <tr>
        <td><strong>partial</strong></td>
        <td>The order is partially fulfilled (some items are shipped already, the rest will follow).</td>
    </tr>
    <tr>
        <td><strong>fulfilled</strong></td>
        <td>All items have been shipped successfully.</td>
    </tr>
</table>

You are only charged for orders when they have been confirmed with the “Confirm Order” operation and have the status “pending” [Confirm Order](#operation/confirmOrder).
If the order encounters a problem after it has been confirmed, then it is moved to the failed state so that the problem can be fixed and the order can be resubmitted.

For order status updates please subscribe to one of the webhook events following
webhook events:
- [Order created](#tag/Webhook-v2/operation/orderCreated)
- [Order updated](#tag/Webhook-v2/operation/orderUpdated)
- [Order failed](#tag/Webhook-v2/operation/orderFailed)
- [Order canceled](#tag/Webhook-v2/operation/orderCanceled)

## Order items

Order items are representations of the actual products that will come with an order.

In the sections below we explain each order item source and provide examples of how a simple request might look like for each source.

### Catalog Source

[Catalog products](https://developers.printful.com/docs/v2-beta/#tag/Catalog-v2/What-is-a-catalog-product) are blank products available in Printful. Since they are blank you’ll need to specify design data during the order creation.
This flow is intended if you allow your customers to specify their own designs which means that every order is unique.

The following example request shows how an item structure might look in JSON.

```
POST https://api.printful.com/v2/orders/{{order_id}}/order-items
Authorization: Bearer {{bearer_token}}
{
    "catalog_variant_id": 4012,
    "source": "catalog",
    "quantity": 1,
    "placements": [
        {
            "placement": "front",
            "technique": "dtg",
            "layers": [
                {
                    "type": "file",
                    "url": "https://www.printful.com/static/images/layout/printful-logo.png"
                }
            ]
        }
    ]
}
```
<img src="images/mockups/example_order_item_mockup.jpg" alt="Example of order item mockup" width="300" href="images/mockups/example_order_item_mockup.png"/>

## Design specification
Design data is the information used to customize the product. A design consists
of placements, layers and unique product options. Placements refer to where a 
design will be printed, for example, a T-Shirt might have the placements `front` 
and `left_sleeve`. Placements can also constrict what techniques can be used in 
that area of the product, for example a product might have both `front` and 
`embroidery_front` and embroidery can only be used in the second placement.
A layer is constrained to the print area of the placement that you’ve chosen. 
A design that is outside of the print area won’t be visible.

Depending on the product, placement, and technique different layers might 
be available. 

You can check which layers are available using the [Retrieve a single catalog product](#tag/Catalog-v2/operation/getProductById)
operation.

The design data structure is the same regardless of the flow (making orders,
generating mockups etc.). So the solution can be re-used in the different
endpoints.

[<img src="images/design-data/design-data.png?center" alt="Design data" width="600"/>](images/design-data/design-data.png)

### Placements
The product placement is the part of the product where design layers will be 
printed, different placements will also offer different options and techniques, 
they will also have different limitations. The available placement list will 
vary depending on the product. To get the available placements of a product use 
the [Retrieve a single catalog product](#tag/Catalog-v2/operation/getProductById) 
endpoint with the product ID that you want to check.

<figure style="display: flex; flex-direction: column; align-items: center;">
    <figcaption>Front placement:</figcaption>
    <img src="images/design-data/placements/front.png" width="300" alt="Front design data placement" href="images/design-data/placements/front.png"/>
</figure>
<figure style="display: flex; flex-direction: column; align-items: center;">
    <figcaption>Left sleeve placement:</figcaption>
    <img src="images/design-data/placements/left_sleeve.png" width="300" alt="Left sleeve design data placement" href="images/design-data/placements/left_sleeve.png"/>
</figure>

<div class="alert alert-info">
  <b>Note</b> depending on the type and quantity of the placement additional fees might apply.
</div>

### Layers
Layers are what is printed in each placement, and there can be up to 5 of them 
per one placement. The layers are printed on top of each other with the 0th 
layer being the furthest back and the 4th element being on top of all other layers.

API supports only one layer type - `file`.

[<img src="images/design-data/layers/layer_diagram.png?center" alt="Layer diagram" width="600"/>](images/design-data/layers/layer_diagram.png)

Layers are defined as an array per placement:
```
...
  "layers": [
    {
      "type": "file",
      "url": "https://picsum.photos/200",
      "layer_options": [
        {
          "name": "thread_colors",
          "value": [
            "#FFFFFF"
          ]
        }
      ],
      "position": {
        "width": 640,
        "height": 640,
        "left": 0,
        "top": 0
      }
    }
  ]
...
```

#### Layer Positioning
You can specify the image position inside the print area by providing a position object.

Each print area has specific dimensions, by default our system will assume that your file 
has to fit into those limitations and not exceed them. The (0,0) point is always 
located in the top left corner of the print area.

<div class="alert alert-info">
  <b>Note</b> all values are in inches.
</div>

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
    "left": 0
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
  "left": 675
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
  "left": 1350
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
  "left": 675
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
  "left": 0
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
  "left": 675
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
  "left": 1350
}
```

</td>
</tr>

</table>

### Options
Apart from placement and layers you can also specify options.
```
# Example of option data.
{
  "name": "unlimited_color",
  "value": true
}
```
Option always consist of the name of the option and the value. Value of the 
option can be different depending on the option type but they are always 
predefined for the specified product. You can always check what options and 
what values are available for a specific product by checking
[Retrieve a single catalog product](#tag/Catalog-v2/operation/getProductById)

Options can be specified on three levels:
- Product options - Those options specify additional design options for an 
entire product. E.g. `inside_pocket` allows you to define if a product should be 
fulfilled with an additional inside pocket. Or `stitch_color` lets you choose the 
stitch colors that will be used to stitch back All-Over print products.
- Placement options - They specify options for an entire placement. E.g. `unlimited_color` 
if specified causes the placement to be fulfilled using [Unlimited Color embroidery](https://www.printful.com/unlimited-color-embroidery). 
For the inside label placement, use `inside_label_type` to control how the label is composed (see [Inside label](#inside-label)).
- Layer options - Specify additional parameters for a single layer. E.g. `3d_puff` 
option if set to true causes that particular layer to use [3D puff embroidery](https://www.printful.com/glossary/3d-puff-embroidery).

## Order Customization
### Packing Slip Customization

The packing slip information can be customized according to the store’s specific
requirements. Some of the options are: a custom logo, the name of the store,
customer service email/phone, and more.

The packing slip fields can be configured at the store level (in the Dashboard 
at Settings > Stores > Branding > Packing slip section.) and overridden for a 
specific order.

To override the packing slip settings for the order, you can use `packing_slip` 
fields for each order.

```
POST https://api.printful.com/v2/orders
Authorization: Bearer {{bearer_token}}
{
    "recipient": {
        "address1": "19749 Dearborn St",
        "city": "Chatsworth",
        "state_code": "CA",
        "state_name": "California",
        "country_code": "US",
        "zip": "91311"
    },
    "customization": {
        "packing_slip": {
            "email": "test@example.com",
            "phone": "+48000000000",
            "message": "this is a message",
            "logo_url": "https://example.com/image.jpg",
            "store_name": "a store"
        }
    }
}
```

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

### Gift
The order gift option is part of the order customization and contains two properties:
subject and message. This option allows users to add a custom note that will be seen 
by the final customer who receives the product.

```
POST https://api.printful.com/v2/orders
Authorization: Bearer {{bearer_token}}
{
    "recipient": {
        "address1": "19749 Dearborn St",
        "city": "Chatsworth",
        "state_code": "CA",
        "state_name": "California",
        "country_code": "US",
        "zip": "91311"
    },
    "customization": {
        "gift": {
            "subject": "To John",
            "message": "Happy birthday!"
        }
    }
}
```

## Notes for Beta

The Orders endpoints for the V2 Beta are incomplete. Some V1 endpoints will
still need to be used to achieve certain functionality.

- To cancel an order use `DELETE https://api.printful.com/orders/{order_id}`
  https://developers.printful.com/docs/#operation/cancelOrderById
- Incomplete, or unsynced, items that were created via an integration store but
  haven't been synced with printful yet will not be visible via the V2 orders
  endpoints, if you need this functionality you will need to use V1
  https://developers.printful.com/docs/#tag/Orders-API
- To update an existing order, use `PUT https://api.printful.com/orders/{order_id}`
  https://developers.printful.com/docs/#operation/updateOrderById

## Order Cost Calculations

**Order costs are calculated asynchronously**. This means when you create or 
update an order you are likely to see a response from the API with `costs` and 
`retail_costs` fields set to null and `"calculation_status": "calculating"`.

```
"costs": {
  "calculation_status": "calculating",
  "currency": null,
  "subtotal": null,
  "discount": null,
  "shipping": null,
  "digitization": null,
  "additional_fee": null,
  "fulfillment_fee": null,
  "retail_delivery_fee": null,
  "tax": null,
  "vat": null,
  "total": null
},
"retail_costs": {
  "calculation_status": "calculating",
  "currency": "USD",
  "subtotal": "0.00",
  "discount": "1.00",
  "shipping": "0.00",
  "tax": "0.00",
  "vat": "0.00",
  "total": null
},
```

Once the costs and retail costs are calculated the "calculation_status" will 
change the status to "done"

```
"costs": {
  "calculation_status": "done",
  "currency": "USD",
  "subtotal": "107.50",
  "discount": "0.00",
  "shipping": "14.84",
  "digitization": "0.00",
  "additional_fee": "0.00",
  "fulfillment_fee": "0.00",
  "retail_delivery_fee": "0.00",
  "tax": "11.20",
  "vat": "0.00",
  "total": "133.54"
},
"retail_costs": {
  "calculation_status": "done",
  "currency": "USD",
  "subtotal": "0.00",
  "discount": "1.00",
  "shipping": "0.00",
  "tax": "0.00",
  "vat": "0.00",
  "total": "134.54"
},
```

If you are subscribed to the `order_updated` [webhook event](#tag/Webhook-v2/operation/orderUpdated), 
you will receive an event when the calculation is finished. 

## Inside label

For products that support the inside label placement, you can control how the label is composed by passing the `inside_label_type` option in `placement_options`. This option accepts three values, each affecting how the material and product origin information is combined with your design and how the final inside label looks.

The entire inside label print area is then placed above the material and origin information, so inside the print area it is not necessary to place the design on top.

The **native** inside label is the default option. It works well with square and rectangular images and fits them to a rectangular print area at the top of the label.

Example request with placement options for the inside label:

```
"placements": [
    {
        "placement": "inside_label",
        "technique": "dtg",
        "placement_options": [
            {
                "name": "inside_label_type",
                "value": "native"
            }
        ],
        "layers": [
            {
                "type": "file",
                "url": "https://example.com/your-design.png"
            }
        ]
    }
]
```

### `inside_label_type` values

| Value  | Description                                                                                                                                                                            | Effect on the inside label                                                                                                                                                   |
|--------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `native` | **Default.** Material and product origin information are added automatically by Printful. Your design is placed above that information. Works well with square and rectangular images. | The design is scaled to fit a **rectangular** print area at the top of the label.                                                                                            |
| `custom` | Material and origin information are **not** added automatically. You are responsible for including them in your design file.                                                           | **Not globally available.** If you use `custom` but your design file does not include the required material and origin information, the order will be marked as **invalid**. |

<div class="alert alert-info">
  <b>custom availability</b> The <code>custom</code> inside label type is not available for all accounts. When using it, ensure your design file contains the required material and product origin information; otherwise the order will be invalid.
</div>


## Endpoints

### GET /v2/orders
**Summary:** Retrieve a list of orders

Retrieve a list of orders from a specific store. The order list will be paginated with twenty items per page by default.

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
      "name": "limit",
      "schema": {
        "type": "integer",
        "minimum": 1,
        "maximum": 100,
        "default": 20
      },
      "required": false,
      "description": "The number of results to return per page."
    },
    {
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer",
        "minimum": 0,
        "default": 0
      },
      "required": false,
      "examples": {
        "startAt100": {
          "summary": "Return results after the initial 100",
          "value": 100
        }
      },
      "description": "The number of results to not include in the response starting from the beginning of the list.\n\nThis can be used to return results after the initial 100. For example, sending offset 100\n"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
            "required": [
              "data",
              "paging",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "OrderSummary",
                  "description": "Order summary",
                  "readOnly": true,
                  "required": [
                    "id",
                    "external_id",
                    "store_id",
                    "shipping",
                    "status",
                    "created_at",
                    "updated_at",
                    "recipient",
                    "costs",
                    "retail_costs",
                    "order_items",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "description": "Order ID",
                      "type": "integer",
                      "example": 123
                    },
                    "external_id": {
                      "description": "Order ID from the external system",
                      "type": "string",
                      "example": "4235234213",
                      "nullable": true
                    },
                    "store_id": {
                      "description": "Store ID",
                      "type": "integer",
                      "example": 10
                    },
                    "shipping": {
                      "description": "Shipping method. Defaults to 'STANDARD'",
                      "type": "string",
                      "example": "STANDARD"
                    },
                    "status": {
                      "description": "Order status:<br />\n**draft** - order is not submitted for fulfillment<br />\n**failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br />\n**inreview** - order is being reviewed but is still cancellable at this point<br />\n**pending** - order has been submitted for fulfillment<br />\n**canceled** - order is canceled<br />\n**onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service<br />\n**inprocess** - order is being fulfilled and is no longer cancellable<br />\n**partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br />\n**fulfilled** - all items are shipped\n",
                      "type": "string",
                      "example": "draft"
                    },
                    "created_at": {
                      "description": "Time when the order was created",
                      "type": "string",
                      "format": "date-time",
                      "example": "2023-04-05T06:07:08Z"
                    },
                    "updated_at": {
                      "description": "Time when the order was updated",
                      "type": "string",
                      "format": "date-time",
                      "example": "2023-04-05T06:07:08Z"
                    },
                    "recipient": {
                      "description": "The recipient data.",
                      "title": "Address",
                      "allOf": [
                        {
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
                        }
                      ]
                    },
                    "costs": {
                      "type": "object",
                      "title": "Costs",
                      "description": "The Order costs (Printful prices)",
                      "readOnly": true,
                      "required": [
                        "calculation_status",
                        "currency",
                        "subtotal",
                        "discount",
                        "shipping",
                        "digitization",
                        "additional_fee",
                        "fulfillment_fee",
                        "retail_delivery_fee",
                        "vat",
                        "tax",
                        "total"
                      ],
                      "properties": {
                        "calculation_status": {
                          "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                          "type": "string",
                          "enum": [
                            "done",
                            "calculating",
                            "failed"
                          ],
                          "example": "done"
                        },
                        "currency": {
                          "description": "The code of the currency in which the costs are returned.",
                          "type": "string",
                          "nullable": true,
                          "example": "USD"
                        },
                        "subtotal": {
                          "description": "Total cost of all items.",
                          "type": "string",
                          "nullable": true,
                          "example": "14.95"
                        },
                        "discount": {
                          "description": "Discount sum.",
                          "type": "string",
                          "nullable": true,
                          "example": "1.79"
                        },
                        "shipping": {
                          "description": "Shipping costs.",
                          "type": "string",
                          "nullable": true,
                          "example": "4.79"
                        },
                        "digitization": {
                          "description": "Digitization costs.",
                          "type": "string",
                          "nullable": true,
                          "example": "3.95"
                        },
                        "additional_fee": {
                          "description": "Additional fee for custom product.",
                          "type": "string",
                          "nullable": true,
                          "example": "0.00"
                        },
                        "fulfillment_fee": {
                          "description": "Custom product fulfillment fee.",
                          "type": "string",
                          "nullable": true,
                          "example": "0.00"
                        },
                        "retail_delivery_fee": {
                          "description": "Retail delivery fee.",
                          "type": "string",
                          "nullable": true,
                          "example": "0.00"
                        },
                        "vat": {
                          "description": "Sum of vat (not included in the item price).",
                          "type": "string",
                          "nullable": true,
                          "example": "4.60"
                        },
                        "tax": {
                          "description": "Sum of taxes (not included in the item price).",
                          "type": "string",
                          "nullable": true,
                          "example": "0.00"
                        },
                        "total": {
                          "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                          "type": "string",
                          "nullable": true,
                          "example": "26.50"
                        }
                      }
                    },
                    "retail_costs": {
                      "type": "object",
                      "title": "RetailCosts",
                      "description": "The Order's retail costs",
                      "readOnly": true,
                      "required": [
                        "calculation_status",
                        "currency",
                        "subtotal",
                        "discount",
                        "shipping",
                        "vat",
                        "tax",
                        "total"
                      ],
                      "properties": {
                        "calculation_status": {
                          "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                          "type": "string",
                          "enum": [
                            "done",
                            "calculating",
                            "failed"
                          ],
                          "example": "done"
                        },
                        "currency": {
                          "description": "The code of the currency in which the retail costs are returned.",
                          "type": "string",
                          "example": "EUR"
                        },
                        "subtotal": {
                          "description": "Total cost of all items.",
                          "type": "string",
                          "nullable": true,
                          "example": "26.55"
                        },
                        "discount": {
                          "description": "Discount sum.",
                          "type": "string",
                          "example": "0.00"
                        },
                        "shipping": {
                          "description": "Shipping costs.",
                          "type": "string",
                          "example": "4.79"
                        },
                        "vat": {
                          "description": "Sum of VAT (not included in the item price).",
                          "type": "string",
                          "example": "0.00"
                        },
                        "tax": {
                          "description": "Sum of taxes (not included in the item price).",
                          "type": "string",
                          "example": "0.00"
                        },
                        "total": {
                          "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                          "type": "string",
                          "nullable": true,
                          "example": "31.34"
                        }
                      }
                    },
                    "order_items": {
                      "type": "array",
                      "description": "Simplified order item list. For a full list of all items use the [Get Order Items](#operation/getItemsByOrderId) endpoint.",
                      "items": {
                        "discriminator": {
                          "propertyName": "source",
                          "mapping": {
                            "catalog": "#/components/schemas/CatalogItemSummary",
                            "warehouse": "#/components/schemas/WarehouseItemSummary"
                          }
                        },
                        "oneOf": [
                          {
                            "type": "object",
                            "title": "CatalogItemSummary",
                            "description": "Simplified information about the Catalog Item",
                            "required": [
                              "id",
                              "type",
                              "source",
                              "catalog_variant_id",
                              "external_id",
                              "quantity",
                              "price",
                              "retail_price",
                              "currency",
                              "retail_currency",
                              "_links"
                            ],
                            "properties": {
                              "id": {
                                "description": "Item ID",
                                "type": "integer",
                                "example": 1234,
                                "readOnly": true
                              },
                              "type": {
                                "description": "The item type",
                                "type": "string",
                                "example": "order_item",
                                "enum": [
                                  "order_item",
                                  "branding_item"
                                ]
                              },
                              "source": {
                                "description": "Item source",
                                "type": "string",
                                "example": "catalog",
                                "enum": [
                                  "catalog"
                                ]
                              },
                              "catalog_variant_id": {
                                "description": "Catalog Variant ID associated with the Item",
                                "type": "integer",
                                "example": 4011
                              },
                              "external_id": {
                                "description": "Item user specified external ID",
                                "type": "string",
                                "nullable": true,
                                "example": "123_abc"
                              },
                              "quantity": {
                                "description": "Item quantity",
                                "type": "integer",
                                "example": 1
                              },
                              "name": {
                                "description": "Item custom name",
                                "type": "string",
                                "example": "Custom name"
                              },
                              "price": {
                                "description": "The price Printful charges for the Item",
                                "type": "string",
                                "example": "8.00"
                              },
                              "retail_price": {
                                "description": "Item retail price",
                                "type": "string",
                                "example": "10.00"
                              },
                              "currency": {
                                "description": "The price currency",
                                "type": "string",
                                "example": "EUR"
                              },
                              "retail_currency": {
                                "description": "The retail price currency",
                                "type": "string",
                                "example": "USD"
                              },
                              "_links": {
                                "type": "object",
                                "readOnly": true,
                                "description": "HATEOAS links",
                                "required": [
                                  "self"
                                ],
                                "properties": {
                                  "self": {
                                    "allOf": [
                                      {
                                        "type": "object",
                                        "title": "HateoasLink",
                                        "readOnly": true,
                                        "required": [
                                          "href"
                                        ],
                                        "properties": {
                                          "href": {
                                            "description": "The HREF of the linked resource.",
                                            "type": "string"
                                          }
                                        }
                                      },
                                      {
                                        "description": "Link to a specific order item",
                                        "format": "HateoasLink",
                                        "example": {
                                          "href": "/v2/orders/123/items/123"
                                        }
                                      }
                                    ]
                                  }
                                }
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "WarehouseItemSummary",
                            "description": "Simplified information about the Warehouse Item",
                            "properties": {
                              "id": {
                                "description": "Item ID",
                                "type": "integer",
                                "example": 1234,
                                "readOnly": true
                              },
                              "type": {
                                "description": "The item type",
                                "type": "string",
                                "example": "order_item",
                                "enum": [
                                  "order_item",
                                  "branding_item"
                                ]
                              },
                              "source": {
                                "description": "Item source",
                                "type": "string",
                                "example": "warehouse",
                                "enum": [
                                  "warehouse"
                                ]
                              },
                              "warehouse_product_variant_id": {
                                "description": "ID of warehouse product associated with the Item",
                                "type": "integer",
                                "example": 1123581321
                              },
                              "external_id": {
                                "description": "Item user specified external ID",
                                "type": "string",
                                "nullable": true,
                                "example": "123_abc"
                              },
                              "quantity": {
                                "description": "Item quantity",
                                "type": "integer",
                                "example": 1
                              },
                              "name": {
                                "description": "Item custom name",
                                "type": "string",
                                "example": "Custom name"
                              },
                              "price": {
                                "description": "The price Printful charges for the Item",
                                "type": "string",
                                "example": "8.00"
                              },
                              "retail_price": {
                                "description": "Item retail price",
                                "type": "string",
                                "example": "10.00"
                              },
                              "currency": {
                                "description": "The price currency",
                                "type": "string",
                                "example": "EUR"
                              },
                              "retail_currency": {
                                "description": "The retail price currency",
                                "type": "string",
                                "example": "USD"
                              },
                              "_links": {
                                "type": "object",
                                "readOnly": true,
                                "description": "HATEOAS links",
                                "properties": {
                                  "self": {
                                    "allOf": [
                                      {
                                        "type": "object",
                                        "title": "HateoasLink",
                                        "readOnly": true,
                                        "required": [
                                          "href"
                                        ],
                                        "properties": {
                                          "href": {
                                            "description": "The HREF of the linked resource.",
                                            "type": "string"
                                          }
                                        }
                                      },
                                      {
                                        "description": "Link to a specific order item",
                                        "format": "HateoasLink",
                                        "example": {
                                          "href": "/v2/orders/123/items/123"
                                        }
                                      }
                                    ]
                                  }
                                }
                              }
                            }
                          }
                        ]
                      }
                    },
                    "_links": {
                      "type": "object",
                      "description": "HATEOAS links",
                      "required": [
                        "self",
                        "order_items",
                        "shipments"
                      ],
                      "properties": {
                        "self": {
                          "description": "Link to the order details",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/orders/123"
                          },
                          "allOf": [
                            {
                              "type": "object",
                              "title": "HateoasLink",
                              "readOnly": true,
                              "required": [
                                "href"
                              ],
                              "properties": {
                                "href": {
                                  "description": "The HREF of the linked resource.",
                                  "type": "string"
                                }
                              }
                            }
                          ]
                        },
                        "order_items": {
                          "description": "Link to all order items associated with the order",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/orders/123/order-items"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        },
                        "shipments": {
                          "description": "Link to the shipments associated with the order",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/orders/123/shipments"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        }
                      }
                    }
                  }
                }
              },
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
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders?limit=10&offset=20"
                    },
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
                          }
                        }
                      }
                    ]
                  },
                  "next": {
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders?limit=10&offset=30"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "previous": {
                    "description": "Link to the previous page (absent if the current page is the first one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders?limit=10&limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "first": {
                    "description": "Link to the first page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders?limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "last": {
                    "description": "Link to the last page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders?limit=10&offset=90"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### POST /v2/orders
**Summary:** Create a new order

This endpoint allows the creation of a new order in which the default status will be `draft`.

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
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "required": [
            "recipient",
            "order_items"
          ],
          "properties": {
            "external_id": {
              "description": "Order ID from the external system",
              "type": "string",
              "example": "4235234213"
            },
            "shipping": {
              "description": "Shipping method. Defaults to 'STANDARD'",
              "type": "string",
              "example": "STANDARD"
            },
            "recipient": {
              "description": "The recipient data.",
              "title": "Address",
              "allOf": [
                {
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
                }
              ]
            },
            "order_items": {
              "type": "array",
              "description": "Array of order items",
              "items": {
                "oneOf": [
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "required": [
                          "placements",
                          "catalog_variant_id",
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Catalog source",
                            "type": "string",
                            "enum": [
                              "catalog"
                            ]
                          },
                          "catalog_variant_id": {
                            "description": "ID of catalog variant",
                            "type": "integer",
                            "example": 4011
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "placements": {
                            "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Placement",
                              "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                              "required": [
                                "placement",
                                "technique",
                                "layers",
                                "status",
                                "status_explanation"
                              ],
                              "properties": {
                                "placement": {
                                  "description": "Name of the placement",
                                  "type": "string",
                                  "example": "front"
                                },
                                "technique": {
                                  "description": "Placement's technique",
                                  "type": "string",
                                  "example": "dtg"
                                },
                                "print_area_type": {
                                  "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                  "type": "string",
                                  "writeOnly": true,
                                  "default": "simple",
                                  "enum": [
                                    "simple",
                                    "advanced"
                                  ]
                                },
                                "layers": {
                                  "description": "Information about placement's layers",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "Layer",
                                    "description": "Information about the Layer",
                                    "required": [
                                      "type",
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "description": "Type of layer (e.g. file, text)",
                                        "type": "string",
                                        "example": "file"
                                      },
                                      "url": {
                                        "description": "File image URL if layer type is file",
                                        "type": "string",
                                        "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                      },
                                      "layer_options": {
                                        "description": "List of layer options",
                                        "type": "array",
                                        "items": {
                                          "oneOf": [
                                            {
                                              "type": "object",
                                              "title": "3dPuffOption",
                                              "description": "Should thread use 3d puff technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "3d_puff",
                                                  "enum": [
                                                    "3d_puff"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Whether the 3d puff technique should be used for the layer.",
                                                  "type": "boolean",
                                                  "example": true
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "ThreadColorsOption",
                                              "description": "Specify thread colors for embroidery technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "thread_colors",
                                                  "enum": [
                                                    "thread_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Thread colors for embroidery technique",
                                                  "type": "array",
                                                  "items": {
                                                    "type": "string",
                                                    "example": "#FFFFFF"
                                                  }
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "KnitwearYarnColorsOption",
                                              "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "yarn_colors",
                                                  "enum": [
                                                    "yarn_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Option value",
                                                  "type": "array",
                                                  "items": {
                                                    "description": "Option value",
                                                    "type": "string",
                                                    "example": "#fdfafa",
                                                    "enum": [
                                                      "#090909",
                                                      "#404040",
                                                      "#563c33",
                                                      "#d52213",
                                                      "#6e5242",
                                                      "#7f6a53",
                                                      "#cd5e38",
                                                      "#b57648",
                                                      "#d1773b",
                                                      "#d68785",
                                                      "#c6b5a7",
                                                      "#d6c6b4",
                                                      "#dcd3cc",
                                                      "#edd9d9",
                                                      "#e2dfdc",
                                                      "#fdfafa",
                                                      "#999996",
                                                      "#dda032",
                                                      "#d1c6ae",
                                                      "#eddea4",
                                                      "#48542e",
                                                      "#6e8c4b",
                                                      "#c0c1bd",
                                                      "#243f33",
                                                      "#c5d1d0",
                                                      "#175387",
                                                      "#237d96",
                                                      "#787979",
                                                      "#343d55",
                                                      "#4e59be",
                                                      "#566e99",
                                                      "#504372",
                                                      "#4c1c29",
                                                      "#f66274",
                                                      "#eda6b4",
                                                      "#ddabc8"
                                                    ]
                                                  }
                                                }
                                              }
                                            }
                                          ]
                                        }
                                      },
                                      "position": {
                                        "type": "object",
                                        "title": "LayerPosition",
                                        "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                        "required": [
                                          "top",
                                          "left",
                                          "width",
                                          "height"
                                        ],
                                        "properties": {
                                          "width": {
                                            "description": "Layer width in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "height": {
                                            "description": "Layer height in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "top": {
                                            "description": "Layer top position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          },
                                          "left": {
                                            "description": "Layer left position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          }
                                        }
                                      }
                                    }
                                  }
                                },
                                "placement_options": {
                                  "description": "List of placement options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "UnlimitedColorOption",
                                        "description": "Specify if the design should use unlimited color technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "unlimited_color",
                                            "enum": [
                                              "unlimited_color"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the design should use unlimited color technique.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "InsideLabelTypeOption",
                                        "description": "Specify the type of inside label",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "inside_label_type",
                                            "enum": [
                                              "inside_label_type"
                                            ]
                                          },
                                          "value": {
                                            "description": "Specifies type of inside label design that should be used",
                                            "type": "string",
                                            "enum": [
                                              "native",
                                              "custom"
                                            ]
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "status": {
                                  "type": "string",
                                  "readOnly": true,
                                  "enum": [
                                    "ok",
                                    "failed"
                                  ],
                                  "description": "Status of the placement design"
                                },
                                "status_explanation": {
                                  "type": "string",
                                  "readOnly": true,
                                  "example": "Product with ID: 656 cannot have disjointed design elements.",
                                  "description": "Reason behind failed status"
                                }
                              }
                            }
                          },
                          "orientation": {
                            "description": "Orientation of the design. Applies only to the products that allow multiple orientations such as framed posters.",
                            "type": "string",
                            "readOnly": false,
                            "nullable": true,
                            "enum": [
                              "horizontal",
                              "vertical",
                              "any"
                            ],
                            "example": "horizontal"
                          },
                          "product_options": {
                            "description": "List of product options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "InsidePocketOption",
                                  "description": "Specify if inside pocket should be added to the product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_pocket",
                                      "enum": [
                                        "inside_pocket"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether inside pocket should be added to the product.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "StitchColorOption",
                                  "description": "Specified what color should be used for stitches",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "stitch_color",
                                      "enum": [
                                        "stitch_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "white"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "NotesOption",
                                  "description": "Include additional notes for fulfillment for embroidery prints",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "notes",
                                      "enum": [
                                        "notes"
                                      ]
                                    },
                                    "value": {
                                      "description": "Additional notes for fulfillment for embroidery prints.",
                                      "type": "string",
                                      "example": "Please make sure that top side of the print is within print area"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "LifelikeOption",
                                  "description": "Specifies if generated mockup should use lifelike effect",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "lifelike",
                                      "enum": [
                                        "lifelike"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether generated mockup should use lifelike effect.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "CustomBorderColorOption",
                                  "description": "Used to specify the border color of a sticker",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "custom_border_color",
                                      "enum": [
                                        "custom_border_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Color defined in hexadecimal format",
                                      "type": "string",
                                      "example": "#FF0000"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearBaseColorOption",
                                  "description": "Used to specify the base color on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "base_color",
                                      "enum": [
                                        "base_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearTrimColorOption",
                                  "description": "Used to specify the color of the trim on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "trim_color",
                                      "enum": [
                                        "trim_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearColorReductionMode",
                                  "description": "Used to set the color reduction mode for a knitwear product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "color_reduction_mode",
                                      "enum": [
                                        "color_reduction_mode"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "enum": [
                                        "solid",
                                        "pixelated"
                                      ],
                                      "example": "pixelated"
                                    }
                                  }
                                }
                              ],
                              "discriminator": {
                                "propertyName": "name",
                                "mapping": {
                                  "inside_pocket": "#/components/schemas/InsidePocketOption",
                                  "stitch_color": "#/components/schemas/StitchColorOption",
                                  "notes": "#/components/schemas/NotesOption",
                                  "lifelike": "#/components/schemas/LifelikeOption",
                                  "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                  "base_color": "#/components/schemas/KnitwearBaseColor",
                                  "trim_color": "#/components/schemas/KnitwearTrimColor",
                                  "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                }
                              }
                            }
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  },
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Product Template Item",
                        "description": "Information about the Product Template.",
                        "required": [
                          "source",
                          "product_template_id",
                          "catalog_variant_id"
                        ],
                        "properties": {
                          "source": {
                            "description": "Product Template source.",
                            "type": "string",
                            "enum": [
                              "product_template"
                            ]
                          },
                          "product_template_id": {
                            "oneOf": [
                              {
                                "type": "integer"
                              },
                              {
                                "type": "string"
                              }
                            ],
                            "description": "ID of Product Template. Allows external ID.",
                            "example": "@12345"
                          },
                          "catalog_variant_id": {
                            "description": "ID of Catalog variant. Must correspond to Product Template.",
                            "type": "integer",
                            "example": 123
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  }
                ],
                "discriminator": {
                  "propertyName": "source",
                  "mapping": {
                    "catalog": "#/components/schemas/CatalogItem",
                    "product_template": "#/components/schemas/ProductTemplateItem"
                  }
                }
              }
            },
            "customization": {
              "type": "object",
              "title": "Customization",
              "description": "The Order's customization values",
              "properties": {
                "gift": {
                  "type": "object",
                  "title": "Gift",
                  "description": "The gift subject and message",
                  "properties": {
                    "subject": {
                      "type": "string",
                      "description": "Gift message subject",
                      "example": "To John"
                    },
                    "message": {
                      "type": "string",
                      "description": "Gift message",
                      "example": "Happy birthday!"
                    }
                  }
                },
                "packing_slip": {
                  "type": "object",
                  "title": "PackingSlip",
                  "description": "The values for customized packing slip",
                  "properties": {
                    "email": {
                      "type": "string",
                      "description": "Customer service email",
                      "example": "test@example.com"
                    },
                    "phone": {
                      "type": "string",
                      "description": "Customer service phone",
                      "example": "+48000000000"
                    },
                    "message": {
                      "type": "string",
                      "description": "Custom packing slip message",
                      "example": "This is a message"
                    },
                    "logo_url": {
                      "type": "string",
                      "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                      "example": "https://example.com/image.jpg"
                    },
                    "store_name": {
                      "type": "string",
                      "description": "Store name override for the return address",
                      "example": "A store"
                    },
                    "custom_order_id": {
                      "type": "string",
                      "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                      "example": "11235813"
                    }
                  }
                }
              }
            },
            "retail_costs": {
              "description": "Retail costs",
              "type": "object",
              "properties": {
                "currency": {
                  "description": "The code of the currency in which the retail costs are returned.",
                  "type": "string",
                  "example": "EUR"
                },
                "discount": {
                  "description": "Discount sum.",
                  "type": "string",
                  "example": "123.40"
                },
                "shipping": {
                  "description": "Shipping costs.",
                  "type": "string",
                  "example": "123.40"
                },
                "tax": {
                  "description": "Sum of taxes (not included in the item price).",
                  "type": "string",
                  "example": "123.40"
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "Order",
                "description": "Order",
                "readOnly": true,
                "required": [
                  "id",
                  "external_id",
                  "store_id",
                  "shipping",
                  "status",
                  "created_at",
                  "updated_at",
                  "recipient",
                  "costs",
                  "retail_costs",
                  "order_items",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "description": "Order ID",
                    "type": "integer",
                    "example": 123
                  },
                  "external_id": {
                    "description": "Order ID from the external system",
                    "type": "string",
                    "example": "4235234213",
                    "nullable": true
                  },
                  "store_id": {
                    "description": "Store ID",
                    "type": "integer",
                    "example": 10
                  },
                  "shipping": {
                    "description": "Shipping method. Defaults to 'STANDARD'",
                    "type": "string",
                    "example": "STANDARD"
                  },
                  "status": {
                    "description": "Order status:<br />\n**draft** - order is not submitted for fulfillment<br />\n**failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br />\n**inreview** - order is being reviewed<br />\n**pending** - order has been submitted for fulfillment<br />\n**canceled** - order is canceled<br />\n**onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service<br />\n**inprocess** - order is being fulfilled and is no longer cancellable<br />\n**partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br />\n**fulfilled** - all items are shipped\n",
                    "type": "string",
                    "example": "draft"
                  },
                  "created_at": {
                    "description": "Time when the order was created",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "updated_at": {
                    "description": "Time when the order was updated",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "recipient": {
                    "description": "The recipient data.",
                    "title": "Address",
                    "allOf": [
                      {
                        "type": "object",
                        "title": "AddressReadonly",
                        "description": "Information about the address",
                        "required": [
                          "name",
                          "company",
                          "address1",
                          "address2",
                          "city",
                          "state_code",
                          "state_name",
                          "country_code",
                          "country_name",
                          "zip",
                          "phone",
                          "email",
                          "tax_number"
                        ],
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
                      }
                    ]
                  },
                  "costs": {
                    "type": "object",
                    "title": "Costs",
                    "description": "The Order costs (Printful prices)",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "digitization",
                      "additional_fee",
                      "fulfillment_fee",
                      "retail_delivery_fee",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the costs are returned.",
                        "type": "string",
                        "nullable": true,
                        "example": "USD"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "14.95"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "nullable": true,
                        "example": "1.79"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "4.79"
                      },
                      "digitization": {
                        "description": "Digitization costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "3.95"
                      },
                      "additional_fee": {
                        "description": "Additional fee for custom product.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "fulfillment_fee": {
                        "description": "Custom product fulfillment fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "retail_delivery_fee": {
                        "description": "Retail delivery fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "vat": {
                        "description": "Sum of vat (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "4.60"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "26.50"
                      }
                    }
                  },
                  "retail_costs": {
                    "type": "object",
                    "title": "RetailCosts",
                    "description": "The Order's retail costs",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the retail costs are returned.",
                        "type": "string",
                        "example": "EUR"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "26.55"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "example": "0.00"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "example": "4.79"
                      },
                      "vat": {
                        "description": "Sum of VAT (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "31.34"
                      }
                    }
                  },
                  "order_items": {
                    "type": "array",
                    "description": "Simplified order item list. For a full list of all items use the [Get Order Items](#operation/getItemsByOrderId) endpoint.",
                    "items": {
                      "discriminator": {
                        "propertyName": "source",
                        "mapping": {
                          "catalog": "#/components/schemas/CatalogItemSummary",
                          "warehouse": "#/components/schemas/WarehouseItemSummary"
                        }
                      },
                      "oneOf": [
                        {
                          "type": "object",
                          "title": "CatalogItemSummary",
                          "description": "Simplified information about the Catalog Item",
                          "required": [
                            "id",
                            "type",
                            "source",
                            "catalog_variant_id",
                            "external_id",
                            "quantity",
                            "price",
                            "retail_price",
                            "currency",
                            "retail_currency",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "catalog",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "Catalog Variant ID associated with the Item",
                              "type": "integer",
                              "example": 4011
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "WarehouseItemSummary",
                          "description": "Simplified information about the Warehouse Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "warehouse",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse product associated with the Item",
                              "type": "integer",
                              "example": 1123581321
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  },
                  "customization": {
                    "type": "object",
                    "title": "CustomizationReadonly",
                    "description": "The Order's customization values",
                    "required": [
                      "gift",
                      "packing_slip"
                    ],
                    "properties": {
                      "gift": {
                        "type": "object",
                        "title": "Gift",
                        "description": "The gift subject and message",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "description": "Gift message subject",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "description": "Gift message",
                            "example": "Happy birthday!"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "PackingSlipReadonly",
                        "description": "The values for customized packing slip",
                        "required": [
                          "email",
                          "phone",
                          "message",
                          "logo_url",
                          "store_name",
                          "custom_order_id"
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "test@example.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+48000000000"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "This is a message"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "https://example.com/image.jpg"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "A store"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "11235813"
                          }
                        }
                      }
                    }
                  },
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "required": [
                      "self",
                      "order_confirmation",
                      "order_invoices",
                      "order_items",
                      "shipments"
                    ],
                    "properties": {
                      "self": {
                        "description": "Link to the order details",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        },
                        "allOf": [
                          {
                            "type": "object",
                            "title": "HateoasLink",
                            "readOnly": true,
                            "required": [
                              "href"
                            ],
                            "properties": {
                              "href": {
                                "description": "The HREF of the linked resource.",
                                "type": "string"
                              }
                            }
                          }
                        ]
                      },
                      "order_confirmation": {
                        "description": "Link to the order confirmation",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/confirmation"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_invoices": {
                        "description": "Link to the order invoice",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/invoices"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_items": {
                        "description": "Link to all order items associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-items"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "shipments": {
                        "description": "Link to the shipments associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/shipments"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      }
                    }
                  }
                }
              }
            }
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### GET /v2/orders/{order_id}
**Summary:** Retrieve a single order

Retrieve a single order from the specified store. The result object will contain links to the same resource, order items, and shipments.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getOrder",
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
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "Order",
                "description": "Order",
                "readOnly": true,
                "required": [
                  "id",
                  "external_id",
                  "store_id",
                  "shipping",
                  "status",
                  "created_at",
                  "updated_at",
                  "recipient",
                  "costs",
                  "retail_costs",
                  "order_items",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "description": "Order ID",
                    "type": "integer",
                    "example": 123
                  },
                  "external_id": {
                    "description": "Order ID from the external system",
                    "type": "string",
                    "example": "4235234213",
                    "nullable": true
                  },
                  "store_id": {
                    "description": "Store ID",
                    "type": "integer",
                    "example": 10
                  },
                  "shipping": {
                    "description": "Shipping method. Defaults to 'STANDARD'",
                    "type": "string",
                    "example": "STANDARD"
                  },
                  "status": {
                    "description": "Order status:<br />\n**draft** - order is not submitted for fulfillment<br />\n**failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br />\n**inreview** - order is being reviewed<br />\n**pending** - order has been submitted for fulfillment<br />\n**canceled** - order is canceled<br />\n**onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service<br />\n**inprocess** - order is being fulfilled and is no longer cancellable<br />\n**partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br />\n**fulfilled** - all items are shipped\n",
                    "type": "string",
                    "example": "draft"
                  },
                  "created_at": {
                    "description": "Time when the order was created",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "updated_at": {
                    "description": "Time when the order was updated",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "recipient": {
                    "description": "The recipient data.",
                    "title": "Address",
                    "allOf": [
                      {
                        "type": "object",
                        "title": "AddressReadonly",
                        "description": "Information about the address",
                        "required": [
                          "name",
                          "company",
                          "address1",
                          "address2",
                          "city",
                          "state_code",
                          "state_name",
                          "country_code",
                          "country_name",
                          "zip",
                          "phone",
                          "email",
                          "tax_number"
                        ],
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
                      }
                    ]
                  },
                  "costs": {
                    "type": "object",
                    "title": "Costs",
                    "description": "The Order costs (Printful prices)",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "digitization",
                      "additional_fee",
                      "fulfillment_fee",
                      "retail_delivery_fee",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the costs are returned.",
                        "type": "string",
                        "nullable": true,
                        "example": "USD"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "14.95"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "nullable": true,
                        "example": "1.79"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "4.79"
                      },
                      "digitization": {
                        "description": "Digitization costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "3.95"
                      },
                      "additional_fee": {
                        "description": "Additional fee for custom product.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "fulfillment_fee": {
                        "description": "Custom product fulfillment fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "retail_delivery_fee": {
                        "description": "Retail delivery fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "vat": {
                        "description": "Sum of vat (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "4.60"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "26.50"
                      }
                    }
                  },
                  "retail_costs": {
                    "type": "object",
                    "title": "RetailCosts",
                    "description": "The Order's retail costs",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the retail costs are returned.",
                        "type": "string",
                        "example": "EUR"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "26.55"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "example": "0.00"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "example": "4.79"
                      },
                      "vat": {
                        "description": "Sum of VAT (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "31.34"
                      }
                    }
                  },
                  "order_items": {
                    "type": "array",
                    "description": "Simplified order item list. For a full list of all items use the [Get Order Items](#operation/getItemsByOrderId) endpoint.",
                    "items": {
                      "discriminator": {
                        "propertyName": "source",
                        "mapping": {
                          "catalog": "#/components/schemas/CatalogItemSummary",
                          "warehouse": "#/components/schemas/WarehouseItemSummary"
                        }
                      },
                      "oneOf": [
                        {
                          "type": "object",
                          "title": "CatalogItemSummary",
                          "description": "Simplified information about the Catalog Item",
                          "required": [
                            "id",
                            "type",
                            "source",
                            "catalog_variant_id",
                            "external_id",
                            "quantity",
                            "price",
                            "retail_price",
                            "currency",
                            "retail_currency",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "catalog",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "Catalog Variant ID associated with the Item",
                              "type": "integer",
                              "example": 4011
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "WarehouseItemSummary",
                          "description": "Simplified information about the Warehouse Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "warehouse",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse product associated with the Item",
                              "type": "integer",
                              "example": 1123581321
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  },
                  "customization": {
                    "type": "object",
                    "title": "CustomizationReadonly",
                    "description": "The Order's customization values",
                    "required": [
                      "gift",
                      "packing_slip"
                    ],
                    "properties": {
                      "gift": {
                        "type": "object",
                        "title": "Gift",
                        "description": "The gift subject and message",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "description": "Gift message subject",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "description": "Gift message",
                            "example": "Happy birthday!"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "PackingSlipReadonly",
                        "description": "The values for customized packing slip",
                        "required": [
                          "email",
                          "phone",
                          "message",
                          "logo_url",
                          "store_name",
                          "custom_order_id"
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "test@example.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+48000000000"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "This is a message"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "https://example.com/image.jpg"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "A store"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "11235813"
                          }
                        }
                      }
                    }
                  },
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "required": [
                      "self",
                      "order_confirmation",
                      "order_invoices",
                      "order_items",
                      "shipments"
                    ],
                    "properties": {
                      "self": {
                        "description": "Link to the order details",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        },
                        "allOf": [
                          {
                            "type": "object",
                            "title": "HateoasLink",
                            "readOnly": true,
                            "required": [
                              "href"
                            ],
                            "properties": {
                              "href": {
                                "description": "The HREF of the linked resource.",
                                "type": "string"
                              }
                            }
                          }
                        ]
                      },
                      "order_confirmation": {
                        "description": "Link to the order confirmation",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/confirmation"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_invoices": {
                        "description": "Link to the order invoice",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/invoices"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_items": {
                        "description": "Link to all order items associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-items"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "shipments": {
                        "description": "Link to the shipments associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/shipments"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      }
                    }
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
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### DELETE /v2/orders/{order_id}
**Summary:** Delete an order

<div class="alert alert-danger">
  <strong>Warning:</strong> The DELETE HTTP method in version 2 of order in the 
  API will delete the order if the order can be deleted. This is distinct from 
  version 1 where the DELETE method will actually cancel the order rather than 
  delete it. Please, keep this in mind if you are migrating to version 2 from 
  version 1
</div>

Delete the order if it can be deleted. An order can be deleted if it's status is
`draft`, `failed` or `cancelled`. The order must also have not been charged yet
and there must be no payments pending.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteOrder",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    }
  ],
  "responses": {
    "204": {
      "description": "No Content"
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
    },
    "409": {
      "description": "Conflict",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `409`",
                "example": 409
              },
              "result": {
                "type": "string",
                "description": "An error message describing a conflict between the request and the current state of the resource.",
                "example": "Attempting to update a resource that is already being updated. Please try again after the previous update has completed"
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "integer",
                    "example": 409
                  },
                  "message": {
                    "type": "string",
                    "example": "Attempting to update a resource that is already being updated. Please try again after the previous update has completed"
                  }
                }
              }
            }
          }
        }
      }
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### PATCH /v2/orders/{order_id}
**Summary:** Update an order

Make a partial update of an order.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "updateOrder",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    }
  ],
  "requestBody": {
    "description": "PATCH request body",
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "properties": {
            "external_id": {
              "description": "Order ID from the external system",
              "type": "string",
              "example": "4235234213"
            },
            "shipping": {
              "description": "Shipping method. Defaults to 'STANDARD'",
              "type": "string",
              "example": "STANDARD"
            },
            "recipient": {
              "description": "The recipient data.",
              "title": "Address",
              "allOf": [
                {
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
                }
              ]
            },
            "order_items": {
              "type": "array",
              "description": "Array of order items",
              "items": {
                "oneOf": [
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "required": [
                          "placements",
                          "catalog_variant_id",
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Catalog source",
                            "type": "string",
                            "enum": [
                              "catalog"
                            ]
                          },
                          "catalog_variant_id": {
                            "description": "ID of catalog variant",
                            "type": "integer",
                            "example": 4011
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "placements": {
                            "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Placement",
                              "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                              "required": [
                                "placement",
                                "technique",
                                "layers",
                                "status",
                                "status_explanation"
                              ],
                              "properties": {
                                "placement": {
                                  "description": "Name of the placement",
                                  "type": "string",
                                  "example": "front"
                                },
                                "technique": {
                                  "description": "Placement's technique",
                                  "type": "string",
                                  "example": "dtg"
                                },
                                "print_area_type": {
                                  "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                  "type": "string",
                                  "writeOnly": true,
                                  "default": "simple",
                                  "enum": [
                                    "simple",
                                    "advanced"
                                  ]
                                },
                                "layers": {
                                  "description": "Information about placement's layers",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "Layer",
                                    "description": "Information about the Layer",
                                    "required": [
                                      "type",
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "description": "Type of layer (e.g. file, text)",
                                        "type": "string",
                                        "example": "file"
                                      },
                                      "url": {
                                        "description": "File image URL if layer type is file",
                                        "type": "string",
                                        "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                      },
                                      "layer_options": {
                                        "description": "List of layer options",
                                        "type": "array",
                                        "items": {
                                          "oneOf": [
                                            {
                                              "type": "object",
                                              "title": "3dPuffOption",
                                              "description": "Should thread use 3d puff technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "3d_puff",
                                                  "enum": [
                                                    "3d_puff"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Whether the 3d puff technique should be used for the layer.",
                                                  "type": "boolean",
                                                  "example": true
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "ThreadColorsOption",
                                              "description": "Specify thread colors for embroidery technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "thread_colors",
                                                  "enum": [
                                                    "thread_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Thread colors for embroidery technique",
                                                  "type": "array",
                                                  "items": {
                                                    "type": "string",
                                                    "example": "#FFFFFF"
                                                  }
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "KnitwearYarnColorsOption",
                                              "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "yarn_colors",
                                                  "enum": [
                                                    "yarn_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Option value",
                                                  "type": "array",
                                                  "items": {
                                                    "description": "Option value",
                                                    "type": "string",
                                                    "example": "#fdfafa",
                                                    "enum": [
                                                      "#090909",
                                                      "#404040",
                                                      "#563c33",
                                                      "#d52213",
                                                      "#6e5242",
                                                      "#7f6a53",
                                                      "#cd5e38",
                                                      "#b57648",
                                                      "#d1773b",
                                                      "#d68785",
                                                      "#c6b5a7",
                                                      "#d6c6b4",
                                                      "#dcd3cc",
                                                      "#edd9d9",
                                                      "#e2dfdc",
                                                      "#fdfafa",
                                                      "#999996",
                                                      "#dda032",
                                                      "#d1c6ae",
                                                      "#eddea4",
                                                      "#48542e",
                                                      "#6e8c4b",
                                                      "#c0c1bd",
                                                      "#243f33",
                                                      "#c5d1d0",
                                                      "#175387",
                                                      "#237d96",
                                                      "#787979",
                                                      "#343d55",
                                                      "#4e59be",
                                                      "#566e99",
                                                      "#504372",
                                                      "#4c1c29",
                                                      "#f66274",
                                                      "#eda6b4",
                                                      "#ddabc8"
                                                    ]
                                                  }
                                                }
                                              }
                                            }
                                          ]
                                        }
                                      },
                                      "position": {
                                        "type": "object",
                                        "title": "LayerPosition",
                                        "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                        "required": [
                                          "top",
                                          "left",
                                          "width",
                                          "height"
                                        ],
                                        "properties": {
                                          "width": {
                                            "description": "Layer width in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "height": {
                                            "description": "Layer height in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "top": {
                                            "description": "Layer top position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          },
                                          "left": {
                                            "description": "Layer left position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          }
                                        }
                                      }
                                    }
                                  }
                                },
                                "placement_options": {
                                  "description": "List of placement options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "UnlimitedColorOption",
                                        "description": "Specify if the design should use unlimited color technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "unlimited_color",
                                            "enum": [
                                              "unlimited_color"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the design should use unlimited color technique.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "InsideLabelTypeOption",
                                        "description": "Specify the type of inside label",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "inside_label_type",
                                            "enum": [
                                              "inside_label_type"
                                            ]
                                          },
                                          "value": {
                                            "description": "Specifies type of inside label design that should be used",
                                            "type": "string",
                                            "enum": [
                                              "native",
                                              "custom"
                                            ]
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "status": {
                                  "type": "string",
                                  "readOnly": true,
                                  "enum": [
                                    "ok",
                                    "failed"
                                  ],
                                  "description": "Status of the placement design"
                                },
                                "status_explanation": {
                                  "type": "string",
                                  "readOnly": true,
                                  "example": "Product with ID: 656 cannot have disjointed design elements.",
                                  "description": "Reason behind failed status"
                                }
                              }
                            }
                          },
                          "orientation": {
                            "description": "Orientation of the design. Applies only to the products that allow multiple orientations such as framed posters.",
                            "type": "string",
                            "readOnly": false,
                            "nullable": true,
                            "enum": [
                              "horizontal",
                              "vertical",
                              "any"
                            ],
                            "example": "horizontal"
                          },
                          "product_options": {
                            "description": "List of product options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "InsidePocketOption",
                                  "description": "Specify if inside pocket should be added to the product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_pocket",
                                      "enum": [
                                        "inside_pocket"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether inside pocket should be added to the product.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "StitchColorOption",
                                  "description": "Specified what color should be used for stitches",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "stitch_color",
                                      "enum": [
                                        "stitch_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "white"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "NotesOption",
                                  "description": "Include additional notes for fulfillment for embroidery prints",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "notes",
                                      "enum": [
                                        "notes"
                                      ]
                                    },
                                    "value": {
                                      "description": "Additional notes for fulfillment for embroidery prints.",
                                      "type": "string",
                                      "example": "Please make sure that top side of the print is within print area"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "LifelikeOption",
                                  "description": "Specifies if generated mockup should use lifelike effect",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "lifelike",
                                      "enum": [
                                        "lifelike"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether generated mockup should use lifelike effect.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "CustomBorderColorOption",
                                  "description": "Used to specify the border color of a sticker",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "custom_border_color",
                                      "enum": [
                                        "custom_border_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Color defined in hexadecimal format",
                                      "type": "string",
                                      "example": "#FF0000"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearBaseColorOption",
                                  "description": "Used to specify the base color on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "base_color",
                                      "enum": [
                                        "base_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearTrimColorOption",
                                  "description": "Used to specify the color of the trim on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "trim_color",
                                      "enum": [
                                        "trim_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearColorReductionMode",
                                  "description": "Used to set the color reduction mode for a knitwear product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "color_reduction_mode",
                                      "enum": [
                                        "color_reduction_mode"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "enum": [
                                        "solid",
                                        "pixelated"
                                      ],
                                      "example": "pixelated"
                                    }
                                  }
                                }
                              ],
                              "discriminator": {
                                "propertyName": "name",
                                "mapping": {
                                  "inside_pocket": "#/components/schemas/InsidePocketOption",
                                  "stitch_color": "#/components/schemas/StitchColorOption",
                                  "notes": "#/components/schemas/NotesOption",
                                  "lifelike": "#/components/schemas/LifelikeOption",
                                  "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                  "base_color": "#/components/schemas/KnitwearBaseColor",
                                  "trim_color": "#/components/schemas/KnitwearTrimColor",
                                  "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                }
                              }
                            }
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  },
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Product Template Item",
                        "description": "Information about the Product Template.",
                        "required": [
                          "source",
                          "product_template_id",
                          "catalog_variant_id"
                        ],
                        "properties": {
                          "source": {
                            "description": "Product Template source.",
                            "type": "string",
                            "enum": [
                              "product_template"
                            ]
                          },
                          "product_template_id": {
                            "oneOf": [
                              {
                                "type": "integer"
                              },
                              {
                                "type": "string"
                              }
                            ],
                            "description": "ID of Product Template. Allows external ID.",
                            "example": "@12345"
                          },
                          "catalog_variant_id": {
                            "description": "ID of Catalog variant. Must correspond to Product Template.",
                            "type": "integer",
                            "example": 123
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  }
                ],
                "discriminator": {
                  "propertyName": "source",
                  "mapping": {
                    "catalog": "#/components/schemas/CatalogItem",
                    "product_template": "#/components/schemas/ProductTemplateItem"
                  }
                }
              }
            },
            "customization": {
              "type": "object",
              "title": "Customization",
              "description": "The Order's customization values",
              "properties": {
                "gift": {
                  "type": "object",
                  "title": "Gift",
                  "description": "The gift subject and message",
                  "properties": {
                    "subject": {
                      "type": "string",
                      "description": "Gift message subject",
                      "example": "To John"
                    },
                    "message": {
                      "type": "string",
                      "description": "Gift message",
                      "example": "Happy birthday!"
                    }
                  }
                },
                "packing_slip": {
                  "type": "object",
                  "title": "PackingSlip",
                  "description": "The values for customized packing slip",
                  "properties": {
                    "email": {
                      "type": "string",
                      "description": "Customer service email",
                      "example": "test@example.com"
                    },
                    "phone": {
                      "type": "string",
                      "description": "Customer service phone",
                      "example": "+48000000000"
                    },
                    "message": {
                      "type": "string",
                      "description": "Custom packing slip message",
                      "example": "This is a message"
                    },
                    "logo_url": {
                      "type": "string",
                      "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                      "example": "https://example.com/image.jpg"
                    },
                    "store_name": {
                      "type": "string",
                      "description": "Store name override for the return address",
                      "example": "A store"
                    },
                    "custom_order_id": {
                      "type": "string",
                      "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                      "example": "11235813"
                    }
                  }
                }
              }
            },
            "retail_costs": {
              "description": "Retail costs",
              "type": "object",
              "properties": {
                "currency": {
                  "description": "The code of the currency in which the retail costs are returned.",
                  "type": "string",
                  "example": "EUR"
                },
                "discount": {
                  "description": "Discount sum.",
                  "type": "string",
                  "example": "123.40"
                },
                "shipping": {
                  "description": "Shipping costs.",
                  "type": "string",
                  "example": "123.40"
                },
                "tax": {
                  "description": "Sum of taxes (not included in the item price).",
                  "type": "string",
                  "example": "123.40"
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "Order",
                "description": "Order",
                "readOnly": true,
                "required": [
                  "id",
                  "external_id",
                  "store_id",
                  "shipping",
                  "status",
                  "created_at",
                  "updated_at",
                  "recipient",
                  "costs",
                  "retail_costs",
                  "order_items",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "description": "Order ID",
                    "type": "integer",
                    "example": 123
                  },
                  "external_id": {
                    "description": "Order ID from the external system",
                    "type": "string",
                    "example": "4235234213",
                    "nullable": true
                  },
                  "store_id": {
                    "description": "Store ID",
                    "type": "integer",
                    "example": 10
                  },
                  "shipping": {
                    "description": "Shipping method. Defaults to 'STANDARD'",
                    "type": "string",
                    "example": "STANDARD"
                  },
                  "status": {
                    "description": "Order status:<br />\n**draft** - order is not submitted for fulfillment<br />\n**failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br />\n**inreview** - order is being reviewed<br />\n**pending** - order has been submitted for fulfillment<br />\n**canceled** - order is canceled<br />\n**onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service<br />\n**inprocess** - order is being fulfilled and is no longer cancellable<br />\n**partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br />\n**fulfilled** - all items are shipped\n",
                    "type": "string",
                    "example": "draft"
                  },
                  "created_at": {
                    "description": "Time when the order was created",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "updated_at": {
                    "description": "Time when the order was updated",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "recipient": {
                    "description": "The recipient data.",
                    "title": "Address",
                    "allOf": [
                      {
                        "type": "object",
                        "title": "AddressReadonly",
                        "description": "Information about the address",
                        "required": [
                          "name",
                          "company",
                          "address1",
                          "address2",
                          "city",
                          "state_code",
                          "state_name",
                          "country_code",
                          "country_name",
                          "zip",
                          "phone",
                          "email",
                          "tax_number"
                        ],
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
                      }
                    ]
                  },
                  "costs": {
                    "type": "object",
                    "title": "Costs",
                    "description": "The Order costs (Printful prices)",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "digitization",
                      "additional_fee",
                      "fulfillment_fee",
                      "retail_delivery_fee",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the costs are returned.",
                        "type": "string",
                        "nullable": true,
                        "example": "USD"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "14.95"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "nullable": true,
                        "example": "1.79"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "4.79"
                      },
                      "digitization": {
                        "description": "Digitization costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "3.95"
                      },
                      "additional_fee": {
                        "description": "Additional fee for custom product.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "fulfillment_fee": {
                        "description": "Custom product fulfillment fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "retail_delivery_fee": {
                        "description": "Retail delivery fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "vat": {
                        "description": "Sum of vat (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "4.60"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "26.50"
                      }
                    }
                  },
                  "retail_costs": {
                    "type": "object",
                    "title": "RetailCosts",
                    "description": "The Order's retail costs",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the retail costs are returned.",
                        "type": "string",
                        "example": "EUR"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "26.55"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "example": "0.00"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "example": "4.79"
                      },
                      "vat": {
                        "description": "Sum of VAT (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "31.34"
                      }
                    }
                  },
                  "order_items": {
                    "type": "array",
                    "description": "Simplified order item list. For a full list of all items use the [Get Order Items](#operation/getItemsByOrderId) endpoint.",
                    "items": {
                      "discriminator": {
                        "propertyName": "source",
                        "mapping": {
                          "catalog": "#/components/schemas/CatalogItemSummary",
                          "warehouse": "#/components/schemas/WarehouseItemSummary"
                        }
                      },
                      "oneOf": [
                        {
                          "type": "object",
                          "title": "CatalogItemSummary",
                          "description": "Simplified information about the Catalog Item",
                          "required": [
                            "id",
                            "type",
                            "source",
                            "catalog_variant_id",
                            "external_id",
                            "quantity",
                            "price",
                            "retail_price",
                            "currency",
                            "retail_currency",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "catalog",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "Catalog Variant ID associated with the Item",
                              "type": "integer",
                              "example": 4011
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "WarehouseItemSummary",
                          "description": "Simplified information about the Warehouse Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "warehouse",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse product associated with the Item",
                              "type": "integer",
                              "example": 1123581321
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  },
                  "customization": {
                    "type": "object",
                    "title": "CustomizationReadonly",
                    "description": "The Order's customization values",
                    "required": [
                      "gift",
                      "packing_slip"
                    ],
                    "properties": {
                      "gift": {
                        "type": "object",
                        "title": "Gift",
                        "description": "The gift subject and message",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "description": "Gift message subject",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "description": "Gift message",
                            "example": "Happy birthday!"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "PackingSlipReadonly",
                        "description": "The values for customized packing slip",
                        "required": [
                          "email",
                          "phone",
                          "message",
                          "logo_url",
                          "store_name",
                          "custom_order_id"
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "test@example.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+48000000000"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "This is a message"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "https://example.com/image.jpg"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "A store"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "11235813"
                          }
                        }
                      }
                    }
                  },
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "required": [
                      "self",
                      "order_confirmation",
                      "order_invoices",
                      "order_items",
                      "shipments"
                    ],
                    "properties": {
                      "self": {
                        "description": "Link to the order details",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        },
                        "allOf": [
                          {
                            "type": "object",
                            "title": "HateoasLink",
                            "readOnly": true,
                            "required": [
                              "href"
                            ],
                            "properties": {
                              "href": {
                                "description": "The HREF of the linked resource.",
                                "type": "string"
                              }
                            }
                          }
                        ]
                      },
                      "order_confirmation": {
                        "description": "Link to the order confirmation",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/confirmation"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_invoices": {
                        "description": "Link to the order invoice",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/invoices"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_items": {
                        "description": "Link to all order items associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-items"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "shipments": {
                        "description": "Link to the shipments associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/shipments"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      }
                    }
                  }
                }
              }
            }
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### POST /v2/orders/{order_id}/confirmation
**Summary:** Confirm an order

This endpoint allows customers to confirm the order and start the fulfillment in the production facility.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "confirmOrder",
  "security": [
    {
      "OAuth": [
        "orders"
      ]
    }
  ],
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    }
  ],
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "Order",
                "description": "Order",
                "readOnly": true,
                "required": [
                  "id",
                  "external_id",
                  "store_id",
                  "shipping",
                  "status",
                  "created_at",
                  "updated_at",
                  "recipient",
                  "costs",
                  "retail_costs",
                  "order_items",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "description": "Order ID",
                    "type": "integer",
                    "example": 123
                  },
                  "external_id": {
                    "description": "Order ID from the external system",
                    "type": "string",
                    "example": "4235234213",
                    "nullable": true
                  },
                  "store_id": {
                    "description": "Store ID",
                    "type": "integer",
                    "example": 10
                  },
                  "shipping": {
                    "description": "Shipping method. Defaults to 'STANDARD'",
                    "type": "string",
                    "example": "STANDARD"
                  },
                  "status": {
                    "description": "Order status:<br />\n**draft** - order is not submitted for fulfillment<br />\n**failed** - order was submitted for fulfillment but was not accepted because of an error (problem with address, printfiles, charging, etc.)<br />\n**inreview** - order is being reviewed<br />\n**pending** - order has been submitted for fulfillment<br />\n**canceled** - order is canceled<br />\n**onhold** - order has encountered a problem during the fulfillment that needs to be resolved together with the Printful customer service<br />\n**inprocess** - order is being fulfilled and is no longer cancellable<br />\n**partial** - order is partially fulfilled (some items are shipped already, the rest will follow)<br />\n**fulfilled** - all items are shipped\n",
                    "type": "string",
                    "example": "draft"
                  },
                  "created_at": {
                    "description": "Time when the order was created",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "updated_at": {
                    "description": "Time when the order was updated",
                    "type": "string",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "recipient": {
                    "description": "The recipient data.",
                    "title": "Address",
                    "allOf": [
                      {
                        "type": "object",
                        "title": "AddressReadonly",
                        "description": "Information about the address",
                        "required": [
                          "name",
                          "company",
                          "address1",
                          "address2",
                          "city",
                          "state_code",
                          "state_name",
                          "country_code",
                          "country_name",
                          "zip",
                          "phone",
                          "email",
                          "tax_number"
                        ],
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
                      }
                    ]
                  },
                  "costs": {
                    "type": "object",
                    "title": "Costs",
                    "description": "The Order costs (Printful prices)",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "digitization",
                      "additional_fee",
                      "fulfillment_fee",
                      "retail_delivery_fee",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the costs are returned.",
                        "type": "string",
                        "nullable": true,
                        "example": "USD"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "14.95"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "nullable": true,
                        "example": "1.79"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "4.79"
                      },
                      "digitization": {
                        "description": "Digitization costs.",
                        "type": "string",
                        "nullable": true,
                        "example": "3.95"
                      },
                      "additional_fee": {
                        "description": "Additional fee for custom product.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "fulfillment_fee": {
                        "description": "Custom product fulfillment fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "retail_delivery_fee": {
                        "description": "Retail delivery fee.",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "vat": {
                        "description": "Sum of vat (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "4.60"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "nullable": true,
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "26.50"
                      }
                    }
                  },
                  "retail_costs": {
                    "type": "object",
                    "title": "RetailCosts",
                    "description": "The Order's retail costs",
                    "readOnly": true,
                    "required": [
                      "calculation_status",
                      "currency",
                      "subtotal",
                      "discount",
                      "shipping",
                      "vat",
                      "tax",
                      "total"
                    ],
                    "properties": {
                      "calculation_status": {
                        "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                        "type": "string",
                        "enum": [
                          "done",
                          "calculating",
                          "failed"
                        ],
                        "example": "done"
                      },
                      "currency": {
                        "description": "The code of the currency in which the retail costs are returned.",
                        "type": "string",
                        "example": "EUR"
                      },
                      "subtotal": {
                        "description": "Total cost of all items.",
                        "type": "string",
                        "nullable": true,
                        "example": "26.55"
                      },
                      "discount": {
                        "description": "Discount sum.",
                        "type": "string",
                        "example": "0.00"
                      },
                      "shipping": {
                        "description": "Shipping costs.",
                        "type": "string",
                        "example": "4.79"
                      },
                      "vat": {
                        "description": "Sum of VAT (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "tax": {
                        "description": "Sum of taxes (not included in the item price).",
                        "type": "string",
                        "example": "0.00"
                      },
                      "total": {
                        "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                        "type": "string",
                        "nullable": true,
                        "example": "31.34"
                      }
                    }
                  },
                  "order_items": {
                    "type": "array",
                    "description": "Simplified order item list. For a full list of all items use the [Get Order Items](#operation/getItemsByOrderId) endpoint.",
                    "items": {
                      "discriminator": {
                        "propertyName": "source",
                        "mapping": {
                          "catalog": "#/components/schemas/CatalogItemSummary",
                          "warehouse": "#/components/schemas/WarehouseItemSummary"
                        }
                      },
                      "oneOf": [
                        {
                          "type": "object",
                          "title": "CatalogItemSummary",
                          "description": "Simplified information about the Catalog Item",
                          "required": [
                            "id",
                            "type",
                            "source",
                            "catalog_variant_id",
                            "external_id",
                            "quantity",
                            "price",
                            "retail_price",
                            "currency",
                            "retail_currency",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "catalog",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "Catalog Variant ID associated with the Item",
                              "type": "integer",
                              "example": 4011
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "WarehouseItemSummary",
                          "description": "Simplified information about the Warehouse Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "type": {
                              "description": "The item type",
                              "type": "string",
                              "example": "order_item",
                              "enum": [
                                "order_item",
                                "branding_item"
                              ]
                            },
                            "source": {
                              "description": "Item source",
                              "type": "string",
                              "example": "warehouse",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse product associated with the Item",
                              "type": "integer",
                              "example": 1123581321
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "price": {
                              "description": "The price Printful charges for the Item",
                              "type": "string",
                              "example": "8.00"
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "currency": {
                              "description": "The price currency",
                              "type": "string",
                              "example": "EUR"
                            },
                            "retail_currency": {
                              "description": "The retail price currency",
                              "type": "string",
                              "example": "USD"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  },
                  "customization": {
                    "type": "object",
                    "title": "CustomizationReadonly",
                    "description": "The Order's customization values",
                    "required": [
                      "gift",
                      "packing_slip"
                    ],
                    "properties": {
                      "gift": {
                        "type": "object",
                        "title": "Gift",
                        "description": "The gift subject and message",
                        "properties": {
                          "subject": {
                            "type": "string",
                            "description": "Gift message subject",
                            "example": "To John"
                          },
                          "message": {
                            "type": "string",
                            "description": "Gift message",
                            "example": "Happy birthday!"
                          }
                        }
                      },
                      "packing_slip": {
                        "type": "object",
                        "title": "PackingSlipReadonly",
                        "description": "The values for customized packing slip",
                        "required": [
                          "email",
                          "phone",
                          "message",
                          "logo_url",
                          "store_name",
                          "custom_order_id"
                        ],
                        "properties": {
                          "email": {
                            "type": "string",
                            "description": "Customer service email",
                            "example": "test@example.com"
                          },
                          "phone": {
                            "type": "string",
                            "description": "Customer service phone",
                            "example": "+48000000000"
                          },
                          "message": {
                            "type": "string",
                            "description": "Custom packing slip message",
                            "example": "This is a message"
                          },
                          "logo_url": {
                            "type": "string",
                            "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                            "example": "https://example.com/image.jpg"
                          },
                          "store_name": {
                            "type": "string",
                            "description": "Store name override for the return address",
                            "example": "A store"
                          },
                          "custom_order_id": {
                            "type": "string",
                            "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                            "example": "11235813"
                          }
                        }
                      }
                    }
                  },
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "required": [
                      "self",
                      "order_confirmation",
                      "order_invoices",
                      "order_items",
                      "shipments"
                    ],
                    "properties": {
                      "self": {
                        "description": "Link to the order details",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        },
                        "allOf": [
                          {
                            "type": "object",
                            "title": "HateoasLink",
                            "readOnly": true,
                            "required": [
                              "href"
                            ],
                            "properties": {
                              "href": {
                                "description": "The HREF of the linked resource.",
                                "type": "string"
                              }
                            }
                          }
                        ]
                      },
                      "order_confirmation": {
                        "description": "Link to the order confirmation",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/confirmation"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_invoices": {
                        "description": "Link to the order invoice",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/invoices"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "order_items": {
                        "description": "Link to all order items associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-items"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "shipments": {
                        "description": "Link to the shipments associated with the order",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/shipments"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      }
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self",
                  "order",
                  "order_items",
                  "shipments"
                ],
                "properties": {
                  "self": {
                    "description": "Link to the order details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/confirmation"
                    },
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
                          }
                        }
                      }
                    ]
                  },
                  "order": {
                    "description": "Link to order",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "order_items": {
                    "description": "Link to all order items associated with the order",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "shipments": {
                    "description": "Link to the shipments associated with the order",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/shipments"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### GET /v2/orders/{order_id}/order-items
**Summary:** Retrieve a list of order items

This endpoint retrieves the list of items that belong to the order.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getItemsByOrderId",
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
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "query",
      "name": "type",
      "schema": {
        "type": "string"
      },
      "required": false,
      "description": "Type of items returned (order_item, branding_item). By default all items are returned."
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer",
        "minimum": 1,
        "maximum": 100,
        "default": 20
      },
      "required": false,
      "description": "The number of results to return per page."
    },
    {
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer",
        "minimum": 0,
        "default": 0
      },
      "required": false,
      "examples": {
        "startAt100": {
          "summary": "Return results after the initial 100",
          "value": 100
        }
      },
      "description": "The number of results to not include in the response starting from the beginning of the list.\n\nThis can be used to return results after the initial 100. For example, sending offset 100\n"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "oneOf": [
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "placements",
                            "catalog_variant_id",
                            "source"
                          ],
                          "properties": {
                            "source": {
                              "description": "Catalog source",
                              "type": "string",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "ID of catalog variant",
                              "type": "integer",
                              "example": 4011
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "id",
                            "external_id",
                            "quantity",
                            "retail_price",
                            "name",
                            "placements",
                            "product_options",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "placements": {
                              "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Placement",
                                "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                                "required": [
                                  "placement",
                                  "technique",
                                  "layers",
                                  "status",
                                  "status_explanation"
                                ],
                                "properties": {
                                  "placement": {
                                    "description": "Name of the placement",
                                    "type": "string",
                                    "example": "front"
                                  },
                                  "technique": {
                                    "description": "Placement's technique",
                                    "type": "string",
                                    "example": "dtg"
                                  },
                                  "print_area_type": {
                                    "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                    "type": "string",
                                    "writeOnly": true,
                                    "default": "simple",
                                    "enum": [
                                      "simple",
                                      "advanced"
                                    ]
                                  },
                                  "layers": {
                                    "description": "Information about placement's layers",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "Layer",
                                      "description": "Information about the Layer",
                                      "required": [
                                        "type",
                                        "url"
                                      ],
                                      "properties": {
                                        "type": {
                                          "description": "Type of layer (e.g. file, text)",
                                          "type": "string",
                                          "example": "file"
                                        },
                                        "url": {
                                          "description": "File image URL if layer type is file",
                                          "type": "string",
                                          "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                        },
                                        "layer_options": {
                                          "description": "List of layer options",
                                          "type": "array",
                                          "items": {
                                            "oneOf": [
                                              {
                                                "type": "object",
                                                "title": "3dPuffOption",
                                                "description": "Should thread use 3d puff technique",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "3d_puff",
                                                    "enum": [
                                                      "3d_puff"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Whether the 3d puff technique should be used for the layer.",
                                                    "type": "boolean",
                                                    "example": true
                                                  }
                                                }
                                              },
                                              {
                                                "type": "object",
                                                "title": "ThreadColorsOption",
                                                "description": "Specify thread colors for embroidery technique",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "thread_colors",
                                                    "enum": [
                                                      "thread_colors"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Thread colors for embroidery technique",
                                                    "type": "array",
                                                    "items": {
                                                      "type": "string",
                                                      "example": "#FFFFFF"
                                                    }
                                                  }
                                                }
                                              },
                                              {
                                                "type": "object",
                                                "title": "KnitwearYarnColorsOption",
                                                "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "yarn_colors",
                                                    "enum": [
                                                      "yarn_colors"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Option value",
                                                    "type": "array",
                                                    "items": {
                                                      "description": "Option value",
                                                      "type": "string",
                                                      "example": "#fdfafa",
                                                      "enum": [
                                                        "#090909",
                                                        "#404040",
                                                        "#563c33",
                                                        "#d52213",
                                                        "#6e5242",
                                                        "#7f6a53",
                                                        "#cd5e38",
                                                        "#b57648",
                                                        "#d1773b",
                                                        "#d68785",
                                                        "#c6b5a7",
                                                        "#d6c6b4",
                                                        "#dcd3cc",
                                                        "#edd9d9",
                                                        "#e2dfdc",
                                                        "#fdfafa",
                                                        "#999996",
                                                        "#dda032",
                                                        "#d1c6ae",
                                                        "#eddea4",
                                                        "#48542e",
                                                        "#6e8c4b",
                                                        "#c0c1bd",
                                                        "#243f33",
                                                        "#c5d1d0",
                                                        "#175387",
                                                        "#237d96",
                                                        "#787979",
                                                        "#343d55",
                                                        "#4e59be",
                                                        "#566e99",
                                                        "#504372",
                                                        "#4c1c29",
                                                        "#f66274",
                                                        "#eda6b4",
                                                        "#ddabc8"
                                                      ]
                                                    }
                                                  }
                                                }
                                              }
                                            ]
                                          }
                                        },
                                        "position": {
                                          "type": "object",
                                          "title": "LayerPosition",
                                          "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                          "required": [
                                            "top",
                                            "left",
                                            "width",
                                            "height"
                                          ],
                                          "properties": {
                                            "width": {
                                              "description": "Layer width in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "minimum": 0.3,
                                              "example": 10
                                            },
                                            "height": {
                                              "description": "Layer height in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "minimum": 0.3,
                                              "example": 10
                                            },
                                            "top": {
                                              "description": "Layer top position in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "example": 0
                                            },
                                            "left": {
                                              "description": "Layer left position in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "example": 0
                                            }
                                          }
                                        }
                                      }
                                    }
                                  },
                                  "placement_options": {
                                    "description": "List of placement options",
                                    "type": "array",
                                    "items": {
                                      "oneOf": [
                                        {
                                          "type": "object",
                                          "title": "UnlimitedColorOption",
                                          "description": "Specify if the design should use unlimited color technique",
                                          "required": [
                                            "name",
                                            "value"
                                          ],
                                          "properties": {
                                            "name": {
                                              "description": "Option identifier",
                                              "type": "string",
                                              "example": "unlimited_color",
                                              "enum": [
                                                "unlimited_color"
                                              ]
                                            },
                                            "value": {
                                              "description": "Whether the design should use unlimited color technique.",
                                              "type": "boolean",
                                              "example": true
                                            }
                                          }
                                        },
                                        {
                                          "type": "object",
                                          "title": "InsideLabelTypeOption",
                                          "description": "Specify the type of inside label",
                                          "required": [
                                            "name",
                                            "value"
                                          ],
                                          "properties": {
                                            "name": {
                                              "description": "Option identifier",
                                              "type": "string",
                                              "example": "inside_label_type",
                                              "enum": [
                                                "inside_label_type"
                                              ]
                                            },
                                            "value": {
                                              "description": "Specifies type of inside label design that should be used",
                                              "type": "string",
                                              "enum": [
                                                "native",
                                                "custom"
                                              ]
                                            }
                                          }
                                        }
                                      ]
                                    }
                                  },
                                  "status": {
                                    "type": "string",
                                    "readOnly": true,
                                    "enum": [
                                      "ok",
                                      "failed"
                                    ],
                                    "description": "Status of the placement design"
                                  },
                                  "status_explanation": {
                                    "type": "string",
                                    "readOnly": true,
                                    "example": "Product with ID: 656 cannot have disjointed design elements.",
                                    "description": "Reason behind failed status"
                                  }
                                }
                              }
                            },
                            "product_options": {
                              "description": "List of product options",
                              "type": "array",
                              "items": {
                                "oneOf": [
                                  {
                                    "type": "object",
                                    "title": "InsidePocketOption",
                                    "description": "Specify if inside pocket should be added to the product",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "inside_pocket",
                                        "enum": [
                                          "inside_pocket"
                                        ]
                                      },
                                      "value": {
                                        "description": "Whether inside pocket should be added to the product.",
                                        "type": "boolean",
                                        "example": true
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "StitchColorOption",
                                    "description": "Specified what color should be used for stitches",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "stitch_color",
                                        "enum": [
                                          "stitch_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "white"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "NotesOption",
                                    "description": "Include additional notes for fulfillment for embroidery prints",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "notes",
                                        "enum": [
                                          "notes"
                                        ]
                                      },
                                      "value": {
                                        "description": "Additional notes for fulfillment for embroidery prints.",
                                        "type": "string",
                                        "example": "Please make sure that top side of the print is within print area"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "LifelikeOption",
                                    "description": "Specifies if generated mockup should use lifelike effect",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "lifelike",
                                        "enum": [
                                          "lifelike"
                                        ]
                                      },
                                      "value": {
                                        "description": "Whether generated mockup should use lifelike effect.",
                                        "type": "boolean",
                                        "example": true
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "CustomBorderColorOption",
                                    "description": "Used to specify the border color of a sticker",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "custom_border_color",
                                        "enum": [
                                          "custom_border_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Color defined in hexadecimal format",
                                        "type": "string",
                                        "example": "#FF0000"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearBaseColorOption",
                                    "description": "Used to specify the base color on a knitwear product.",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "base_color",
                                        "enum": [
                                          "base_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "#fdfafa",
                                        "enum": [
                                          "#090909",
                                          "#404040",
                                          "#563c33",
                                          "#d52213",
                                          "#6e5242",
                                          "#7f6a53",
                                          "#cd5e38",
                                          "#b57648",
                                          "#d1773b",
                                          "#d68785",
                                          "#c6b5a7",
                                          "#d6c6b4",
                                          "#dcd3cc",
                                          "#edd9d9",
                                          "#e2dfdc",
                                          "#fdfafa",
                                          "#999996",
                                          "#dda032",
                                          "#d1c6ae",
                                          "#eddea4",
                                          "#48542e",
                                          "#6e8c4b",
                                          "#c0c1bd",
                                          "#243f33",
                                          "#c5d1d0",
                                          "#175387",
                                          "#237d96",
                                          "#787979",
                                          "#343d55",
                                          "#4e59be",
                                          "#566e99",
                                          "#504372",
                                          "#4c1c29",
                                          "#f66274",
                                          "#eda6b4",
                                          "#ddabc8"
                                        ]
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearTrimColorOption",
                                    "description": "Used to specify the color of the trim on a knitwear product.",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "trim_color",
                                        "enum": [
                                          "trim_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "#fdfafa",
                                        "enum": [
                                          "#090909",
                                          "#404040",
                                          "#563c33",
                                          "#d52213",
                                          "#6e5242",
                                          "#7f6a53",
                                          "#cd5e38",
                                          "#b57648",
                                          "#d1773b",
                                          "#d68785",
                                          "#c6b5a7",
                                          "#d6c6b4",
                                          "#dcd3cc",
                                          "#edd9d9",
                                          "#e2dfdc",
                                          "#fdfafa",
                                          "#999996",
                                          "#dda032",
                                          "#d1c6ae",
                                          "#eddea4",
                                          "#48542e",
                                          "#6e8c4b",
                                          "#c0c1bd",
                                          "#243f33",
                                          "#c5d1d0",
                                          "#175387",
                                          "#237d96",
                                          "#787979",
                                          "#343d55",
                                          "#4e59be",
                                          "#566e99",
                                          "#504372",
                                          "#4c1c29",
                                          "#f66274",
                                          "#eda6b4",
                                          "#ddabc8"
                                        ]
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearColorReductionMode",
                                    "description": "Used to set the color reduction mode for a knitwear product",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "color_reduction_mode",
                                        "enum": [
                                          "color_reduction_mode"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "enum": [
                                          "solid",
                                          "pixelated"
                                        ],
                                        "example": "pixelated"
                                      }
                                    }
                                  }
                                ],
                                "discriminator": {
                                  "propertyName": "name",
                                  "mapping": {
                                    "inside_pocket": "#/components/schemas/InsidePocketOption",
                                    "stitch_color": "#/components/schemas/StitchColorOption",
                                    "notes": "#/components/schemas/NotesOption",
                                    "lifelike": "#/components/schemas/LifelikeOption",
                                    "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                    "base_color": "#/components/schemas/KnitwearBaseColor",
                                    "trim_color": "#/components/schemas/KnitwearTrimColor",
                                    "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                  }
                                }
                              }
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    },
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Warehouse Item",
                          "description": "Information about the Warehouse Item",
                          "required": [
                            "source",
                            "warehouse_product_variant_id"
                          ],
                          "properties": {
                            "source": {
                              "description": "Warehouse source",
                              "type": "string",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse variant",
                              "type": "integer",
                              "example": 1234
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "id",
                            "external_id",
                            "quantity",
                            "retail_price",
                            "name",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    },
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Product Template Item",
                          "description": "Information about the Product Template.",
                          "required": [
                            "source",
                            "product_template_id",
                            "catalog_variant_id"
                          ],
                          "properties": {
                            "source": {
                              "description": "Product Template source.",
                              "type": "string",
                              "enum": [
                                "product_template"
                              ]
                            },
                            "product_template_id": {
                              "oneOf": [
                                {
                                  "type": "integer"
                                },
                                {
                                  "type": "string"
                                }
                              ],
                              "description": "ID of Product Template. Allows external ID.",
                              "example": "@12345"
                            },
                            "catalog_variant_id": {
                              "description": "ID of Catalog variant. Must correspond to Product Template.",
                              "type": "integer",
                              "example": 123
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  ],
                  "discriminator": {
                    "propertyName": "source",
                    "mapping": {
                      "catalog": "#/components/schemas/CatalogItemReadonly",
                      "warehouse": "#/components/schemas/WarehouseItemReadonly",
                      "product_template": "#/components/schemas/ProductTemplateItem"
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self",
                  "order"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=20"
                    },
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
                          }
                        }
                      }
                    ]
                  },
                  "next": {
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=30"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "previous": {
                    "description": "Link to the previous page (absent if the current page is the first one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "first": {
                    "description": "Link to the first page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "last": {
                    "description": "Link to the last page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=90"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "order": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to the order",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        }
                      }
                    ]
                  }
                }
              }
            }
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
    "403": {
      "description": "Forbidden",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `403`",
                "example": 403
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "This endpoint requires Oauth authentication!."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "integer",
                    "example": 403
                  },
                  "message": {
                    "type": "string",
                    "example": "This endpoint requires Oauth authentication!."
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
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### POST /v2/orders/{order_id}/order-items
**Summary:** Create a new order item

This endpoint allows the creation of a new item that will be added to an existing order.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createItemByOrderId",
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
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
          "discriminator": {
            "propertyName": "source",
            "mapping": {
              "catalog": "#/components/schemas/CatalogItem",
              "product_template": "#/components/schemas/ProductTemplateItem"
            }
          },
          "oneOf": [
            {
              "allOf": [
                {
                  "type": "object",
                  "title": "Item",
                  "description": "Information about the Item",
                  "required": [
                    "placements",
                    "catalog_variant_id",
                    "source"
                  ],
                  "properties": {
                    "source": {
                      "description": "Catalog source",
                      "type": "string",
                      "enum": [
                        "catalog"
                      ]
                    },
                    "catalog_variant_id": {
                      "description": "ID of catalog variant",
                      "type": "integer",
                      "example": 4011
                    }
                  }
                },
                {
                  "type": "object",
                  "title": "Item",
                  "description": "Information about the Item",
                  "properties": {
                    "id": {
                      "description": "Item ID",
                      "type": "integer",
                      "example": 1234,
                      "readOnly": true
                    },
                    "external_id": {
                      "description": "Item user specified external ID",
                      "type": "string",
                      "nullable": true,
                      "example": "123_abc"
                    },
                    "quantity": {
                      "description": "Item quantity",
                      "type": "integer",
                      "example": 1
                    },
                    "retail_price": {
                      "description": "Item retail price",
                      "type": "string",
                      "example": "10.00"
                    },
                    "name": {
                      "description": "Item custom name",
                      "type": "string",
                      "example": "Custom name"
                    },
                    "placements": {
                      "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "Placement",
                        "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                        "required": [
                          "placement",
                          "technique",
                          "layers",
                          "status",
                          "status_explanation"
                        ],
                        "properties": {
                          "placement": {
                            "description": "Name of the placement",
                            "type": "string",
                            "example": "front"
                          },
                          "technique": {
                            "description": "Placement's technique",
                            "type": "string",
                            "example": "dtg"
                          },
                          "print_area_type": {
                            "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                            "type": "string",
                            "writeOnly": true,
                            "default": "simple",
                            "enum": [
                              "simple",
                              "advanced"
                            ]
                          },
                          "layers": {
                            "description": "Information about placement's layers",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Layer",
                              "description": "Information about the Layer",
                              "required": [
                                "type",
                                "url"
                              ],
                              "properties": {
                                "type": {
                                  "description": "Type of layer (e.g. file, text)",
                                  "type": "string",
                                  "example": "file"
                                },
                                "url": {
                                  "description": "File image URL if layer type is file",
                                  "type": "string",
                                  "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                },
                                "layer_options": {
                                  "description": "List of layer options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "3dPuffOption",
                                        "description": "Should thread use 3d puff technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "3d_puff",
                                            "enum": [
                                              "3d_puff"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the 3d puff technique should be used for the layer.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "ThreadColorsOption",
                                        "description": "Specify thread colors for embroidery technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "thread_colors",
                                            "enum": [
                                              "thread_colors"
                                            ]
                                          },
                                          "value": {
                                            "description": "Thread colors for embroidery technique",
                                            "type": "array",
                                            "items": {
                                              "type": "string",
                                              "example": "#FFFFFF"
                                            }
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "KnitwearYarnColorsOption",
                                        "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "yarn_colors",
                                            "enum": [
                                              "yarn_colors"
                                            ]
                                          },
                                          "value": {
                                            "description": "Option value",
                                            "type": "array",
                                            "items": {
                                              "description": "Option value",
                                              "type": "string",
                                              "example": "#fdfafa",
                                              "enum": [
                                                "#090909",
                                                "#404040",
                                                "#563c33",
                                                "#d52213",
                                                "#6e5242",
                                                "#7f6a53",
                                                "#cd5e38",
                                                "#b57648",
                                                "#d1773b",
                                                "#d68785",
                                                "#c6b5a7",
                                                "#d6c6b4",
                                                "#dcd3cc",
                                                "#edd9d9",
                                                "#e2dfdc",
                                                "#fdfafa",
                                                "#999996",
                                                "#dda032",
                                                "#d1c6ae",
                                                "#eddea4",
                                                "#48542e",
                                                "#6e8c4b",
                                                "#c0c1bd",
                                                "#243f33",
                                                "#c5d1d0",
                                                "#175387",
                                                "#237d96",
                                                "#787979",
                                                "#343d55",
                                                "#4e59be",
                                                "#566e99",
                                                "#504372",
                                                "#4c1c29",
                                                "#f66274",
                                                "#eda6b4",
                                                "#ddabc8"
                                              ]
                                            }
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "position": {
                                  "type": "object",
                                  "title": "LayerPosition",
                                  "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                  "required": [
                                    "top",
                                    "left",
                                    "width",
                                    "height"
                                  ],
                                  "properties": {
                                    "width": {
                                      "description": "Layer width in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "minimum": 0.3,
                                      "example": 10
                                    },
                                    "height": {
                                      "description": "Layer height in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "minimum": 0.3,
                                      "example": 10
                                    },
                                    "top": {
                                      "description": "Layer top position in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "example": 0
                                    },
                                    "left": {
                                      "description": "Layer left position in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "example": 0
                                    }
                                  }
                                }
                              }
                            }
                          },
                          "placement_options": {
                            "description": "List of placement options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "UnlimitedColorOption",
                                  "description": "Specify if the design should use unlimited color technique",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "unlimited_color",
                                      "enum": [
                                        "unlimited_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether the design should use unlimited color technique.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "InsideLabelTypeOption",
                                  "description": "Specify the type of inside label",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_label_type",
                                      "enum": [
                                        "inside_label_type"
                                      ]
                                    },
                                    "value": {
                                      "description": "Specifies type of inside label design that should be used",
                                      "type": "string",
                                      "enum": [
                                        "native",
                                        "custom"
                                      ]
                                    }
                                  }
                                }
                              ]
                            }
                          },
                          "status": {
                            "type": "string",
                            "readOnly": true,
                            "enum": [
                              "ok",
                              "failed"
                            ],
                            "description": "Status of the placement design"
                          },
                          "status_explanation": {
                            "type": "string",
                            "readOnly": true,
                            "example": "Product with ID: 656 cannot have disjointed design elements.",
                            "description": "Reason behind failed status"
                          }
                        }
                      }
                    },
                    "orientation": {
                      "description": "Orientation of the design. Applies only to the products that allow multiple orientations such as framed posters.",
                      "type": "string",
                      "readOnly": false,
                      "nullable": true,
                      "enum": [
                        "horizontal",
                        "vertical",
                        "any"
                      ],
                      "example": "horizontal"
                    },
                    "product_options": {
                      "description": "List of product options",
                      "type": "array",
                      "items": {
                        "oneOf": [
                          {
                            "type": "object",
                            "title": "InsidePocketOption",
                            "description": "Specify if inside pocket should be added to the product",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "inside_pocket",
                                "enum": [
                                  "inside_pocket"
                                ]
                              },
                              "value": {
                                "description": "Whether inside pocket should be added to the product.",
                                "type": "boolean",
                                "example": true
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "StitchColorOption",
                            "description": "Specified what color should be used for stitches",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "stitch_color",
                                "enum": [
                                  "stitch_color"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "example": "white"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "NotesOption",
                            "description": "Include additional notes for fulfillment for embroidery prints",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "notes",
                                "enum": [
                                  "notes"
                                ]
                              },
                              "value": {
                                "description": "Additional notes for fulfillment for embroidery prints.",
                                "type": "string",
                                "example": "Please make sure that top side of the print is within print area"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "LifelikeOption",
                            "description": "Specifies if generated mockup should use lifelike effect",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "lifelike",
                                "enum": [
                                  "lifelike"
                                ]
                              },
                              "value": {
                                "description": "Whether generated mockup should use lifelike effect.",
                                "type": "boolean",
                                "example": true
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "CustomBorderColorOption",
                            "description": "Used to specify the border color of a sticker",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "custom_border_color",
                                "enum": [
                                  "custom_border_color"
                                ]
                              },
                              "value": {
                                "description": "Color defined in hexadecimal format",
                                "type": "string",
                                "example": "#FF0000"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "KnitwearBaseColorOption",
                            "description": "Used to specify the base color on a knitwear product.",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "base_color",
                                "enum": [
                                  "base_color"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "example": "#fdfafa",
                                "enum": [
                                  "#090909",
                                  "#404040",
                                  "#563c33",
                                  "#d52213",
                                  "#6e5242",
                                  "#7f6a53",
                                  "#cd5e38",
                                  "#b57648",
                                  "#d1773b",
                                  "#d68785",
                                  "#c6b5a7",
                                  "#d6c6b4",
                                  "#dcd3cc",
                                  "#edd9d9",
                                  "#e2dfdc",
                                  "#fdfafa",
                                  "#999996",
                                  "#dda032",
                                  "#d1c6ae",
                                  "#eddea4",
                                  "#48542e",
                                  "#6e8c4b",
                                  "#c0c1bd",
                                  "#243f33",
                                  "#c5d1d0",
                                  "#175387",
                                  "#237d96",
                                  "#787979",
                                  "#343d55",
                                  "#4e59be",
                                  "#566e99",
                                  "#504372",
                                  "#4c1c29",
                                  "#f66274",
                                  "#eda6b4",
                                  "#ddabc8"
                                ]
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "KnitwearTrimColorOption",
                            "description": "Used to specify the color of the trim on a knitwear product.",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "trim_color",
                                "enum": [
                                  "trim_color"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "example": "#fdfafa",
                                "enum": [
                                  "#090909",
                                  "#404040",
                                  "#563c33",
                                  "#d52213",
                                  "#6e5242",
                                  "#7f6a53",
                                  "#cd5e38",
                                  "#b57648",
                                  "#d1773b",
                                  "#d68785",
                                  "#c6b5a7",
                                  "#d6c6b4",
                                  "#dcd3cc",
                                  "#edd9d9",
                                  "#e2dfdc",
                                  "#fdfafa",
                                  "#999996",
                                  "#dda032",
                                  "#d1c6ae",
                                  "#eddea4",
                                  "#48542e",
                                  "#6e8c4b",
                                  "#c0c1bd",
                                  "#243f33",
                                  "#c5d1d0",
                                  "#175387",
                                  "#237d96",
                                  "#787979",
                                  "#343d55",
                                  "#4e59be",
                                  "#566e99",
                                  "#504372",
                                  "#4c1c29",
                                  "#f66274",
                                  "#eda6b4",
                                  "#ddabc8"
                                ]
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "KnitwearColorReductionMode",
                            "description": "Used to set the color reduction mode for a knitwear product",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "color_reduction_mode",
                                "enum": [
                                  "color_reduction_mode"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "enum": [
                                  "solid",
                                  "pixelated"
                                ],
                                "example": "pixelated"
                              }
                            }
                          }
                        ],
                        "discriminator": {
                          "propertyName": "name",
                          "mapping": {
                            "inside_pocket": "#/components/schemas/InsidePocketOption",
                            "stitch_color": "#/components/schemas/StitchColorOption",
                            "notes": "#/components/schemas/NotesOption",
                            "lifelike": "#/components/schemas/LifelikeOption",
                            "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                            "base_color": "#/components/schemas/KnitwearBaseColor",
                            "trim_color": "#/components/schemas/KnitwearTrimColor",
                            "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                          }
                        }
                      }
                    },
                    "_links": {
                      "type": "object",
                      "readOnly": true,
                      "description": "HATEOAS links",
                      "properties": {
                        "self": {
                          "allOf": [
                            {
                              "type": "object",
                              "title": "HateoasLink",
                              "readOnly": true,
                              "required": [
                                "href"
                              ],
                              "properties": {
                                "href": {
                                  "description": "The HREF of the linked resource.",
                                  "type": "string"
                                }
                              }
                            },
                            {
                              "description": "Link to a specific order item",
                              "format": "HateoasLink",
                              "example": {
                                "href": "/v2/orders/123/order-items/123"
                              }
                            }
                          ]
                        }
                      }
                    }
                  }
                }
              ]
            },
            {
              "allOf": [
                {
                  "type": "object",
                  "title": "Product Template Item",
                  "description": "Information about the Product Template.",
                  "required": [
                    "source",
                    "product_template_id",
                    "catalog_variant_id"
                  ],
                  "properties": {
                    "source": {
                      "description": "Product Template source.",
                      "type": "string",
                      "enum": [
                        "product_template"
                      ]
                    },
                    "product_template_id": {
                      "oneOf": [
                        {
                          "type": "integer"
                        },
                        {
                          "type": "string"
                        }
                      ],
                      "description": "ID of Product Template. Allows external ID.",
                      "example": "@12345"
                    },
                    "catalog_variant_id": {
                      "description": "ID of Catalog variant. Must correspond to Product Template.",
                      "type": "integer",
                      "example": 123
                    }
                  }
                },
                {
                  "type": "object",
                  "title": "Item",
                  "description": "Information about the Item",
                  "properties": {
                    "id": {
                      "description": "Item ID",
                      "type": "integer",
                      "example": 1234,
                      "readOnly": true
                    },
                    "external_id": {
                      "description": "Item user specified external ID",
                      "type": "string",
                      "nullable": true,
                      "example": "123_abc"
                    },
                    "quantity": {
                      "description": "Item quantity",
                      "type": "integer",
                      "example": 1
                    },
                    "retail_price": {
                      "description": "Item retail price",
                      "type": "string",
                      "example": "10.00"
                    },
                    "name": {
                      "description": "Item custom name",
                      "type": "string",
                      "example": "Custom name"
                    },
                    "_links": {
                      "type": "object",
                      "readOnly": true,
                      "description": "HATEOAS links",
                      "properties": {
                        "self": {
                          "allOf": [
                            {
                              "type": "object",
                              "title": "HateoasLink",
                              "readOnly": true,
                              "required": [
                                "href"
                              ],
                              "properties": {
                                "href": {
                                  "description": "The HREF of the linked resource.",
                                  "type": "string"
                                }
                              }
                            },
                            {
                              "description": "Link to a specific order item",
                              "format": "HateoasLink",
                              "example": {
                                "href": "/v2/orders/123/order-items/123"
                              }
                            }
                          ]
                        }
                      }
                    }
                  }
                }
              ]
            }
          ]
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
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "oneOf": [
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "placements",
                            "catalog_variant_id",
                            "source"
                          ],
                          "properties": {
                            "source": {
                              "description": "Catalog source",
                              "type": "string",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "ID of catalog variant",
                              "type": "integer",
                              "example": 4011
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "id",
                            "external_id",
                            "quantity",
                            "retail_price",
                            "name",
                            "placements",
                            "product_options",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "placements": {
                              "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Placement",
                                "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                                "required": [
                                  "placement",
                                  "technique",
                                  "layers",
                                  "status",
                                  "status_explanation"
                                ],
                                "properties": {
                                  "placement": {
                                    "description": "Name of the placement",
                                    "type": "string",
                                    "example": "front"
                                  },
                                  "technique": {
                                    "description": "Placement's technique",
                                    "type": "string",
                                    "example": "dtg"
                                  },
                                  "print_area_type": {
                                    "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                    "type": "string",
                                    "writeOnly": true,
                                    "default": "simple",
                                    "enum": [
                                      "simple",
                                      "advanced"
                                    ]
                                  },
                                  "layers": {
                                    "description": "Information about placement's layers",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "Layer",
                                      "description": "Information about the Layer",
                                      "required": [
                                        "type",
                                        "url"
                                      ],
                                      "properties": {
                                        "type": {
                                          "description": "Type of layer (e.g. file, text)",
                                          "type": "string",
                                          "example": "file"
                                        },
                                        "url": {
                                          "description": "File image URL if layer type is file",
                                          "type": "string",
                                          "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                        },
                                        "layer_options": {
                                          "description": "List of layer options",
                                          "type": "array",
                                          "items": {
                                            "oneOf": [
                                              {
                                                "type": "object",
                                                "title": "3dPuffOption",
                                                "description": "Should thread use 3d puff technique",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "3d_puff",
                                                    "enum": [
                                                      "3d_puff"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Whether the 3d puff technique should be used for the layer.",
                                                    "type": "boolean",
                                                    "example": true
                                                  }
                                                }
                                              },
                                              {
                                                "type": "object",
                                                "title": "ThreadColorsOption",
                                                "description": "Specify thread colors for embroidery technique",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "thread_colors",
                                                    "enum": [
                                                      "thread_colors"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Thread colors for embroidery technique",
                                                    "type": "array",
                                                    "items": {
                                                      "type": "string",
                                                      "example": "#FFFFFF"
                                                    }
                                                  }
                                                }
                                              },
                                              {
                                                "type": "object",
                                                "title": "KnitwearYarnColorsOption",
                                                "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "yarn_colors",
                                                    "enum": [
                                                      "yarn_colors"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Option value",
                                                    "type": "array",
                                                    "items": {
                                                      "description": "Option value",
                                                      "type": "string",
                                                      "example": "#fdfafa",
                                                      "enum": [
                                                        "#090909",
                                                        "#404040",
                                                        "#563c33",
                                                        "#d52213",
                                                        "#6e5242",
                                                        "#7f6a53",
                                                        "#cd5e38",
                                                        "#b57648",
                                                        "#d1773b",
                                                        "#d68785",
                                                        "#c6b5a7",
                                                        "#d6c6b4",
                                                        "#dcd3cc",
                                                        "#edd9d9",
                                                        "#e2dfdc",
                                                        "#fdfafa",
                                                        "#999996",
                                                        "#dda032",
                                                        "#d1c6ae",
                                                        "#eddea4",
                                                        "#48542e",
                                                        "#6e8c4b",
                                                        "#c0c1bd",
                                                        "#243f33",
                                                        "#c5d1d0",
                                                        "#175387",
                                                        "#237d96",
                                                        "#787979",
                                                        "#343d55",
                                                        "#4e59be",
                                                        "#566e99",
                                                        "#504372",
                                                        "#4c1c29",
                                                        "#f66274",
                                                        "#eda6b4",
                                                        "#ddabc8"
                                                      ]
                                                    }
                                                  }
                                                }
                                              }
                                            ]
                                          }
                                        },
                                        "position": {
                                          "type": "object",
                                          "title": "LayerPosition",
                                          "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                          "required": [
                                            "top",
                                            "left",
                                            "width",
                                            "height"
                                          ],
                                          "properties": {
                                            "width": {
                                              "description": "Layer width in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "minimum": 0.3,
                                              "example": 10
                                            },
                                            "height": {
                                              "description": "Layer height in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "minimum": 0.3,
                                              "example": 10
                                            },
                                            "top": {
                                              "description": "Layer top position in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "example": 0
                                            },
                                            "left": {
                                              "description": "Layer left position in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "example": 0
                                            }
                                          }
                                        }
                                      }
                                    }
                                  },
                                  "placement_options": {
                                    "description": "List of placement options",
                                    "type": "array",
                                    "items": {
                                      "oneOf": [
                                        {
                                          "type": "object",
                                          "title": "UnlimitedColorOption",
                                          "description": "Specify if the design should use unlimited color technique",
                                          "required": [
                                            "name",
                                            "value"
                                          ],
                                          "properties": {
                                            "name": {
                                              "description": "Option identifier",
                                              "type": "string",
                                              "example": "unlimited_color",
                                              "enum": [
                                                "unlimited_color"
                                              ]
                                            },
                                            "value": {
                                              "description": "Whether the design should use unlimited color technique.",
                                              "type": "boolean",
                                              "example": true
                                            }
                                          }
                                        },
                                        {
                                          "type": "object",
                                          "title": "InsideLabelTypeOption",
                                          "description": "Specify the type of inside label",
                                          "required": [
                                            "name",
                                            "value"
                                          ],
                                          "properties": {
                                            "name": {
                                              "description": "Option identifier",
                                              "type": "string",
                                              "example": "inside_label_type",
                                              "enum": [
                                                "inside_label_type"
                                              ]
                                            },
                                            "value": {
                                              "description": "Specifies type of inside label design that should be used",
                                              "type": "string",
                                              "enum": [
                                                "native",
                                                "custom"
                                              ]
                                            }
                                          }
                                        }
                                      ]
                                    }
                                  },
                                  "status": {
                                    "type": "string",
                                    "readOnly": true,
                                    "enum": [
                                      "ok",
                                      "failed"
                                    ],
                                    "description": "Status of the placement design"
                                  },
                                  "status_explanation": {
                                    "type": "string",
                                    "readOnly": true,
                                    "example": "Product with ID: 656 cannot have disjointed design elements.",
                                    "description": "Reason behind failed status"
                                  }
                                }
                              }
                            },
                            "product_options": {
                              "description": "List of product options",
                              "type": "array",
                              "items": {
                                "oneOf": [
                                  {
                                    "type": "object",
                                    "title": "InsidePocketOption",
                                    "description": "Specify if inside pocket should be added to the product",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "inside_pocket",
                                        "enum": [
                                          "inside_pocket"
                                        ]
                                      },
                                      "value": {
                                        "description": "Whether inside pocket should be added to the product.",
                                        "type": "boolean",
                                        "example": true
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "StitchColorOption",
                                    "description": "Specified what color should be used for stitches",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "stitch_color",
                                        "enum": [
                                          "stitch_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "white"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "NotesOption",
                                    "description": "Include additional notes for fulfillment for embroidery prints",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "notes",
                                        "enum": [
                                          "notes"
                                        ]
                                      },
                                      "value": {
                                        "description": "Additional notes for fulfillment for embroidery prints.",
                                        "type": "string",
                                        "example": "Please make sure that top side of the print is within print area"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "LifelikeOption",
                                    "description": "Specifies if generated mockup should use lifelike effect",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "lifelike",
                                        "enum": [
                                          "lifelike"
                                        ]
                                      },
                                      "value": {
                                        "description": "Whether generated mockup should use lifelike effect.",
                                        "type": "boolean",
                                        "example": true
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "CustomBorderColorOption",
                                    "description": "Used to specify the border color of a sticker",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "custom_border_color",
                                        "enum": [
                                          "custom_border_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Color defined in hexadecimal format",
                                        "type": "string",
                                        "example": "#FF0000"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearBaseColorOption",
                                    "description": "Used to specify the base color on a knitwear product.",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "base_color",
                                        "enum": [
                                          "base_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "#fdfafa",
                                        "enum": [
                                          "#090909",
                                          "#404040",
                                          "#563c33",
                                          "#d52213",
                                          "#6e5242",
                                          "#7f6a53",
                                          "#cd5e38",
                                          "#b57648",
                                          "#d1773b",
                                          "#d68785",
                                          "#c6b5a7",
                                          "#d6c6b4",
                                          "#dcd3cc",
                                          "#edd9d9",
                                          "#e2dfdc",
                                          "#fdfafa",
                                          "#999996",
                                          "#dda032",
                                          "#d1c6ae",
                                          "#eddea4",
                                          "#48542e",
                                          "#6e8c4b",
                                          "#c0c1bd",
                                          "#243f33",
                                          "#c5d1d0",
                                          "#175387",
                                          "#237d96",
                                          "#787979",
                                          "#343d55",
                                          "#4e59be",
                                          "#566e99",
                                          "#504372",
                                          "#4c1c29",
                                          "#f66274",
                                          "#eda6b4",
                                          "#ddabc8"
                                        ]
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearTrimColorOption",
                                    "description": "Used to specify the color of the trim on a knitwear product.",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "trim_color",
                                        "enum": [
                                          "trim_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "#fdfafa",
                                        "enum": [
                                          "#090909",
                                          "#404040",
                                          "#563c33",
                                          "#d52213",
                                          "#6e5242",
                                          "#7f6a53",
                                          "#cd5e38",
                                          "#b57648",
                                          "#d1773b",
                                          "#d68785",
                                          "#c6b5a7",
                                          "#d6c6b4",
                                          "#dcd3cc",
                                          "#edd9d9",
                                          "#e2dfdc",
                                          "#fdfafa",
                                          "#999996",
                                          "#dda032",
                                          "#d1c6ae",
                                          "#eddea4",
                                          "#48542e",
                                          "#6e8c4b",
                                          "#c0c1bd",
                                          "#243f33",
                                          "#c5d1d0",
                                          "#175387",
                                          "#237d96",
                                          "#787979",
                                          "#343d55",
                                          "#4e59be",
                                          "#566e99",
                                          "#504372",
                                          "#4c1c29",
                                          "#f66274",
                                          "#eda6b4",
                                          "#ddabc8"
                                        ]
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearColorReductionMode",
                                    "description": "Used to set the color reduction mode for a knitwear product",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "color_reduction_mode",
                                        "enum": [
                                          "color_reduction_mode"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "enum": [
                                          "solid",
                                          "pixelated"
                                        ],
                                        "example": "pixelated"
                                      }
                                    }
                                  }
                                ],
                                "discriminator": {
                                  "propertyName": "name",
                                  "mapping": {
                                    "inside_pocket": "#/components/schemas/InsidePocketOption",
                                    "stitch_color": "#/components/schemas/StitchColorOption",
                                    "notes": "#/components/schemas/NotesOption",
                                    "lifelike": "#/components/schemas/LifelikeOption",
                                    "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                    "base_color": "#/components/schemas/KnitwearBaseColor",
                                    "trim_color": "#/components/schemas/KnitwearTrimColor",
                                    "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                  }
                                }
                              }
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    },
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Warehouse Item",
                          "description": "Information about the Warehouse Item",
                          "required": [
                            "source",
                            "warehouse_product_variant_id"
                          ],
                          "properties": {
                            "source": {
                              "description": "Warehouse source",
                              "type": "string",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse variant",
                              "type": "integer",
                              "example": 1234
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "id",
                            "external_id",
                            "quantity",
                            "retail_price",
                            "name",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    },
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Product Template Item",
                          "description": "Information about the Product Template.",
                          "required": [
                            "source",
                            "product_template_id",
                            "catalog_variant_id"
                          ],
                          "properties": {
                            "source": {
                              "description": "Product Template source.",
                              "type": "string",
                              "enum": [
                                "product_template"
                              ]
                            },
                            "product_template_id": {
                              "oneOf": [
                                {
                                  "type": "integer"
                                },
                                {
                                  "type": "string"
                                }
                              ],
                              "description": "ID of Product Template. Allows external ID.",
                              "example": "@12345"
                            },
                            "catalog_variant_id": {
                              "description": "ID of Catalog variant. Must correspond to Product Template.",
                              "type": "integer",
                              "example": 123
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  ],
                  "discriminator": {
                    "propertyName": "source",
                    "mapping": {
                      "catalog": "#/components/schemas/CatalogItemReadonly",
                      "warehouse": "#/components/schemas/WarehouseItemReadonly",
                      "product_template": "#/components/schemas/ProductTemplateItem"
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self",
                  "order"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=20"
                    },
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
                          }
                        }
                      }
                    ]
                  },
                  "next": {
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=30"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "previous": {
                    "description": "Link to the previous page (absent if the current page is the first one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "first": {
                    "description": "Link to the first page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "last": {
                    "description": "Link to the last page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=90"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "order": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to the order",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        }
                      }
                    ]
                  }
                }
              }
            }
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
    "403": {
      "description": "Forbidden",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `403`",
                "example": 403
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "This endpoint requires Oauth authentication!."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "integer",
                    "example": 403
                  },
                  "message": {
                    "type": "string",
                    "example": "This endpoint requires Oauth authentication!."
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
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### GET /v2/orders/{order_id}/order-items/{order_item_id}
**Summary:** Retrieve a single order item

This endpoint will retrieve a single order item specified in the request.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getItemById",
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
      "name": "order_item_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Item ID (integer) or Item External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "discriminator": {
                  "propertyName": "source",
                  "mapping": {
                    "catalog": "#/components/schemas/CatalogItemReadonly",
                    "warehouse": "#/components/schemas/WarehouseItemReadonly",
                    "product_template": "#/components/schemas/ProductTemplateItem"
                  }
                },
                "oneOf": [
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "required": [
                          "placements",
                          "catalog_variant_id",
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Catalog source",
                            "type": "string",
                            "enum": [
                              "catalog"
                            ]
                          },
                          "catalog_variant_id": {
                            "description": "ID of catalog variant",
                            "type": "integer",
                            "example": 4011
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "required": [
                          "id",
                          "external_id",
                          "quantity",
                          "retail_price",
                          "name",
                          "placements",
                          "product_options",
                          "_links"
                        ],
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "placements": {
                            "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Placement",
                              "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                              "required": [
                                "placement",
                                "technique",
                                "layers",
                                "status",
                                "status_explanation"
                              ],
                              "properties": {
                                "placement": {
                                  "description": "Name of the placement",
                                  "type": "string",
                                  "example": "front"
                                },
                                "technique": {
                                  "description": "Placement's technique",
                                  "type": "string",
                                  "example": "dtg"
                                },
                                "print_area_type": {
                                  "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                  "type": "string",
                                  "writeOnly": true,
                                  "default": "simple",
                                  "enum": [
                                    "simple",
                                    "advanced"
                                  ]
                                },
                                "layers": {
                                  "description": "Information about placement's layers",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "Layer",
                                    "description": "Information about the Layer",
                                    "required": [
                                      "type",
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "description": "Type of layer (e.g. file, text)",
                                        "type": "string",
                                        "example": "file"
                                      },
                                      "url": {
                                        "description": "File image URL if layer type is file",
                                        "type": "string",
                                        "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                      },
                                      "layer_options": {
                                        "description": "List of layer options",
                                        "type": "array",
                                        "items": {
                                          "oneOf": [
                                            {
                                              "type": "object",
                                              "title": "3dPuffOption",
                                              "description": "Should thread use 3d puff technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "3d_puff",
                                                  "enum": [
                                                    "3d_puff"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Whether the 3d puff technique should be used for the layer.",
                                                  "type": "boolean",
                                                  "example": true
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "ThreadColorsOption",
                                              "description": "Specify thread colors for embroidery technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "thread_colors",
                                                  "enum": [
                                                    "thread_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Thread colors for embroidery technique",
                                                  "type": "array",
                                                  "items": {
                                                    "type": "string",
                                                    "example": "#FFFFFF"
                                                  }
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "KnitwearYarnColorsOption",
                                              "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "yarn_colors",
                                                  "enum": [
                                                    "yarn_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Option value",
                                                  "type": "array",
                                                  "items": {
                                                    "description": "Option value",
                                                    "type": "string",
                                                    "example": "#fdfafa",
                                                    "enum": [
                                                      "#090909",
                                                      "#404040",
                                                      "#563c33",
                                                      "#d52213",
                                                      "#6e5242",
                                                      "#7f6a53",
                                                      "#cd5e38",
                                                      "#b57648",
                                                      "#d1773b",
                                                      "#d68785",
                                                      "#c6b5a7",
                                                      "#d6c6b4",
                                                      "#dcd3cc",
                                                      "#edd9d9",
                                                      "#e2dfdc",
                                                      "#fdfafa",
                                                      "#999996",
                                                      "#dda032",
                                                      "#d1c6ae",
                                                      "#eddea4",
                                                      "#48542e",
                                                      "#6e8c4b",
                                                      "#c0c1bd",
                                                      "#243f33",
                                                      "#c5d1d0",
                                                      "#175387",
                                                      "#237d96",
                                                      "#787979",
                                                      "#343d55",
                                                      "#4e59be",
                                                      "#566e99",
                                                      "#504372",
                                                      "#4c1c29",
                                                      "#f66274",
                                                      "#eda6b4",
                                                      "#ddabc8"
                                                    ]
                                                  }
                                                }
                                              }
                                            }
                                          ]
                                        }
                                      },
                                      "position": {
                                        "type": "object",
                                        "title": "LayerPosition",
                                        "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                        "required": [
                                          "top",
                                          "left",
                                          "width",
                                          "height"
                                        ],
                                        "properties": {
                                          "width": {
                                            "description": "Layer width in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "height": {
                                            "description": "Layer height in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "top": {
                                            "description": "Layer top position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          },
                                          "left": {
                                            "description": "Layer left position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          }
                                        }
                                      }
                                    }
                                  }
                                },
                                "placement_options": {
                                  "description": "List of placement options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "UnlimitedColorOption",
                                        "description": "Specify if the design should use unlimited color technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "unlimited_color",
                                            "enum": [
                                              "unlimited_color"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the design should use unlimited color technique.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "InsideLabelTypeOption",
                                        "description": "Specify the type of inside label",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "inside_label_type",
                                            "enum": [
                                              "inside_label_type"
                                            ]
                                          },
                                          "value": {
                                            "description": "Specifies type of inside label design that should be used",
                                            "type": "string",
                                            "enum": [
                                              "native",
                                              "custom"
                                            ]
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "status": {
                                  "type": "string",
                                  "readOnly": true,
                                  "enum": [
                                    "ok",
                                    "failed"
                                  ],
                                  "description": "Status of the placement design"
                                },
                                "status_explanation": {
                                  "type": "string",
                                  "readOnly": true,
                                  "example": "Product with ID: 656 cannot have disjointed design elements.",
                                  "description": "Reason behind failed status"
                                }
                              }
                            }
                          },
                          "product_options": {
                            "description": "List of product options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "InsidePocketOption",
                                  "description": "Specify if inside pocket should be added to the product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_pocket",
                                      "enum": [
                                        "inside_pocket"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether inside pocket should be added to the product.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "StitchColorOption",
                                  "description": "Specified what color should be used for stitches",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "stitch_color",
                                      "enum": [
                                        "stitch_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "white"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "NotesOption",
                                  "description": "Include additional notes for fulfillment for embroidery prints",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "notes",
                                      "enum": [
                                        "notes"
                                      ]
                                    },
                                    "value": {
                                      "description": "Additional notes for fulfillment for embroidery prints.",
                                      "type": "string",
                                      "example": "Please make sure that top side of the print is within print area"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "LifelikeOption",
                                  "description": "Specifies if generated mockup should use lifelike effect",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "lifelike",
                                      "enum": [
                                        "lifelike"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether generated mockup should use lifelike effect.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "CustomBorderColorOption",
                                  "description": "Used to specify the border color of a sticker",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "custom_border_color",
                                      "enum": [
                                        "custom_border_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Color defined in hexadecimal format",
                                      "type": "string",
                                      "example": "#FF0000"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearBaseColorOption",
                                  "description": "Used to specify the base color on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "base_color",
                                      "enum": [
                                        "base_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearTrimColorOption",
                                  "description": "Used to specify the color of the trim on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "trim_color",
                                      "enum": [
                                        "trim_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearColorReductionMode",
                                  "description": "Used to set the color reduction mode for a knitwear product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "color_reduction_mode",
                                      "enum": [
                                        "color_reduction_mode"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "enum": [
                                        "solid",
                                        "pixelated"
                                      ],
                                      "example": "pixelated"
                                    }
                                  }
                                }
                              ],
                              "discriminator": {
                                "propertyName": "name",
                                "mapping": {
                                  "inside_pocket": "#/components/schemas/InsidePocketOption",
                                  "stitch_color": "#/components/schemas/StitchColorOption",
                                  "notes": "#/components/schemas/NotesOption",
                                  "lifelike": "#/components/schemas/LifelikeOption",
                                  "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                  "base_color": "#/components/schemas/KnitwearBaseColor",
                                  "trim_color": "#/components/schemas/KnitwearTrimColor",
                                  "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                }
                              }
                            }
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "required": [
                              "self"
                            ],
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  },
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Warehouse Item",
                        "description": "Information about the Warehouse Item",
                        "required": [
                          "source",
                          "warehouse_product_variant_id"
                        ],
                        "properties": {
                          "source": {
                            "description": "Warehouse source",
                            "type": "string",
                            "enum": [
                              "warehouse"
                            ]
                          },
                          "warehouse_product_variant_id": {
                            "description": "ID of warehouse variant",
                            "type": "integer",
                            "example": 1234
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "required": [
                          "id",
                          "external_id",
                          "quantity",
                          "retail_price",
                          "name",
                          "_links"
                        ],
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "required": [
                              "self"
                            ],
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  },
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Product Template Item",
                        "description": "Information about the Product Template.",
                        "required": [
                          "source",
                          "product_template_id",
                          "catalog_variant_id"
                        ],
                        "properties": {
                          "source": {
                            "description": "Product Template source.",
                            "type": "string",
                            "enum": [
                              "product_template"
                            ]
                          },
                          "product_template_id": {
                            "oneOf": [
                              {
                                "type": "integer"
                              },
                              {
                                "type": "string"
                              }
                            ],
                            "description": "ID of Product Template. Allows external ID.",
                            "example": "@12345"
                          },
                          "catalog_variant_id": {
                            "description": "ID of Catalog variant. Must correspond to Product Template.",
                            "type": "integer",
                            "example": 123
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  }
                ]
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self",
                  "order",
                  "all_items",
                  "shipments"
                ],
                "properties": {
                  "self": {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
                          }
                        }
                      },
                      {
                        "description": "Link to a specific order item",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-items/123"
                        }
                      }
                    ]
                  },
                  "order": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to a specific order",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        }
                      }
                    ]
                  },
                  "all_items": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to all items associated with the order",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-items"
                        }
                      }
                    ]
                  },
                  "shipments": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to all shipments associated with the order",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123/order-shipments"
                        }
                      }
                    ]
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### PATCH /v2/orders/{order_id}/order-items/{order_item_id}
**Summary:** Update an order item

Make a partial update of an order item. NOTE that the source of the order item can't be changed via a PATCH request, to create an order item from another source you must delete the current one and add a new one.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "updateItem",
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "order_item_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Item ID (integer) or Item External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    }
  ],
  "requestBody": {
    "description": "PATCH request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "oneOf": [
            {
              "allOf": [
                {
                  "type": "object",
                  "title": "Item",
                  "description": "Information about the Item",
                  "required": [
                    "placements",
                    "catalog_variant_id",
                    "source"
                  ],
                  "properties": {
                    "source": {
                      "description": "Catalog source",
                      "type": "string",
                      "enum": [
                        "catalog"
                      ]
                    },
                    "catalog_variant_id": {
                      "description": "ID of catalog variant",
                      "type": "integer",
                      "example": 4011
                    }
                  }
                },
                {
                  "type": "object",
                  "title": "Item",
                  "description": "Information about the Item",
                  "properties": {
                    "id": {
                      "description": "Item ID",
                      "type": "integer",
                      "example": 1234,
                      "readOnly": true
                    },
                    "external_id": {
                      "description": "Item user specified external ID",
                      "type": "string",
                      "nullable": true,
                      "example": "123_abc"
                    },
                    "quantity": {
                      "description": "Item quantity",
                      "type": "integer",
                      "example": 1
                    },
                    "retail_price": {
                      "description": "Item retail price",
                      "type": "string",
                      "example": "10.00"
                    },
                    "name": {
                      "description": "Item custom name",
                      "type": "string",
                      "example": "Custom name"
                    },
                    "placements": {
                      "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "Placement",
                        "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                        "required": [
                          "placement",
                          "technique",
                          "layers",
                          "status",
                          "status_explanation"
                        ],
                        "properties": {
                          "placement": {
                            "description": "Name of the placement",
                            "type": "string",
                            "example": "front"
                          },
                          "technique": {
                            "description": "Placement's technique",
                            "type": "string",
                            "example": "dtg"
                          },
                          "print_area_type": {
                            "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                            "type": "string",
                            "writeOnly": true,
                            "default": "simple",
                            "enum": [
                              "simple",
                              "advanced"
                            ]
                          },
                          "layers": {
                            "description": "Information about placement's layers",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Layer",
                              "description": "Information about the Layer",
                              "required": [
                                "type",
                                "url"
                              ],
                              "properties": {
                                "type": {
                                  "description": "Type of layer (e.g. file, text)",
                                  "type": "string",
                                  "example": "file"
                                },
                                "url": {
                                  "description": "File image URL if layer type is file",
                                  "type": "string",
                                  "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                },
                                "layer_options": {
                                  "description": "List of layer options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "3dPuffOption",
                                        "description": "Should thread use 3d puff technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "3d_puff",
                                            "enum": [
                                              "3d_puff"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the 3d puff technique should be used for the layer.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "ThreadColorsOption",
                                        "description": "Specify thread colors for embroidery technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "thread_colors",
                                            "enum": [
                                              "thread_colors"
                                            ]
                                          },
                                          "value": {
                                            "description": "Thread colors for embroidery technique",
                                            "type": "array",
                                            "items": {
                                              "type": "string",
                                              "example": "#FFFFFF"
                                            }
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "KnitwearYarnColorsOption",
                                        "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "yarn_colors",
                                            "enum": [
                                              "yarn_colors"
                                            ]
                                          },
                                          "value": {
                                            "description": "Option value",
                                            "type": "array",
                                            "items": {
                                              "description": "Option value",
                                              "type": "string",
                                              "example": "#fdfafa",
                                              "enum": [
                                                "#090909",
                                                "#404040",
                                                "#563c33",
                                                "#d52213",
                                                "#6e5242",
                                                "#7f6a53",
                                                "#cd5e38",
                                                "#b57648",
                                                "#d1773b",
                                                "#d68785",
                                                "#c6b5a7",
                                                "#d6c6b4",
                                                "#dcd3cc",
                                                "#edd9d9",
                                                "#e2dfdc",
                                                "#fdfafa",
                                                "#999996",
                                                "#dda032",
                                                "#d1c6ae",
                                                "#eddea4",
                                                "#48542e",
                                                "#6e8c4b",
                                                "#c0c1bd",
                                                "#243f33",
                                                "#c5d1d0",
                                                "#175387",
                                                "#237d96",
                                                "#787979",
                                                "#343d55",
                                                "#4e59be",
                                                "#566e99",
                                                "#504372",
                                                "#4c1c29",
                                                "#f66274",
                                                "#eda6b4",
                                                "#ddabc8"
                                              ]
                                            }
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "position": {
                                  "type": "object",
                                  "title": "LayerPosition",
                                  "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                  "required": [
                                    "top",
                                    "left",
                                    "width",
                                    "height"
                                  ],
                                  "properties": {
                                    "width": {
                                      "description": "Layer width in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "minimum": 0.3,
                                      "example": 10
                                    },
                                    "height": {
                                      "description": "Layer height in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "minimum": 0.3,
                                      "example": 10
                                    },
                                    "top": {
                                      "description": "Layer top position in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "example": 0
                                    },
                                    "left": {
                                      "description": "Layer left position in inches",
                                      "type": "number",
                                      "format": "float",
                                      "multipleOf": 0.001,
                                      "example": 0
                                    }
                                  }
                                }
                              }
                            }
                          },
                          "placement_options": {
                            "description": "List of placement options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "UnlimitedColorOption",
                                  "description": "Specify if the design should use unlimited color technique",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "unlimited_color",
                                      "enum": [
                                        "unlimited_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether the design should use unlimited color technique.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "InsideLabelTypeOption",
                                  "description": "Specify the type of inside label",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_label_type",
                                      "enum": [
                                        "inside_label_type"
                                      ]
                                    },
                                    "value": {
                                      "description": "Specifies type of inside label design that should be used",
                                      "type": "string",
                                      "enum": [
                                        "native",
                                        "custom"
                                      ]
                                    }
                                  }
                                }
                              ]
                            }
                          },
                          "status": {
                            "type": "string",
                            "readOnly": true,
                            "enum": [
                              "ok",
                              "failed"
                            ],
                            "description": "Status of the placement design"
                          },
                          "status_explanation": {
                            "type": "string",
                            "readOnly": true,
                            "example": "Product with ID: 656 cannot have disjointed design elements.",
                            "description": "Reason behind failed status"
                          }
                        }
                      }
                    },
                    "orientation": {
                      "description": "Orientation of the design. Applies only to the products that allow multiple orientations such as framed posters.",
                      "type": "string",
                      "readOnly": false,
                      "nullable": true,
                      "enum": [
                        "horizontal",
                        "vertical",
                        "any"
                      ],
                      "example": "horizontal"
                    },
                    "product_options": {
                      "description": "List of product options",
                      "type": "array",
                      "items": {
                        "oneOf": [
                          {
                            "type": "object",
                            "title": "InsidePocketOption",
                            "description": "Specify if inside pocket should be added to the product",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "inside_pocket",
                                "enum": [
                                  "inside_pocket"
                                ]
                              },
                              "value": {
                                "description": "Whether inside pocket should be added to the product.",
                                "type": "boolean",
                                "example": true
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "StitchColorOption",
                            "description": "Specified what color should be used for stitches",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "stitch_color",
                                "enum": [
                                  "stitch_color"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "example": "white"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "NotesOption",
                            "description": "Include additional notes for fulfillment for embroidery prints",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "notes",
                                "enum": [
                                  "notes"
                                ]
                              },
                              "value": {
                                "description": "Additional notes for fulfillment for embroidery prints.",
                                "type": "string",
                                "example": "Please make sure that top side of the print is within print area"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "LifelikeOption",
                            "description": "Specifies if generated mockup should use lifelike effect",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "lifelike",
                                "enum": [
                                  "lifelike"
                                ]
                              },
                              "value": {
                                "description": "Whether generated mockup should use lifelike effect.",
                                "type": "boolean",
                                "example": true
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "CustomBorderColorOption",
                            "description": "Used to specify the border color of a sticker",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "custom_border_color",
                                "enum": [
                                  "custom_border_color"
                                ]
                              },
                              "value": {
                                "description": "Color defined in hexadecimal format",
                                "type": "string",
                                "example": "#FF0000"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "KnitwearBaseColorOption",
                            "description": "Used to specify the base color on a knitwear product.",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "base_color",
                                "enum": [
                                  "base_color"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "example": "#fdfafa",
                                "enum": [
                                  "#090909",
                                  "#404040",
                                  "#563c33",
                                  "#d52213",
                                  "#6e5242",
                                  "#7f6a53",
                                  "#cd5e38",
                                  "#b57648",
                                  "#d1773b",
                                  "#d68785",
                                  "#c6b5a7",
                                  "#d6c6b4",
                                  "#dcd3cc",
                                  "#edd9d9",
                                  "#e2dfdc",
                                  "#fdfafa",
                                  "#999996",
                                  "#dda032",
                                  "#d1c6ae",
                                  "#eddea4",
                                  "#48542e",
                                  "#6e8c4b",
                                  "#c0c1bd",
                                  "#243f33",
                                  "#c5d1d0",
                                  "#175387",
                                  "#237d96",
                                  "#787979",
                                  "#343d55",
                                  "#4e59be",
                                  "#566e99",
                                  "#504372",
                                  "#4c1c29",
                                  "#f66274",
                                  "#eda6b4",
                                  "#ddabc8"
                                ]
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "KnitwearTrimColorOption",
                            "description": "Used to specify the color of the trim on a knitwear product.",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "trim_color",
                                "enum": [
                                  "trim_color"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "example": "#fdfafa",
                                "enum": [
                                  "#090909",
                                  "#404040",
                                  "#563c33",
                                  "#d52213",
                                  "#6e5242",
                                  "#7f6a53",
                                  "#cd5e38",
                                  "#b57648",
                                  "#d1773b",
                                  "#d68785",
                                  "#c6b5a7",
                                  "#d6c6b4",
                                  "#dcd3cc",
                                  "#edd9d9",
                                  "#e2dfdc",
                                  "#fdfafa",
                                  "#999996",
                                  "#dda032",
                                  "#d1c6ae",
                                  "#eddea4",
                                  "#48542e",
                                  "#6e8c4b",
                                  "#c0c1bd",
                                  "#243f33",
                                  "#c5d1d0",
                                  "#175387",
                                  "#237d96",
                                  "#787979",
                                  "#343d55",
                                  "#4e59be",
                                  "#566e99",
                                  "#504372",
                                  "#4c1c29",
                                  "#f66274",
                                  "#eda6b4",
                                  "#ddabc8"
                                ]
                              }
                            }
                          },
                          {
                            "type": "object",
                            "title": "KnitwearColorReductionMode",
                            "description": "Used to set the color reduction mode for a knitwear product",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "color_reduction_mode",
                                "enum": [
                                  "color_reduction_mode"
                                ]
                              },
                              "value": {
                                "description": "Option value",
                                "type": "string",
                                "enum": [
                                  "solid",
                                  "pixelated"
                                ],
                                "example": "pixelated"
                              }
                            }
                          }
                        ],
                        "discriminator": {
                          "propertyName": "name",
                          "mapping": {
                            "inside_pocket": "#/components/schemas/InsidePocketOption",
                            "stitch_color": "#/components/schemas/StitchColorOption",
                            "notes": "#/components/schemas/NotesOption",
                            "lifelike": "#/components/schemas/LifelikeOption",
                            "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                            "base_color": "#/components/schemas/KnitwearBaseColor",
                            "trim_color": "#/components/schemas/KnitwearTrimColor",
                            "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                          }
                        }
                      }
                    },
                    "_links": {
                      "type": "object",
                      "readOnly": true,
                      "description": "HATEOAS links",
                      "properties": {
                        "self": {
                          "allOf": [
                            {
                              "type": "object",
                              "title": "HateoasLink",
                              "readOnly": true,
                              "required": [
                                "href"
                              ],
                              "properties": {
                                "href": {
                                  "description": "The HREF of the linked resource.",
                                  "type": "string"
                                }
                              }
                            },
                            {
                              "description": "Link to a specific order item",
                              "format": "HateoasLink",
                              "example": {
                                "href": "/v2/orders/123/order-items/123"
                              }
                            }
                          ]
                        }
                      }
                    }
                  }
                }
              ]
            },
            {
              "allOf": [
                {
                  "type": "object",
                  "title": "Product Template Item",
                  "description": "Information about the Product Template.",
                  "required": [
                    "source",
                    "product_template_id",
                    "catalog_variant_id"
                  ],
                  "properties": {
                    "source": {
                      "description": "Product Template source.",
                      "type": "string",
                      "enum": [
                        "product_template"
                      ]
                    },
                    "product_template_id": {
                      "oneOf": [
                        {
                          "type": "integer"
                        },
                        {
                          "type": "string"
                        }
                      ],
                      "description": "ID of Product Template. Allows external ID.",
                      "example": "@12345"
                    },
                    "catalog_variant_id": {
                      "description": "ID of Catalog variant. Must correspond to Product Template.",
                      "type": "integer",
                      "example": 123
                    }
                  }
                },
                {
                  "type": "object",
                  "title": "Item",
                  "description": "Information about the Item",
                  "properties": {
                    "id": {
                      "description": "Item ID",
                      "type": "integer",
                      "example": 1234,
                      "readOnly": true
                    },
                    "external_id": {
                      "description": "Item user specified external ID",
                      "type": "string",
                      "nullable": true,
                      "example": "123_abc"
                    },
                    "quantity": {
                      "description": "Item quantity",
                      "type": "integer",
                      "example": 1
                    },
                    "retail_price": {
                      "description": "Item retail price",
                      "type": "string",
                      "example": "10.00"
                    },
                    "name": {
                      "description": "Item custom name",
                      "type": "string",
                      "example": "Custom name"
                    },
                    "_links": {
                      "type": "object",
                      "readOnly": true,
                      "description": "HATEOAS links",
                      "properties": {
                        "self": {
                          "allOf": [
                            {
                              "type": "object",
                              "title": "HateoasLink",
                              "readOnly": true,
                              "required": [
                                "href"
                              ],
                              "properties": {
                                "href": {
                                  "description": "The HREF of the linked resource.",
                                  "type": "string"
                                }
                              }
                            },
                            {
                              "description": "Link to a specific order item",
                              "format": "HateoasLink",
                              "example": {
                                "href": "/v2/orders/123/order-items/123"
                              }
                            }
                          ]
                        }
                      }
                    }
                  }
                }
              ]
            }
          ],
          "discriminator": {
            "propertyName": "source",
            "mapping": {
              "catalog": "#/components/schemas/CatalogItem",
              "product_template": "#/components/schemas/ProductTemplateItem"
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
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "oneOf": [
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "placements",
                            "catalog_variant_id",
                            "source"
                          ],
                          "properties": {
                            "source": {
                              "description": "Catalog source",
                              "type": "string",
                              "enum": [
                                "catalog"
                              ]
                            },
                            "catalog_variant_id": {
                              "description": "ID of catalog variant",
                              "type": "integer",
                              "example": 4011
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "id",
                            "external_id",
                            "quantity",
                            "retail_price",
                            "name",
                            "placements",
                            "product_options",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "placements": {
                              "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Placement",
                                "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                                "required": [
                                  "placement",
                                  "technique",
                                  "layers",
                                  "status",
                                  "status_explanation"
                                ],
                                "properties": {
                                  "placement": {
                                    "description": "Name of the placement",
                                    "type": "string",
                                    "example": "front"
                                  },
                                  "technique": {
                                    "description": "Placement's technique",
                                    "type": "string",
                                    "example": "dtg"
                                  },
                                  "print_area_type": {
                                    "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                    "type": "string",
                                    "writeOnly": true,
                                    "default": "simple",
                                    "enum": [
                                      "simple",
                                      "advanced"
                                    ]
                                  },
                                  "layers": {
                                    "description": "Information about placement's layers",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "Layer",
                                      "description": "Information about the Layer",
                                      "required": [
                                        "type",
                                        "url"
                                      ],
                                      "properties": {
                                        "type": {
                                          "description": "Type of layer (e.g. file, text)",
                                          "type": "string",
                                          "example": "file"
                                        },
                                        "url": {
                                          "description": "File image URL if layer type is file",
                                          "type": "string",
                                          "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                        },
                                        "layer_options": {
                                          "description": "List of layer options",
                                          "type": "array",
                                          "items": {
                                            "oneOf": [
                                              {
                                                "type": "object",
                                                "title": "3dPuffOption",
                                                "description": "Should thread use 3d puff technique",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "3d_puff",
                                                    "enum": [
                                                      "3d_puff"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Whether the 3d puff technique should be used for the layer.",
                                                    "type": "boolean",
                                                    "example": true
                                                  }
                                                }
                                              },
                                              {
                                                "type": "object",
                                                "title": "ThreadColorsOption",
                                                "description": "Specify thread colors for embroidery technique",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "thread_colors",
                                                    "enum": [
                                                      "thread_colors"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Thread colors for embroidery technique",
                                                    "type": "array",
                                                    "items": {
                                                      "type": "string",
                                                      "example": "#FFFFFF"
                                                    }
                                                  }
                                                }
                                              },
                                              {
                                                "type": "object",
                                                "title": "KnitwearYarnColorsOption",
                                                "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                                "required": [
                                                  "name",
                                                  "value"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "yarn_colors",
                                                    "enum": [
                                                      "yarn_colors"
                                                    ]
                                                  },
                                                  "value": {
                                                    "description": "Option value",
                                                    "type": "array",
                                                    "items": {
                                                      "description": "Option value",
                                                      "type": "string",
                                                      "example": "#fdfafa",
                                                      "enum": [
                                                        "#090909",
                                                        "#404040",
                                                        "#563c33",
                                                        "#d52213",
                                                        "#6e5242",
                                                        "#7f6a53",
                                                        "#cd5e38",
                                                        "#b57648",
                                                        "#d1773b",
                                                        "#d68785",
                                                        "#c6b5a7",
                                                        "#d6c6b4",
                                                        "#dcd3cc",
                                                        "#edd9d9",
                                                        "#e2dfdc",
                                                        "#fdfafa",
                                                        "#999996",
                                                        "#dda032",
                                                        "#d1c6ae",
                                                        "#eddea4",
                                                        "#48542e",
                                                        "#6e8c4b",
                                                        "#c0c1bd",
                                                        "#243f33",
                                                        "#c5d1d0",
                                                        "#175387",
                                                        "#237d96",
                                                        "#787979",
                                                        "#343d55",
                                                        "#4e59be",
                                                        "#566e99",
                                                        "#504372",
                                                        "#4c1c29",
                                                        "#f66274",
                                                        "#eda6b4",
                                                        "#ddabc8"
                                                      ]
                                                    }
                                                  }
                                                }
                                              }
                                            ]
                                          }
                                        },
                                        "position": {
                                          "type": "object",
                                          "title": "LayerPosition",
                                          "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                          "required": [
                                            "top",
                                            "left",
                                            "width",
                                            "height"
                                          ],
                                          "properties": {
                                            "width": {
                                              "description": "Layer width in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "minimum": 0.3,
                                              "example": 10
                                            },
                                            "height": {
                                              "description": "Layer height in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "minimum": 0.3,
                                              "example": 10
                                            },
                                            "top": {
                                              "description": "Layer top position in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "example": 0
                                            },
                                            "left": {
                                              "description": "Layer left position in inches",
                                              "type": "number",
                                              "format": "float",
                                              "multipleOf": 0.001,
                                              "example": 0
                                            }
                                          }
                                        }
                                      }
                                    }
                                  },
                                  "placement_options": {
                                    "description": "List of placement options",
                                    "type": "array",
                                    "items": {
                                      "oneOf": [
                                        {
                                          "type": "object",
                                          "title": "UnlimitedColorOption",
                                          "description": "Specify if the design should use unlimited color technique",
                                          "required": [
                                            "name",
                                            "value"
                                          ],
                                          "properties": {
                                            "name": {
                                              "description": "Option identifier",
                                              "type": "string",
                                              "example": "unlimited_color",
                                              "enum": [
                                                "unlimited_color"
                                              ]
                                            },
                                            "value": {
                                              "description": "Whether the design should use unlimited color technique.",
                                              "type": "boolean",
                                              "example": true
                                            }
                                          }
                                        },
                                        {
                                          "type": "object",
                                          "title": "InsideLabelTypeOption",
                                          "description": "Specify the type of inside label",
                                          "required": [
                                            "name",
                                            "value"
                                          ],
                                          "properties": {
                                            "name": {
                                              "description": "Option identifier",
                                              "type": "string",
                                              "example": "inside_label_type",
                                              "enum": [
                                                "inside_label_type"
                                              ]
                                            },
                                            "value": {
                                              "description": "Specifies type of inside label design that should be used",
                                              "type": "string",
                                              "enum": [
                                                "native",
                                                "custom"
                                              ]
                                            }
                                          }
                                        }
                                      ]
                                    }
                                  },
                                  "status": {
                                    "type": "string",
                                    "readOnly": true,
                                    "enum": [
                                      "ok",
                                      "failed"
                                    ],
                                    "description": "Status of the placement design"
                                  },
                                  "status_explanation": {
                                    "type": "string",
                                    "readOnly": true,
                                    "example": "Product with ID: 656 cannot have disjointed design elements.",
                                    "description": "Reason behind failed status"
                                  }
                                }
                              }
                            },
                            "product_options": {
                              "description": "List of product options",
                              "type": "array",
                              "items": {
                                "oneOf": [
                                  {
                                    "type": "object",
                                    "title": "InsidePocketOption",
                                    "description": "Specify if inside pocket should be added to the product",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "inside_pocket",
                                        "enum": [
                                          "inside_pocket"
                                        ]
                                      },
                                      "value": {
                                        "description": "Whether inside pocket should be added to the product.",
                                        "type": "boolean",
                                        "example": true
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "StitchColorOption",
                                    "description": "Specified what color should be used for stitches",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "stitch_color",
                                        "enum": [
                                          "stitch_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "white"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "NotesOption",
                                    "description": "Include additional notes for fulfillment for embroidery prints",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "notes",
                                        "enum": [
                                          "notes"
                                        ]
                                      },
                                      "value": {
                                        "description": "Additional notes for fulfillment for embroidery prints.",
                                        "type": "string",
                                        "example": "Please make sure that top side of the print is within print area"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "LifelikeOption",
                                    "description": "Specifies if generated mockup should use lifelike effect",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "lifelike",
                                        "enum": [
                                          "lifelike"
                                        ]
                                      },
                                      "value": {
                                        "description": "Whether generated mockup should use lifelike effect.",
                                        "type": "boolean",
                                        "example": true
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "CustomBorderColorOption",
                                    "description": "Used to specify the border color of a sticker",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "custom_border_color",
                                        "enum": [
                                          "custom_border_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Color defined in hexadecimal format",
                                        "type": "string",
                                        "example": "#FF0000"
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearBaseColorOption",
                                    "description": "Used to specify the base color on a knitwear product.",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "base_color",
                                        "enum": [
                                          "base_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "#fdfafa",
                                        "enum": [
                                          "#090909",
                                          "#404040",
                                          "#563c33",
                                          "#d52213",
                                          "#6e5242",
                                          "#7f6a53",
                                          "#cd5e38",
                                          "#b57648",
                                          "#d1773b",
                                          "#d68785",
                                          "#c6b5a7",
                                          "#d6c6b4",
                                          "#dcd3cc",
                                          "#edd9d9",
                                          "#e2dfdc",
                                          "#fdfafa",
                                          "#999996",
                                          "#dda032",
                                          "#d1c6ae",
                                          "#eddea4",
                                          "#48542e",
                                          "#6e8c4b",
                                          "#c0c1bd",
                                          "#243f33",
                                          "#c5d1d0",
                                          "#175387",
                                          "#237d96",
                                          "#787979",
                                          "#343d55",
                                          "#4e59be",
                                          "#566e99",
                                          "#504372",
                                          "#4c1c29",
                                          "#f66274",
                                          "#eda6b4",
                                          "#ddabc8"
                                        ]
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearTrimColorOption",
                                    "description": "Used to specify the color of the trim on a knitwear product.",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "trim_color",
                                        "enum": [
                                          "trim_color"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "example": "#fdfafa",
                                        "enum": [
                                          "#090909",
                                          "#404040",
                                          "#563c33",
                                          "#d52213",
                                          "#6e5242",
                                          "#7f6a53",
                                          "#cd5e38",
                                          "#b57648",
                                          "#d1773b",
                                          "#d68785",
                                          "#c6b5a7",
                                          "#d6c6b4",
                                          "#dcd3cc",
                                          "#edd9d9",
                                          "#e2dfdc",
                                          "#fdfafa",
                                          "#999996",
                                          "#dda032",
                                          "#d1c6ae",
                                          "#eddea4",
                                          "#48542e",
                                          "#6e8c4b",
                                          "#c0c1bd",
                                          "#243f33",
                                          "#c5d1d0",
                                          "#175387",
                                          "#237d96",
                                          "#787979",
                                          "#343d55",
                                          "#4e59be",
                                          "#566e99",
                                          "#504372",
                                          "#4c1c29",
                                          "#f66274",
                                          "#eda6b4",
                                          "#ddabc8"
                                        ]
                                      }
                                    }
                                  },
                                  {
                                    "type": "object",
                                    "title": "KnitwearColorReductionMode",
                                    "description": "Used to set the color reduction mode for a knitwear product",
                                    "required": [
                                      "name",
                                      "value"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "color_reduction_mode",
                                        "enum": [
                                          "color_reduction_mode"
                                        ]
                                      },
                                      "value": {
                                        "description": "Option value",
                                        "type": "string",
                                        "enum": [
                                          "solid",
                                          "pixelated"
                                        ],
                                        "example": "pixelated"
                                      }
                                    }
                                  }
                                ],
                                "discriminator": {
                                  "propertyName": "name",
                                  "mapping": {
                                    "inside_pocket": "#/components/schemas/InsidePocketOption",
                                    "stitch_color": "#/components/schemas/StitchColorOption",
                                    "notes": "#/components/schemas/NotesOption",
                                    "lifelike": "#/components/schemas/LifelikeOption",
                                    "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                    "base_color": "#/components/schemas/KnitwearBaseColor",
                                    "trim_color": "#/components/schemas/KnitwearTrimColor",
                                    "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                  }
                                }
                              }
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    },
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Warehouse Item",
                          "description": "Information about the Warehouse Item",
                          "required": [
                            "source",
                            "warehouse_product_variant_id"
                          ],
                          "properties": {
                            "source": {
                              "description": "Warehouse source",
                              "type": "string",
                              "enum": [
                                "warehouse"
                              ]
                            },
                            "warehouse_product_variant_id": {
                              "description": "ID of warehouse variant",
                              "type": "integer",
                              "example": 1234
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "required": [
                            "id",
                            "external_id",
                            "quantity",
                            "retail_price",
                            "name",
                            "_links"
                          ],
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "required": [
                                "self"
                              ],
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    },
                    {
                      "allOf": [
                        {
                          "type": "object",
                          "title": "Product Template Item",
                          "description": "Information about the Product Template.",
                          "required": [
                            "source",
                            "product_template_id",
                            "catalog_variant_id"
                          ],
                          "properties": {
                            "source": {
                              "description": "Product Template source.",
                              "type": "string",
                              "enum": [
                                "product_template"
                              ]
                            },
                            "product_template_id": {
                              "oneOf": [
                                {
                                  "type": "integer"
                                },
                                {
                                  "type": "string"
                                }
                              ],
                              "description": "ID of Product Template. Allows external ID.",
                              "example": "@12345"
                            },
                            "catalog_variant_id": {
                              "description": "ID of Catalog variant. Must correspond to Product Template.",
                              "type": "integer",
                              "example": 123
                            }
                          }
                        },
                        {
                          "type": "object",
                          "title": "Item",
                          "description": "Information about the Item",
                          "properties": {
                            "id": {
                              "description": "Item ID",
                              "type": "integer",
                              "example": 1234,
                              "readOnly": true
                            },
                            "external_id": {
                              "description": "Item user specified external ID",
                              "type": "string",
                              "nullable": true,
                              "example": "123_abc"
                            },
                            "quantity": {
                              "description": "Item quantity",
                              "type": "integer",
                              "example": 1
                            },
                            "retail_price": {
                              "description": "Item retail price",
                              "type": "string",
                              "example": "10.00"
                            },
                            "name": {
                              "description": "Item custom name",
                              "type": "string",
                              "example": "Custom name"
                            },
                            "_links": {
                              "type": "object",
                              "readOnly": true,
                              "description": "HATEOAS links",
                              "properties": {
                                "self": {
                                  "allOf": [
                                    {
                                      "type": "object",
                                      "title": "HateoasLink",
                                      "readOnly": true,
                                      "required": [
                                        "href"
                                      ],
                                      "properties": {
                                        "href": {
                                          "description": "The HREF of the linked resource.",
                                          "type": "string"
                                        }
                                      }
                                    },
                                    {
                                      "description": "Link to a specific order item",
                                      "format": "HateoasLink",
                                      "example": {
                                        "href": "/v2/orders/123/order-items/123"
                                      }
                                    }
                                  ]
                                }
                              }
                            }
                          }
                        }
                      ]
                    }
                  ],
                  "discriminator": {
                    "propertyName": "source",
                    "mapping": {
                      "catalog": "#/components/schemas/CatalogItemReadonly",
                      "warehouse": "#/components/schemas/WarehouseItemReadonly",
                      "product_template": "#/components/schemas/ProductTemplateItem"
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self",
                  "order"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=20"
                    },
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
                          }
                        }
                      }
                    ]
                  },
                  "next": {
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=30"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "previous": {
                    "description": "Link to the previous page (absent if the current page is the first one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "first": {
                    "description": "Link to the first page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "last": {
                    "description": "Link to the last page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/orders/123/order-items?limit=10&offset=90"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "order": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to the order",
                        "format": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/orders/123"
                        }
                      }
                    ]
                  }
                }
              }
            }
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
    "403": {
      "description": "Forbidden",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `403`",
                "example": 403
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "This endpoint requires Oauth authentication!."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "integer",
                    "example": 403
                  },
                  "message": {
                    "type": "string",
                    "example": "This endpoint requires Oauth authentication!."
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
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### DELETE /v2/orders/{order_id}/order-items/{order_item_id}
**Summary:** Delete Order Item

Remove a single item from the order.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteItemById",
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
      "name": "order_item_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Item ID (integer) or Item External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "responses": {
    "204": {
      "description": "No Content"
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
    "403": {
      "description": "Forbidden",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `403`",
                "example": 403
              },
              "result": {
                "type": "string",
                "description": "Actual error message",
                "example": "This endpoint requires Oauth authentication!."
              },
              "error": {
                "type": "object",
                "properties": {
                  "reason": {
                    "type": "integer",
                    "example": 403
                  },
                  "message": {
                    "type": "string",
                    "example": "This endpoint requires Oauth authentication!."
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
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### GET /v2/orders/{order_id}/shipments
**Summary:** Retrieve a list of shipments

Shipments contain information about how and when your orders items will be delivered and fulfilled.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getShipments",
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
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer",
        "minimum": 1,
        "maximum": 100,
        "default": 20
      },
      "required": false,
      "description": "The number of results to return per page."
    },
    {
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer",
        "minimum": 0,
        "default": 0
      },
      "required": false,
      "examples": {
        "startAt100": {
          "summary": "Return results after the initial 100",
          "value": 100
        }
      },
      "description": "The number of results to not include in the response starting from the beginning of the list.\n\nThis can be used to return results after the initial 100. For example, sending offset 100\n"
    }
  ],
  "responses": {
    "200": {
      "description": "OK",
      "content": {
        "application/json": {
          "schema": {
            "required": [
              "data",
              "_links",
              "paging"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "Shipment",
                  "properties": {
                    "id": {
                      "type": "integer",
                      "example": 1
                    },
                    "order_id": {
                      "type": "integer",
                      "example": 2
                    },
                    "order_external_id": {
                      "type": "string",
                      "nullable": true,
                      "example": "my_custom_id_1234"
                    },
                    "carrier": {
                      "type": "string",
                      "nullable": true,
                      "example": "USPS",
                      "description": "The carrier that will fulfill the shipment."
                    },
                    "service": {
                      "type": "string",
                      "nullable": true,
                      "example": "USPS Priority Mail",
                      "description": "The service being used to fulfill the shipment."
                    },
                    "shipment_status": {
                      "type": "string",
                      "enum": [
                        "pending",
                        "onhold",
                        "canceled",
                        "packaged",
                        "shipped",
                        "returned",
                        "outstock"
                      ],
                      "example": "canceled"
                    },
                    "shipped_at": {
                      "type": "string",
                      "nullable": true,
                      "format": "date-time",
                      "example": "2023-06-15T16:35:35Z"
                    },
                    "delivery_status": {
                      "type": "string",
                      "enum": [
                        "unknown",
                        "delivered",
                        "pre_transit",
                        "in_transit",
                        "out_for_delivery",
                        "available_for_pickup",
                        "return_to_sender",
                        "failure",
                        "canceled"
                      ]
                    },
                    "delivered_at": {
                      "type": "string",
                      "nullable": true,
                      "format": "date-time",
                      "example": "2023-06-15T16:35:35Z"
                    },
                    "departure_address": {
                      "type": "object",
                      "properties": {
                        "country_name": {
                          "type": "string",
                          "example": "United States"
                        },
                        "country_code": {
                          "type": "string",
                          "example": "US"
                        },
                        "state_code": {
                          "type": "string",
                          "example": "CA"
                        }
                      }
                    },
                    "is_reshipment": {
                      "type": "boolean",
                      "description": "If there is an issue with items in a shipment, a reshipment might be necessary. This property will be false if it is the original shipment and true if it is a reshipment"
                    },
                    "tracking_number": {
                      "type": "string",
                      "nullable": true,
                      "example": "39925631",
                      "description": "The tracking code associated with the shipment."
                    },
                    "tracking_url": {
                      "type": "string",
                      "example": "\u200bhttps://myorders.com/tracking/39925631"
                    },
                    "tracking_events": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "Tracking Event",
                        "properties": {
                          "triggered_at": {
                            "type": "string",
                            "format": "date-time",
                            "example": "2023-06-15T19:15:05Z"
                          },
                          "description": {
                            "type": "string",
                            "example": "Arrived At Destination"
                          }
                        }
                      }
                    },
                    "estimated_delivery": {
                      "type": "object",
                      "nullable": true,
                      "properties": {
                        "from_date": {
                          "type": "string",
                          "format": "date",
                          "example": "2023-06-15",
                          "description": "Earliest estimated date the shipment will arrive"
                        },
                        "to_date": {
                          "type": "string",
                          "format": "date",
                          "example": "2023-06-15",
                          "description": "Latest estimated date the shipment will arrive"
                        },
                        "calculated_at": {
                          "type": "string",
                          "format": "date-time",
                          "example": "2023-06-15T16:35:35Z"
                        }
                      }
                    },
                    "shipment_items": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "ShipmentItem",
                        "properties": {
                          "id": {
                            "type": "integer",
                            "example": 10
                          },
                          "order_item_id": {
                            "type": "integer",
                            "example": 20
                          },
                          "order_item_external_id": {
                            "type": "string",
                            "nullable": true,
                            "example": "item-external-id"
                          },
                          "order_item_name": {
                            "type": "string",
                            "nullable": true,
                            "example": "Item name"
                          },
                          "quantity": {
                            "type": "integer",
                            "example": 1
                          },
                          "_links": {
                            "type": "object",
                            "properties": {
                              "order_item": {
                                "type": "object",
                                "properties": {
                                  "href": {
                                    "type": "string",
                                    "example": "\u200bhttps://api.printful.com/v2/orders/2/order-items/20"
                                  }
                                }
                              }
                            }
                          }
                        }
                      }
                    },
                    "_links": {
                      "type": "object",
                      "properties": {
                        "self": {
                          "type": "object",
                          "properties": {
                            "href": {
                              "type": "string",
                              "example": "\u200bhttps://api.printful.com/v2/shipments/1"
                            }
                          }
                        },
                        "order": {
                          "type": "object",
                          "properties": {
                            "href": {
                              "type": "string",
                              "example": "\u200bhttps://api.printful.com/v2/orders/2"
                            }
                          }
                        }
                      }
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/orders/123/shipments?limit=20&offset=0"
                      }
                    }
                  },
                  "next": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/orders/123/shipments?limit=20&offset=20"
                      }
                    }
                  },
                  "previous": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/orders/123/shipments?limit=20&offset=0"
                      }
                    }
                  },
                  "first": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/orders/123/shipments?limit=20&offset=0"
                      }
                    }
                  },
                  "last": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/orders/123/shipments?limit=20&offset=20"
                      }
                    }
                  }
                }
              },
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### GET /v2/orders/{order_id}/invoices
**Summary:** Retrieve an invoice

Returns the invoice for an order as a base64 encoded document. Decoding the base64 content can be different depending on the client, for most browsers this format will alow you to view and display the invoice `data:application/pdf;base64,{the_base_64_content_string}`.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getInvoiceByOrderId",
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
      "name": "order_id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Order ID (integer) or Order External ID (string prepended with \"@\" symbol)"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "type": "object",
                "required": [
                  "media_type",
                  "content",
                  "_links"
                ],
                "properties": {
                  "media_type": {
                    "type": "string",
                    "example": "application/pdf",
                    "description": "This property is used to describe the media type of the encoded base64 value. It is necessary to use this property when decoding the content property."
                  },
                  "content": {
                    "type": "string",
                    "example": "0xLjQKJeLjz9MKMyAwIG9iago8PC9UeXBlIC9QYWdlCi9QYXJlbnQgMSAwIFIKL01lZG",
                    "description": "The base64 encoded document containing the invoice."
                  },
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "required": [
                      "self",
                      "order"
                    ],
                    "properties": {
                      "self": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "HateoasLink",
                            "readOnly": true,
                            "required": [
                              "href"
                            ],
                            "properties": {
                              "href": {
                                "description": "The HREF of the linked resource.",
                                "type": "string"
                              }
                            }
                          },
                          {
                            "description": "Link to an order invoice",
                            "format": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/orders/123/invoices"
                            }
                          }
                        ]
                      },
                      "order": {
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          },
                          {
                            "description": "Link to a specific order",
                            "format": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/orders/123"
                            }
                          }
                        ]
                      }
                    }
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### GET /v2/order-estimation-tasks
**Summary:** Retrieve an order estimation task

Retrieve an order cost estimation task from a specific store.
Estimation results are only available for one hour after cost estimation task is done.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getOrderEstimationTask",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "id",
      "schema": {
        "type": "string"
      },
      "required": true,
      "description": "Order estimation task ID.",
      "example": null
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "allOf": [
                  {
                    "type": "object",
                    "title": "OrderEstimationTaskSummary",
                    "description": "Order estimation task summary",
                    "readOnly": true,
                    "required": [
                      "id",
                      "status",
                      "costs",
                      "retail_costs",
                      "failure_reasons"
                    ],
                    "properties": {
                      "id": {
                        "description": "Task ID",
                        "type": "string",
                        "example": "fc959efb-b3a0-4c12-9cc6-f54d3158291d"
                      },
                      "status": {
                        "description": "Task status",
                        "type": "string",
                        "enum": [
                          "pending",
                          "failed",
                          "completed"
                        ]
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "Costs",
                            "description": "The Order costs (Printful prices)",
                            "readOnly": true,
                            "required": [
                              "calculation_status",
                              "currency",
                              "subtotal",
                              "discount",
                              "shipping",
                              "digitization",
                              "additional_fee",
                              "fulfillment_fee",
                              "retail_delivery_fee",
                              "vat",
                              "tax",
                              "total"
                            ],
                            "properties": {
                              "calculation_status": {
                                "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                                "type": "string",
                                "enum": [
                                  "done",
                                  "calculating",
                                  "failed"
                                ],
                                "example": "done"
                              },
                              "currency": {
                                "description": "The code of the currency in which the costs are returned.",
                                "type": "string",
                                "nullable": true,
                                "example": "USD"
                              },
                              "subtotal": {
                                "description": "Total cost of all items.",
                                "type": "string",
                                "nullable": true,
                                "example": "14.95"
                              },
                              "discount": {
                                "description": "Discount sum.",
                                "type": "string",
                                "nullable": true,
                                "example": "1.79"
                              },
                              "shipping": {
                                "description": "Shipping costs.",
                                "type": "string",
                                "nullable": true,
                                "example": "4.79"
                              },
                              "digitization": {
                                "description": "Digitization costs.",
                                "type": "string",
                                "nullable": true,
                                "example": "3.95"
                              },
                              "additional_fee": {
                                "description": "Additional fee for custom product.",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "description": "Custom product fulfillment fee.",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "description": "Retail delivery fee.",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "vat": {
                                "description": "Sum of vat (not included in the item price).",
                                "type": "string",
                                "nullable": true,
                                "example": "4.60"
                              },
                              "tax": {
                                "description": "Sum of taxes (not included in the item price).",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "total": {
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                                "type": "string",
                                "nullable": true,
                                "example": "26.50"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "nullable": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "RetailCosts",
                            "description": "The Order's retail costs",
                            "readOnly": true,
                            "required": [
                              "calculation_status",
                              "currency",
                              "subtotal",
                              "discount",
                              "shipping",
                              "vat",
                              "tax",
                              "total"
                            ],
                            "properties": {
                              "calculation_status": {
                                "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                                "type": "string",
                                "enum": [
                                  "done",
                                  "calculating",
                                  "failed"
                                ],
                                "example": "done"
                              },
                              "currency": {
                                "description": "The code of the currency in which the retail costs are returned.",
                                "type": "string",
                                "example": "EUR"
                              },
                              "subtotal": {
                                "description": "Total cost of all items.",
                                "type": "string",
                                "nullable": true,
                                "example": "26.55"
                              },
                              "discount": {
                                "description": "Discount sum.",
                                "type": "string",
                                "example": "0.00"
                              },
                              "shipping": {
                                "description": "Shipping costs.",
                                "type": "string",
                                "example": "4.79"
                              },
                              "vat": {
                                "description": "Sum of VAT (not included in the item price).",
                                "type": "string",
                                "example": "0.00"
                              },
                              "tax": {
                                "description": "Sum of taxes (not included in the item price).",
                                "type": "string",
                                "example": "0.00"
                              },
                              "total": {
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                                "type": "string",
                                "nullable": true,
                                "example": "31.34"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "nullable": true
                          }
                        ]
                      },
                      "failure_reasons": {
                        "type": "array",
                        "description": "Reasons why calculation failed.",
                        "items": {
                          "type": "string",
                          "example": "Recipient: Country cannot be blank."
                        },
                        "example": []
                      }
                    }
                  },
                  {
                    "type": "object",
                    "properties": {
                      "status": {
                        "example": "completed"
                      }
                    }
                  }
                ]
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
            "required": [
              "data",
              "error"
            ],
            "properties": {
              "data": {
                "type": "string",
                "description": "Actual error message",
                "example": "Couldn't find order estimation with this UUID."
              },
              "error": {
                "type": "object",
                "required": [
                  "reason",
                  "message"
                ],
                "properties": {
                  "reason": {
                    "type": "string",
                    "example": "NotFound"
                  },
                  "message": {
                    "type": "string",
                    "example": "Couldn't find order estimation with this UUID."
                  }
                }
              }
            }
          }
        }
      }
    },
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---

### POST /v2/order-estimation-tasks
**Summary:** Create a new order estimation task

Use this endpoint to estimate orders with items.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createOrderEstimationTask",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "required": [
            "recipient",
            "order_items"
          ],
          "properties": {
            "recipient": {
              "description": "The recipient data.",
              "title": "Address",
              "allOf": [
                {
                  "type": "object",
                  "title": "Estimation address",
                  "description": "Information about the address for estimations.",
                  "required": [
                    "country_code",
                    "zip"
                  ],
                  "properties": {
                    "state_code": {
                      "description": "State code.",
                      "type": "string",
                      "example": "CA"
                    },
                    "country_code": {
                      "description": "Country code",
                      "type": "string",
                      "example": "US"
                    },
                    "zip": {
                      "description": "ZIP/Postal code",
                      "type": "string",
                      "example": "91311"
                    }
                  }
                }
              ]
            },
            "order_items": {
              "type": "array",
              "description": "Array of order items",
              "items": {
                "oneOf": [
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "required": [
                          "placements",
                          "catalog_variant_id",
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Catalog source",
                            "type": "string",
                            "enum": [
                              "catalog"
                            ]
                          },
                          "catalog_variant_id": {
                            "description": "ID of catalog variant",
                            "type": "integer",
                            "example": 4011
                          }
                        }
                      },
                      {
                        "type": "object",
                        "title": "Item",
                        "description": "Information about the Item",
                        "properties": {
                          "id": {
                            "description": "Item ID",
                            "type": "integer",
                            "example": 1234,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Item user specified external ID",
                            "type": "string",
                            "nullable": true,
                            "example": "123_abc"
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "example": 1
                          },
                          "retail_price": {
                            "description": "Item retail price",
                            "type": "string",
                            "example": "10.00"
                          },
                          "name": {
                            "description": "Item custom name",
                            "type": "string",
                            "example": "Custom name"
                          },
                          "placements": {
                            "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Placement",
                              "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                              "required": [
                                "placement",
                                "technique",
                                "layers",
                                "status",
                                "status_explanation"
                              ],
                              "properties": {
                                "placement": {
                                  "description": "Name of the placement",
                                  "type": "string",
                                  "example": "front"
                                },
                                "technique": {
                                  "description": "Placement's technique",
                                  "type": "string",
                                  "example": "dtg"
                                },
                                "print_area_type": {
                                  "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                  "type": "string",
                                  "writeOnly": true,
                                  "default": "simple",
                                  "enum": [
                                    "simple",
                                    "advanced"
                                  ]
                                },
                                "layers": {
                                  "description": "Information about placement's layers",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "Layer",
                                    "description": "Information about the Layer",
                                    "required": [
                                      "type",
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "description": "Type of layer (e.g. file, text)",
                                        "type": "string",
                                        "example": "file"
                                      },
                                      "url": {
                                        "description": "File image URL if layer type is file",
                                        "type": "string",
                                        "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                      },
                                      "layer_options": {
                                        "description": "List of layer options",
                                        "type": "array",
                                        "items": {
                                          "oneOf": [
                                            {
                                              "type": "object",
                                              "title": "3dPuffOption",
                                              "description": "Should thread use 3d puff technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "3d_puff",
                                                  "enum": [
                                                    "3d_puff"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Whether the 3d puff technique should be used for the layer.",
                                                  "type": "boolean",
                                                  "example": true
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "ThreadColorsOption",
                                              "description": "Specify thread colors for embroidery technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "thread_colors",
                                                  "enum": [
                                                    "thread_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Thread colors for embroidery technique",
                                                  "type": "array",
                                                  "items": {
                                                    "type": "string",
                                                    "example": "#FFFFFF"
                                                  }
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "KnitwearYarnColorsOption",
                                              "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "yarn_colors",
                                                  "enum": [
                                                    "yarn_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Option value",
                                                  "type": "array",
                                                  "items": {
                                                    "description": "Option value",
                                                    "type": "string",
                                                    "example": "#fdfafa",
                                                    "enum": [
                                                      "#090909",
                                                      "#404040",
                                                      "#563c33",
                                                      "#d52213",
                                                      "#6e5242",
                                                      "#7f6a53",
                                                      "#cd5e38",
                                                      "#b57648",
                                                      "#d1773b",
                                                      "#d68785",
                                                      "#c6b5a7",
                                                      "#d6c6b4",
                                                      "#dcd3cc",
                                                      "#edd9d9",
                                                      "#e2dfdc",
                                                      "#fdfafa",
                                                      "#999996",
                                                      "#dda032",
                                                      "#d1c6ae",
                                                      "#eddea4",
                                                      "#48542e",
                                                      "#6e8c4b",
                                                      "#c0c1bd",
                                                      "#243f33",
                                                      "#c5d1d0",
                                                      "#175387",
                                                      "#237d96",
                                                      "#787979",
                                                      "#343d55",
                                                      "#4e59be",
                                                      "#566e99",
                                                      "#504372",
                                                      "#4c1c29",
                                                      "#f66274",
                                                      "#eda6b4",
                                                      "#ddabc8"
                                                    ]
                                                  }
                                                }
                                              }
                                            }
                                          ]
                                        }
                                      },
                                      "position": {
                                        "type": "object",
                                        "title": "LayerPosition",
                                        "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                        "required": [
                                          "top",
                                          "left",
                                          "width",
                                          "height"
                                        ],
                                        "properties": {
                                          "width": {
                                            "description": "Layer width in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "height": {
                                            "description": "Layer height in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "top": {
                                            "description": "Layer top position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          },
                                          "left": {
                                            "description": "Layer left position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          }
                                        }
                                      }
                                    }
                                  }
                                },
                                "placement_options": {
                                  "description": "List of placement options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "UnlimitedColorOption",
                                        "description": "Specify if the design should use unlimited color technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "unlimited_color",
                                            "enum": [
                                              "unlimited_color"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the design should use unlimited color technique.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "InsideLabelTypeOption",
                                        "description": "Specify the type of inside label",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "inside_label_type",
                                            "enum": [
                                              "inside_label_type"
                                            ]
                                          },
                                          "value": {
                                            "description": "Specifies type of inside label design that should be used",
                                            "type": "string",
                                            "enum": [
                                              "native",
                                              "custom"
                                            ]
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "status": {
                                  "type": "string",
                                  "readOnly": true,
                                  "enum": [
                                    "ok",
                                    "failed"
                                  ],
                                  "description": "Status of the placement design"
                                },
                                "status_explanation": {
                                  "type": "string",
                                  "readOnly": true,
                                  "example": "Product with ID: 656 cannot have disjointed design elements.",
                                  "description": "Reason behind failed status"
                                }
                              }
                            }
                          },
                          "orientation": {
                            "description": "Orientation of the design. Applies only to the products that allow multiple orientations such as framed posters.",
                            "type": "string",
                            "readOnly": false,
                            "nullable": true,
                            "enum": [
                              "horizontal",
                              "vertical",
                              "any"
                            ],
                            "example": "horizontal"
                          },
                          "product_options": {
                            "description": "List of product options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "InsidePocketOption",
                                  "description": "Specify if inside pocket should be added to the product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_pocket",
                                      "enum": [
                                        "inside_pocket"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether inside pocket should be added to the product.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "StitchColorOption",
                                  "description": "Specified what color should be used for stitches",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "stitch_color",
                                      "enum": [
                                        "stitch_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "white"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "NotesOption",
                                  "description": "Include additional notes for fulfillment for embroidery prints",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "notes",
                                      "enum": [
                                        "notes"
                                      ]
                                    },
                                    "value": {
                                      "description": "Additional notes for fulfillment for embroidery prints.",
                                      "type": "string",
                                      "example": "Please make sure that top side of the print is within print area"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "LifelikeOption",
                                  "description": "Specifies if generated mockup should use lifelike effect",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "lifelike",
                                      "enum": [
                                        "lifelike"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether generated mockup should use lifelike effect.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "CustomBorderColorOption",
                                  "description": "Used to specify the border color of a sticker",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "custom_border_color",
                                      "enum": [
                                        "custom_border_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Color defined in hexadecimal format",
                                      "type": "string",
                                      "example": "#FF0000"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearBaseColorOption",
                                  "description": "Used to specify the base color on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "base_color",
                                      "enum": [
                                        "base_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearTrimColorOption",
                                  "description": "Used to specify the color of the trim on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "trim_color",
                                      "enum": [
                                        "trim_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearColorReductionMode",
                                  "description": "Used to set the color reduction mode for a knitwear product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "color_reduction_mode",
                                      "enum": [
                                        "color_reduction_mode"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "enum": [
                                        "solid",
                                        "pixelated"
                                      ],
                                      "example": "pixelated"
                                    }
                                  }
                                }
                              ],
                              "discriminator": {
                                "propertyName": "name",
                                "mapping": {
                                  "inside_pocket": "#/components/schemas/InsidePocketOption",
                                  "stitch_color": "#/components/schemas/StitchColorOption",
                                  "notes": "#/components/schemas/NotesOption",
                                  "lifelike": "#/components/schemas/LifelikeOption",
                                  "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                  "base_color": "#/components/schemas/KnitwearBaseColor",
                                  "trim_color": "#/components/schemas/KnitwearTrimColor",
                                  "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                }
                              }
                            }
                          },
                          "_links": {
                            "type": "object",
                            "readOnly": true,
                            "description": "HATEOAS links",
                            "properties": {
                              "self": {
                                "allOf": [
                                  {
                                    "type": "object",
                                    "title": "HateoasLink",
                                    "readOnly": true,
                                    "required": [
                                      "href"
                                    ],
                                    "properties": {
                                      "href": {
                                        "description": "The HREF of the linked resource.",
                                        "type": "string"
                                      }
                                    }
                                  },
                                  {
                                    "description": "Link to a specific order item",
                                    "format": "HateoasLink",
                                    "example": {
                                      "href": "/v2/orders/123/order-items/123"
                                    }
                                  }
                                ]
                              }
                            }
                          }
                        }
                      }
                    ]
                  }
                ],
                "discriminator": {
                  "propertyName": "source",
                  "mapping": {
                    "catalog": "#/components/schemas/CatalogItem",
                    "product_template": "#/components/schemas/ProductTemplateItem"
                  }
                }
              }
            },
            "retail_costs": {
              "description": "Retail costs",
              "type": "object",
              "properties": {
                "currency": {
                  "description": "The code of the currency in which the retail costs are returned.",
                  "type": "string",
                  "example": "EUR"
                },
                "discount": {
                  "description": "Discount sum.",
                  "type": "string",
                  "example": "123.40"
                },
                "shipping": {
                  "description": "Shipping costs.",
                  "type": "string",
                  "example": "123.40"
                },
                "tax": {
                  "description": "Sum of taxes (not included in the item price).",
                  "type": "string",
                  "example": "123.40"
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "allOf": [
                  {
                    "type": "object",
                    "title": "OrderEstimationTaskSummary",
                    "description": "Order estimation task summary",
                    "readOnly": true,
                    "required": [
                      "id",
                      "status",
                      "costs",
                      "retail_costs",
                      "failure_reasons"
                    ],
                    "properties": {
                      "id": {
                        "description": "Task ID",
                        "type": "string",
                        "example": "fc959efb-b3a0-4c12-9cc6-f54d3158291d"
                      },
                      "status": {
                        "description": "Task status",
                        "type": "string",
                        "enum": [
                          "pending",
                          "failed",
                          "completed"
                        ]
                      },
                      "costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "Costs",
                            "description": "The Order costs (Printful prices)",
                            "readOnly": true,
                            "required": [
                              "calculation_status",
                              "currency",
                              "subtotal",
                              "discount",
                              "shipping",
                              "digitization",
                              "additional_fee",
                              "fulfillment_fee",
                              "retail_delivery_fee",
                              "vat",
                              "tax",
                              "total"
                            ],
                            "properties": {
                              "calculation_status": {
                                "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                                "type": "string",
                                "enum": [
                                  "done",
                                  "calculating",
                                  "failed"
                                ],
                                "example": "done"
                              },
                              "currency": {
                                "description": "The code of the currency in which the costs are returned.",
                                "type": "string",
                                "nullable": true,
                                "example": "USD"
                              },
                              "subtotal": {
                                "description": "Total cost of all items.",
                                "type": "string",
                                "nullable": true,
                                "example": "14.95"
                              },
                              "discount": {
                                "description": "Discount sum.",
                                "type": "string",
                                "nullable": true,
                                "example": "1.79"
                              },
                              "shipping": {
                                "description": "Shipping costs.",
                                "type": "string",
                                "nullable": true,
                                "example": "4.79"
                              },
                              "digitization": {
                                "description": "Digitization costs.",
                                "type": "string",
                                "nullable": true,
                                "example": "3.95"
                              },
                              "additional_fee": {
                                "description": "Additional fee for custom product.",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "fulfillment_fee": {
                                "description": "Custom product fulfillment fee.",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "retail_delivery_fee": {
                                "description": "Retail delivery fee.",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "vat": {
                                "description": "Sum of vat (not included in the item price).",
                                "type": "string",
                                "nullable": true,
                                "example": "4.60"
                              },
                              "tax": {
                                "description": "Sum of taxes (not included in the item price).",
                                "type": "string",
                                "nullable": true,
                                "example": "0.00"
                              },
                              "total": {
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                                "type": "string",
                                "nullable": true,
                                "example": "26.50"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "nullable": true
                          }
                        ]
                      },
                      "retail_costs": {
                        "allOf": [
                          {
                            "type": "object",
                            "title": "RetailCosts",
                            "description": "The Order's retail costs",
                            "readOnly": true,
                            "required": [
                              "calculation_status",
                              "currency",
                              "subtotal",
                              "discount",
                              "shipping",
                              "vat",
                              "tax",
                              "total"
                            ],
                            "properties": {
                              "calculation_status": {
                                "description": "If the costs are being calculated or recalculated, this will have the status `calculating`. Once finished the status will be `done`.",
                                "type": "string",
                                "enum": [
                                  "done",
                                  "calculating",
                                  "failed"
                                ],
                                "example": "done"
                              },
                              "currency": {
                                "description": "The code of the currency in which the retail costs are returned.",
                                "type": "string",
                                "example": "EUR"
                              },
                              "subtotal": {
                                "description": "Total cost of all items.",
                                "type": "string",
                                "nullable": true,
                                "example": "26.55"
                              },
                              "discount": {
                                "description": "Discount sum.",
                                "type": "string",
                                "example": "0.00"
                              },
                              "shipping": {
                                "description": "Shipping costs.",
                                "type": "string",
                                "example": "4.79"
                              },
                              "vat": {
                                "description": "Sum of VAT (not included in the item price).",
                                "type": "string",
                                "example": "0.00"
                              },
                              "tax": {
                                "description": "Sum of taxes (not included in the item price).",
                                "type": "string",
                                "example": "0.00"
                              },
                              "total": {
                                "description": "Grand Total (subtotal-discount+tax+vat+shipping).",
                                "type": "string",
                                "nullable": true,
                                "example": "31.34"
                              }
                            }
                          },
                          {
                            "type": "object",
                            "nullable": true
                          }
                        ]
                      },
                      "failure_reasons": {
                        "type": "array",
                        "description": "Reasons why calculation failed.",
                        "items": {
                          "type": "string",
                          "example": "Recipient: Country cannot be blank."
                        },
                        "example": []
                      }
                    }
                  },
                  {
                    "type": "object",
                    "properties": {
                      "costs": {
                        "example": null
                      },
                      "retail_costs": {
                        "example": null
                      }
                    }
                  }
                ]
              }
            }
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
    "5XX": {
      "description": "Server Error",
      "content": {
        "application/json": {
          "schema": {
            "allOf": [
              {
                "type": "object",
                "properties": {
                  "type": {
                    "type": "string",
                    "format": "URL",
                    "description": "A URL that can be followed to get to our [documentation](#section/Errors) for the problem type."
                  },
                  "status": {
                    "type": "integer",
                    "description": "The HTTP status code."
                  },
                  "title": {
                    "type": "string",
                    "description": "A human-readable summary of the problem type."
                  },
                  "details": {
                    "type": "string",
                    "description": "A human-readable explanation specific to the occurrence of the problem."
                  },
                  "instance": {
                    "type": "string",
                    "format": "URI",
                    "description": "Optional. A URI that uniquely identifies the specific occurence of the problem"
                  }
                }
              },
              {
                "type": "object",
                "properties": {
                  "status": {
                    "type": "integer",
                    "minimum": 500,
                    "maximum": 599
                  }
                }
              }
            ]
          },
          "examples": {
            "InternalServerError": {
              "summary": "Internal Server Error",
              "value": {
                "type": "https://developers.printful.com/docs/v2-beta/#errors/internal-server-error",
                "status": 500,
                "title": "Internal Server Error",
                "details": "An error occurred and we have been notified. You can contact us for more information.",
                "instance": "tag:api.printful.com,2023-10-04:errors/86657"
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

---


# Examples

# Catalog API examples

## Using size guides

In this example, we'll fetch the size guides for the
"Women's Basic Organic T-Shirt | SOL'S 02077" product (ID: 561).

We use the default `en_US` locale and don't provide the `unit` parameter, so the measurement values will be returned in
inches.

URL: https://api.printful.com/v2/catalog-products/561/sizes

<details>
    <summary>Whole response body</summary>

```javascript
{
  "data": {
    "product_id": 561,
    "available_sizes": [
      "S",
      "M",
      "L",
      "XL",
      "2XL"
    ],
    "size_tables": [
      {
        "type": "measure_yourself",
        "unit": "inches",
        "description": "<p>Measurements are provided by suppliers.<br /><br />US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\n<p>Product measurements may vary by up to 2\" (5 cm).&nbsp;</p>",
        "image_url": "https://s3-printful.stage.printful.dev/upload/measure-yourself/6a/6a4fe322f592f2b91d5a735d7ff8d1c0_t?v=1652962720",
        "image_description": "<h6><strong>A Length</strong></h6>\n<p dir=\"ltr\"><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">o the bottom of the shirt.</span></p>\n<h6>B Chest</h6>\n<p dir=\"ltr\">Measure yourself around the fullest part of your chest. Keep the tape measure horizontal.</p>",
        "measurements": [
          {
            "type_label": "Length",
            "values": [
              {
                "size": "S",
                "value": "25.2"
              },
              {
                "size": "M",
                "value": "26"
              },
              {
                "size": "L",
                "value": "26.7"
              },
              {
                "size": "XL",
                "value": "27.6"
              },
              {
                "size": "2XL",
                "value": "28.3"
              }
            ]
          },
          {
            "type_label": "Chest",
            "values": [
              {
                "size": "S",
                "min_value": "34",
                "max_value": "37"
              },
              {
                "size": "M",
                "min_value": "36",
                "max_value": "39"
              },
              {
                "size": "L",
                "min_value": "39",
                "max_value": "42"
              },
              {
                "size": "XL",
                "min_value": "41",
                "max_value": "44"
              },
              {
                "size": "2XL",
                "min_value": "43",
                "max_value": "46"
              }
            ]
          }
        ]
      },
      {
        "type": "product_measure",
        "unit": "inches",
        "description": "<p dir=\"ltr\">Measurements are provided by our suppliers. Product measurements may vary by up to 2\" (5 cm).</p>\n<p dir=\"ltr\">US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\n<p dir=\"ltr\">Pro tip! Measure one of your products at home and compare with the measurements you see in this guide.</p>",
        "image_url": "https://s3-printful.stage.printful.dev/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
        "image_description": "<h6><strong>A Length</strong></h6>\n<p dir=\"ltr\"><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">o the bottom of the shirt.</span></p>\n<h6>B Width</h6>\n<p dir=\"ltr\">Place the end of the tape at the seam under the sleeve and pull the tape measure across the shirt to the seam under the opposite sleeve.</p>",
        "measurements": [
          {
            "type_label": "Length",
            "values": [
              {
                "size": "S",
                "value": "25.2"
              },
              {
                "size": "M",
                "value": "26"
              },
              {
                "size": "L",
                "value": "26.7"
              },
              {
                "size": "XL",
                "value": "27.6"
              },
              {
                "size": "2XL",
                "value": "28.3"
              }
            ]
          },
          {
            "type_label": "Width",
            "values": [
              {
                "size": "S",
                "value": "16.1"
              },
              {
                "size": "M",
                "value": "17.3"
              },
              {
                "size": "L",
                "value": "18.5"
              },
              {
                "size": "XL",
                "value": "19.7"
              },
              {
                "size": "2XL",
                "value": "20.9"
              }
            ]
          }
        ]
      },
      {
        "type": "international",
        "unit": "none",
        "image_url": "https://s3-printful.stage.printful.dev/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
        "measurements": [
          {
            "type_label": "US size",
            "values": [
              {
                "size": "S",
                "min_value": "6",
                "max_value": "8"
              },
              {
                "size": "M",
                "min_value": "10",
                "max_value": "12"
              },
              {
                "size": "L",
                "min_value": "14",
                "max_value": "16"
              },
              {
                "size": "XL",
                "min_value": "18",
                "max_value": "20"
              },
              {
                "size": "2XL",
                "min_value": "22",
                "max_value": "24"
              }
            ]
          },
          {
            "type_label": "EU size",
            "values": [
              {
                "size": "S",
                "min_value": "36",
                "max_value": "38"
              },
              {
                "size": "M",
                "min_value": "40",
                "max_value": "42"
              },
              {
                "size": "L",
                "min_value": "44",
                "max_value": "46"
              },
              {
                "size": "XL",
                "min_value": "48",
                "max_value": "50"
              },
              {
                "size": "2XL",
                "min_value": "52",
                "max_value": "54"
              }
            ]
          },
          {
            "type_label": "UK size",
            "values": [
              {
                "size": "S",
                "min_value": "10",
                "max_value": "12"
              },
              {
                "size": "M",
                "min_value": "14",
                "max_value": "16"
              },
              {
                "size": "L",
                "min_value": "18",
                "max_value": "20"
              },
              {
                "size": "XL",
                "min_value": "22",
                "max_value": "24"
              },
              {
                "size": "2XL",
                "min_value": "26",
                "max_value": "28"
              }
            ]
          }
        ]
      }
    ],
    "_links": {
      "self": {
        "href": "https://api.printful.test/v2/catalog-products/561/sizes"
      },
      "product": {
        "href": "https://api.printful.test/v2/catalog-products/561"
      }
    }
  },
  "extra": [],
  "debug": []
}
```

</details>

Now, we'll take a closer look at the objects related to all three types of size tables.

### Measure yourself size table

This object provides all measurements for the end customers to be able to measure themselves and see what size they
should buy.

<details>
    <summary>The corresponding response fragment</summary>

```javascript
...
{
    "type": "measure_yourself",
    "unit": "inches",
    "description": "<p>Measurements are provided by suppliers.<br /><br />US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\n<p>Product measurements may vary by up to 2\" (5 cm).&nbsp;</p>",
    "image_url": "https://s3-printful.stage.printful.dev/upload/measure-yourself/6a/6a4fe322f592f2b91d5a735d7ff8d1c0_t?v=1652962720",
    "image_description": "<h6><strong>A Length</strong></h6>\n<p dir=\"ltr\"><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">o the bottom of the shirt.</span></p>\n<h6>B Chest</h6>\n<p dir=\"ltr\">Measure yourself around the fullest part of your chest. Keep the tape measure horizontal.</p>",
    "measurements": [
        {
            "type_label": "Length",
            "values": [
                {
                    "size": "S",
                    "value": "25.2"
                },
                {
                    "size": "M",
                    "value": "26"
                },
                {
                    "size": "L",
                    "value": "26.7"
                },
                {
                    "size": "XL",
                    "value": "27.6"
                },
                {
                    "size": "2XL",
                    "value": "28.3"
                }
            ]
        },
        {
            "type_label": "Chest",
            "values": [
                {
                    "size": "S",
                    "min_value": "34",
                    "max_value": "37"
                },
                {
                    "size": "M",
                    "min_value": "36",
                    "max_value": "39"
                },
                {
                    "size": "L",
                    "min_value": "39",
                    "max_value": "42"
                },
                {
                    "size": "XL",
                    "min_value": "41",
                    "max_value": "44"
                },
                {
                    "size": "2XL",
                    "min_value": "43",
                    "max_value": "46"
                }
            ]
        }
    ]
},
...
```

</details>

The measurement image with the descriptions for the product as seen in the web version of the size guides:

![Image](images/size-guides/my_measurement_image.png)

The size chart from the web version:

![Image](images/size-guides/my_size_chart.png)

### Product measurements size table

This object provides all product measurements so the end customer can measure a product they own and see what size they
should buy.

<details>
    <summary>The corresponding response fragment</summary>

```javascript
...
{
    "type": "product_measure",
    "unit": "inches",
    "description": "<p dir=\"ltr\">Measurements are provided by our suppliers. Product measurements may vary by up to 2\" (5 cm).</p>\n<p dir=\"ltr\">US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\n<p dir=\"ltr\">Pro tip! Measure one of your products at home and compare with the measurements you see in this guide.</p>",
    "image_url": "https://s3-printful.stage.printful.dev/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
    "image_description": "<h6><strong>A Length</strong></h6>\n<p dir=\"ltr\"><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\">o the bottom of the shirt.</span></p>\n<h6>B Width</h6>\n<p dir=\"ltr\">Place the end of the tape at the seam under the sleeve and pull the tape measure across the shirt to the seam under the opposite sleeve.</p>",
    "measurements": [
        {
            "type_label": "Length",
            "values": [
                {
                    "size": "S",
                    "value": "25.2"
                },
                {
                    "size": "M",
                    "value": "26"
                },
                {
                    "size": "L",
                    "value": "26.7"
                },
                {
                    "size": "XL",
                    "value": "27.6"
                },
                {
                    "size": "2XL",
                    "value": "28.3"
                }
            ]
        },
        {
            "type_label": "Width",
            "values": [
                {
                    "size": "S",
                    "value": "16.1"
                },
                {
                    "size": "M",
                    "value": "17.3"
                },
                {
                    "size": "L",
                    "value": "18.5"
                },
                {
                    "size": "XL",
                    "value": "19.7"
                },
                {
                    "size": "2XL",
                    "value": "20.9"
                }
            ]
        }
    ]
},
...
```

</details>

The measurement image with the descriptions for the product as seen in the web version of the size guides:

![Image](images/size-guides/pm_measurement_image.png)

The size chart from the web version:

![Image](images/size-guides/pm_size_chart.png)

### International size conversion

This object provides information what international (US, EU, UK) sizes correspond to the product sizes.

<details>
    <summary>The corresponding response fragment</summary>

```javascript
...
{
    "type": "international",
    "unit": "none",
    "measurements": [
        {
            "type_label": "US size",
            "values": [
                {
                    "size": "S",
                    "min_value": "6",
                    "max_value": "8"
                },
                {
                    "size": "M",
                    "min_value": "10",
                    "max_value": "12"
                },
                {
                    "size": "L",
                    "min_value": "14",
                    "max_value": "16"
                },
                {
                    "size": "XL",
                    "min_value": "18",
                    "max_value": "20"
                },
                {
                    "size": "2XL",
                    "min_value": "22",
                    "max_value": "24"
                }
            ]
        },
        {
            "type_label": "EU size",
            "values": [
                {
                    "size": "S",
                    "min_value": "36",
                    "max_value": "38"
                },
                {
                    "size": "M",
                    "min_value": "40",
                    "max_value": "42"
                },
                {
                    "size": "L",
                    "min_value": "44",
                    "max_value": "46"
                },
                {
                    "size": "XL",
                    "min_value": "48",
                    "max_value": "50"
                },
                {
                    "size": "2XL",
                    "min_value": "52",
                    "max_value": "54"
                }
            ]
        },
        {
            "type_label": "UK size",
            "values": [
                {
                    "size": "S",
                    "min_value": "10",
                    "max_value": "12"
                },
                {
                    "size": "M",
                    "min_value": "14",
                    "max_value": "16"
                },
                {
                    "size": "L",
                    "min_value": "18",
                    "max_value": "20"
                },
                {
                    "size": "XL",
                    "min_value": "22",
                    "max_value": "24"
                },
                {
                    "size": "2XL",
                    "min_value": "26",
                    "max_value": "28"
                }
            ]
        }
    ]
}
...
```

</details>

The international size conversion table from the web version:

![Image](images/size-guides/international_sizes.png)

# File Library API examples

## Add a new file

Add file to the print file library, filename will be detected from URL. After creation file is not processed instantly.

Request body:

```
{
    "url": "http://www.example.com/files/tshirts/example.png"
}
```

Response data:

```
{
    "data": {
        "id": 11816,
        "url": "http://www.example.com/files/tshirts/example.png",
        "hash": null,
        "filename": null,
        "mime_type": null,
        "size": 0,
        "width": null,
        "height": null,
        "dpi": null,
        "status": "waiting",
        "created": "2024-06-10T15:31:49Z",
        "thumbnail_url": null,
        "preview_url": null,
        "visible": true,
        "is_temporary": false,
        "_links": {
            "self": {
                "href": "https://api.printful.com/v2/files/11816"
            }
        }
    }
}
```

Add file to the mockup library, and specify file name manually

Request body:

```
{
    "type": "preview",
    "url": "http://www.example.com/files/tshirts/example.png",
    "filename": "shirt1.png"
}
```

Add file to the library, but not show up in the web interface.

Request body:

``` 
{
    "url": "http://www.example.com/files/tshirts/example.png",
    "visible": false
}
```

# Shipping Rates API examples

## Shipment information

For each entry in the response array, in addition to the basic information about the shipping rate (name, rate,
estimated delivery time),
an expected shipment split will be provided. Each shipment object will contain the following data:

* departure country (2-letter code) – currently, this may be null for some branches. We plan to fix this in the
  foreseeable future;
* shipment items included in the shipment (Catalog Variant ID and item quantity);
* whether customs fees may be required for the shipment (`customs_fees_possible`).

It's possible the fees won't apply even if the value of the `customs_fees_possible` field is `true`, depending on the
characteristics of the products and agreements between the countries. The value of `false` means the customs fees
won't be required for the shipment (e.g. shipment within the EU or the same country).

### Example: Same-country shipment

In this example two shipping options are available. All items can be fulfilled in the US and they are split into two shipments because of large quantities. No customs fees will be charged.

<details>
    <summary>Request body</summary>

```
{
    "recipient": {
        "address1": "19749 Dearborn St",
        "city": "Chatsworth",
        "state_code": "CA",
        "country_code": "US",
        "zip": "91311"
    },
    "order_items": [
        {
            "source": "catalog",
            "quantity": 20,
            "catalog_variant_id": 408
        },
        {
            "source": "catalog",
            "quantity": 20,
            "catalog_variant_id": 4011
        },
        {
            "source": "catalog",
            "quantity": 30,
            "catalog_variant_id": 4016
        },
        {
            "source": "catalog",
            "quantity": 40,
            "catalog_variant_id": 4020
        }
    ],
    "currency": "USD"
}
```

</details>

<details>
    <summary>Response data</summary>

```
{
    "data": [
        {
            "shipping": "STANDARD",
            "shipping_method_name": "Flat Rate (Estimated delivery: Jan 20⁠–27) ",
            "rate": "60.74",
            "currency": "USD",
            "min_delivery_days": 5,
            "max_delivery_days": 8,
            "min_delivery_date": "2025-01-20",
            "max_delivery_date": "2025-01-27",
            "shipments": [
                {
                    "departure_country": "US",
                    "shipment_items": [
                        {
                            "catalog_variant_id": 408,
                            "quantity": 20
                        },
                        {
                            "catalog_variant_id": 4011,
                            "quantity": 20
                        },
                        {
                            "catalog_variant_id": 4016,
                            "quantity": 20
                        }
                    ],
                    "customs_fees_possible": false
                },
                {
                    "departure_country": "US",
                    "shipment_items": [
                        {
                            "catalog_variant_id": 4016,
                            "quantity": 10
                        },
                        {
                            "catalog_variant_id": 4020,
                            "quantity": 40
                        }
                    ],
                    "customs_fees_possible": false
                }
            ]
        },
        {
            "shipping": "STANDARD_CARBON_OFFSET",
            "shipping_method_name": "Standard rate with CO2 offsetting (Estimated delivery: Jan 20⁠–27) ",
            "rate": "70.64",
            "currency": "USD",
            "min_delivery_days": 5,
            "max_delivery_days": 8,
            "min_delivery_date": "2025-01-20",
            "max_delivery_date": "2025-01-27",
            "shipments": [
                {
                    "departure_country": "US",
                    "shipment_items": [
                        {
                            "catalog_variant_id": 408,
                            "quantity": 20
                        },
                        {
                            "catalog_variant_id": 4011,
                            "quantity": 20
                        },
                        {
                            "catalog_variant_id": 4016,
                            "quantity": 20
                        }
                    ],
                    "customs_fees_possible": false
                },
                {
                    "departure_country": "US",
                    "shipment_items": [
                        {
                            "catalog_variant_id": 4016,
                            "quantity": 10
                        },
                        {
                            "catalog_variant_id": 4020,
                            "quantity": 40
                        }
                    ],
                    "customs_fees_possible": false
                }
            ]
        }
    ],
    "extra": []
}
```

</details>

### Example: shipments from different countries

In this example only one shipping option is available. One item can't be shipped from Europe, so it will be shipped from the US instead,
probably incurring customs fees.

<details>
    <summary>Request body</summary>

```
{
    "recipient": {
        "address1": "Warszawska 1",
        "city": "Gdańsk",
        "country_code": "PL"
    },
    "order_items": [
        {
            "source": "catalog",
            "quantity": 1,
            "catalog_variant_id": 408
        },
        {
            "source": "catalog",
            "quantity": 1,
            "catalog_variant_id": 4011
        }
    ]
}
```

</details>

<details>
    <summary>Response data</summary>

```
{
    "data": [
        {
            "shipping": "STANDARD",
            "shipping_method_name": "Flat Rate (Estimated delivery: Jan 21⁠–28) ",
            "rate": "5.54",
            "currency": "EUR",
            "min_delivery_days": 8,
            "max_delivery_days": 14,
            "min_delivery_date": "2025-01-21",
            "max_delivery_date": "2025-01-28",
            "shipments": [
                {
                    "departure_country": "US",
                    "shipment_items": [
                        {
                            "catalog_variant_id": 408,
                            "quantity": 1
                        }
                    ],
                    "customs_fees_possible": true
                },
                {
                    "departure_country": "LV",
                    "shipment_items": [
                        {
                            "catalog_variant_id": 4011,
                            "quantity": 1
                        }
                    ],
                    "customs_fees_possible": false
                }
            ]
        }
    ],
    "extra": []
}
```

</details>

# Webhook API examples

## Configuring webhooks

### Enabling webhook support

We will use an endpoint allowing us to enable webhook support for a store specifying multiple events in one request.

We want to enable support for `order_put_hold`, `shipment_sent`, `shipment_returned`,
and `catalog_stock_updated` events.

We want to receive the event at https://example.org/webhook/store for most events
but for `shipment_returned` we want to use https://example.org/webhook/catalog.

You may replace the URLs with your own. If you don't have a server software that will accept the events,
you may use a site like https://webhook.site/.

We don't want the configuration to expire.

We want to be notified of stock updates for the following product IDs: 1, 71.

Endpoint: `POST https://api.printful.com/v2/webhooks`

The request needs to be authorized with a store-level token
or an account-level token with the store ID specified in `X-PF-Store-ID` header.

<details>
    <summary>Request body</summary>

```
{
    "default_url": "https://example.org/webhook/store",
    "events": [
        {
            "type": "order_put_hold"
        },
        {
            "type": "shipment_sent"
        },
        {
            "type": "shipment_returned",
            "url": "https://example.org/webhook/catalog"
        },
        {
            "type": "catalog_stock_updated",
            "params": [
                {
                    "name": "products",
                    "value": [
                        {
                            "id": 1
                        },
                        {
                            "id": 71
                        }
                    ]
                }
            ]
        }
    ]
}
```

</details>

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "default_url": "https://example.org/webhook/store",
        "expires_at": null,
        "events": [
            {
                "type": "order_put_hold",
                "url": null,
                "params": []
            },
            {
                "type": "shipment_sent",
                "url": null,
                "params": []
            },
            {
                "type": "shipment_returned",
                "url": "https://example.org/webhook/catalog",
                "params": []
            },
            {
                "type": "catalog_stock_updated",
                "url": null,
                "params": [
                    {
                        "name": "products",
                        "value": [
                            {
                                "id": 1
                            },
                            {
                                "id": 71
                            }
                        ]
                    }
                ]
            }
        ],
        "public_key": "somePublicKey",
        "secret_key": "0123456789abcdef..."
    },
    "extra": []
}
```

</details>

The public key will be used to distinguish between events coming from different configurations.

For example, if you use the same URL for different stores or create multiple configurations for the same store
with different tokens for the same URL, you may use the public key to know which configuration was used.

The secret key will be used to verify the signature of the received event.

### Updating or adding event configuration

If you have the general settings created for the store, you may modify or update the settings for specific events.

We will now show how to add support for another event: `order_created`.

Endpoint: `POST https://api.printful.com/v2/webhooks/{eventType}`

If you don't need to specify additional fields like URL, you may send the request without body.
We will send such a request in this example.

Request URL: https://api.printful.com/v2/webhooks/order_created

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "type": "order_created",
        "url": null,
        "params": []
    },
    "extra": []
}
```

</details>

The endpoint may be used to replace existing event settings. It may be used for an already configured
event or a new one.

Request URL: https://api.printful.com/v2/webhooks/shipment_returned

If we omit the request body, we will replace the existing configuration with a custom URL
with a default one, using the store's `default_url`.

However, we want to update the URL with another one.

<details>
    <summary>Request body</summary>

```
{
    "url": "https://example.org/webhook/another"
}
```

</details>

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "type": "shipment_returned",
        "url": "https://example.org/webhook/another",
        "params": []
    },
    "extra": []
}
```

</details>

Note it's not possible to omit the request body for an event requiring a parameter.

Sending a request without body to https://api.printful.com/v2/webhooks/catalog_stock_updated will result in an error.

## Event example

This is an example of an `order_created` event sent to the specified URL:


<details>
    <summary>Event content</summary>

```
{
  "type": "order_created",
  "occurred_at": "2023-04-04T12:36:42Z",
  "retries": 0,
  "store_id": 123456,
  "data": {
    "order": {
      "id": 11111,
      "external_id": null,
      "status": "pending",
      "created": 1680611801,
      "updated": 1680611802,
      "dashboard_url": "https://www.printful.test/dashboard?order_id=90630659",
    }
  }
}
```

</details>

In headers, you will also receive:

* The base64-encoded public key associated with the configuration used to trigger the event in `x-pf-webhook-public-key`
  header
* The hexadecimal representation of the signature in `x-pf-webhook-signature` header.

You may use the secret key associated with the settings to calculate the HMAC-SHA256 hash for the event data
and compare it to the value from header. You may create a script or use an online or offline tool to do this.

Please note that in the `secret_key` field we return the hexadecimal representation of the secret key. It means that you
need to decode it before passing it to the tool or method that calculates the signature, unless it supports keys in
hexadecimal format.

# PATCH Partial Updates

See [RFC 7386](https://datatracker.ietf.org/doc/html/rfc7386).

PATCH updates allows you to change only the fields that you've specified in the request.

If you'd like to update the value that is an array please keep in mind that in case of arrays the entire
array is replaced with the value provided in the request. For example:

Current array value:
```
{
  "items": [
    {
      "catalog_variant_id": 4011,
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
    },
    {
      "catalog_variant_id": 4012,
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
        },
        {
          "placement": "back",
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

PATCH request:
```
{
  "items": [
    {
      "catalog_variant_id": 4013,
      "placements": [
        {
          "placement": "left_sleeve",
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

Results in:
```
{
  "items": [
    {
      "catalog_variant_id": 4013,
      "placements": [
        {
          "placement": "left_sleeve",
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

Notice that all of the previous items were discarded and were replaced with items provided in the request.
It's also not possible to override properties in nested arrays. So whenever you are changing the array
value you need to provide full object.

## Example of order update

<details>
    <summary>Order that is being updated</summary>

```
{
    "data": {
        "id": 94188292,
        "external_id": null,
        "store_id": 11229252,
        "shipping": "STANDARD",
        "status": "draft",
        "created_at": "2023-10-18T12:35:07Z",
        "updated_at": "2023-10-18T12:35:07Z",
        "recipient": {
            "name": "John Smith",
            "company": "John Smith Inc",
            "address1": "19749 Dearborn St",
            "address2": "string",
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": "string",
            "email": "dev@printful.com",
            "tax_number": "123.456.789-10"
        },
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
            "subtotal": null,
            "discount": "0.00",
            "shipping": "0.00",
            "tax": "0.00",
            "vat": "0.00",
            "total": null
        },
        "order_items": [
            {
                "id": 66655730,
                "external_id": "some_external_id",
                "type": "order_item",
                "source": "catalog",
                "catalog_variant_id": 4012,
                "quantity": 1,
                "name": "my_custom_name",
                "price": "10.50",
                "retail_price": null,
                "currency": "EUR",
                "retail_currency": "USD",
                "_links": {
                    "self": {
                        "href": "https://api.printful.test/v2/orders/94188292/order-items/66655730"
                    }
                }
            }
        ],
        "customization": null,
        "_links": {
            "self": {
                "href": "https://api.printful.test/v2/orders/94188292"
            },
            "order_confirmation": {
                "href": "https://api.printful.test/v2/orders/94188292/confirmation"
            },
            "order_items": {
                "href": "https://api.printful.test/v2/orders/94188292/order-items"
            },
            "shipments": {
                "href": "https://api.printful.test/v2/orders/94188292/shipments"
            }
        }
    },
    "extra": [],
}
```

</details>

### Changing specified fields in recipient object

Endpoint `PATCH https://api.printful.com/v2/orders/{order_id}`
<details>
    <summary>Request body</summary>

```
{
  "recipient": {
    "address1": "new address",
    "name": "new name"
  }
}
```

</details>

Endpoint `GET https://api.printful.com/v2/orders/{order_id}`
<details>
    <summary>Response body</summary>

```
{
    "data": {
        "id": 94188292,
        "external_id": null,
        "store_id": 11229252,
        "shipping": "STANDARD",
        "status": "draft",
        "created_at": "2023-10-18T12:35:07Z",
        "updated_at": "2023-10-18T12:35:07Z",
        "recipient": {
            "name": "new name", <--------Changed field
            "company": "John Smith Inc",
            "address1": "new address", <--------Changed field
            "address2": "string",
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": "string",
            "email": "dev@printful.com",
            "tax_number": "123.456.789-10"
        },
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
            "subtotal": null,
            "discount": "0.00",
            "shipping": "0.00",
            "tax": "0.00",
            "vat": "0.00",
            "total": null
        },
        "order_items": [
            {
                "id": 66655730,
                "external_id": "some_external_id",
                "type": "order_item",
                "source": "catalog",
                "catalog_variant_id": 4012,
                "quantity": 1,
                "name": "my_custom_name",
                "price": "10.50",
                "retail_price": null,
                "currency": "EUR",
                "retail_currency": "USD",
                "_links": {
                    "self": {
                        "href": "https://api.printful.test/v2/orders/94188292/order-items/66655730"
                    }
                }
            }
        ],
        "customization": null,
        "_links": {
            "self": {
                "href": "https://api.printful.test/v2/orders/94188292"
            },
            "order_confirmation": {
                "href": "https://api.printful.test/v2/orders/94188292/confirmation"
            },
            "order_items": {
                "href": "https://api.printful.test/v2/orders/94188292/order-items"
            },
            "shipments": {
                "href": "https://api.printful.test/v2/orders/94188292/shipments"
            }
        }
    },
    "extra": [],
}
```

</details>

### Changing `external_id` property

Endpoint `PATCH https://api.printful.com/v2/orders/{order_id}`
<details>
    <summary>Request body</summary>

```
{
  "external_id": "1234_abc"
}
```

</details>

Endpoint `GET https://api.printful.com/v2/orders/{order_id}`
<details>
    <summary>Response body</summary>

```
{
    "data": {
        "id": 94188292,
        "external_id": "1234_abc", <--------Changed field
        "store_id": 11229252,
        "shipping": "STANDARD",
        "status": "draft",
        "created_at": "2023-10-18T12:35:07Z",
        "updated_at": "2023-10-18T12:35:07Z",
        "recipient": {
            "name": "John Smith",
            "company": "John Smith Inc",
            "address1": "19749 Dearborn St",
            "address2": "string",
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": "string",
            "email": "dev@printful.com",
            "tax_number": "123.456.789-10"
        },
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
            "subtotal": null,
            "discount": "0.00",
            "shipping": "0.00",
            "tax": "0.00",
            "vat": "0.00",
            "total": null
        },
        "order_items": [
            {
                "id": 66655730,
                "external_id": "some_external_id",
                "type": "order_item",
                "source": "catalog",
                "catalog_variant_id": 4012,
                "quantity": 1,
                "name": "my_custom_name",
                "price": "10.50",
                "retail_price": null,
                "currency": "EUR",
                "retail_currency": "USD",
                "_links": {
                    "self": {
                        "href": "https://api.printful.test/v2/orders/94188292/order-items/66655730"
                    }
                }
            }
        ],
        "customization": null,
        "_links": {
            "self": {
                "href": "https://api.printful.test/v2/orders/94188292"
            },
            "order_confirmation": {
                "href": "https://api.printful.test/v2/orders/94188292/confirmation"
            },
            "order_items": {
                "href": "https://api.printful.test/v2/orders/94188292/order-items"
            },
            "shipments": {
                "href": "https://api.printful.test/v2/orders/94188292/shipments"
            }
        }
    },
    "extra": [],
}
```

</details>

### Replacing ordered item with another one

Endpoint `PATCH https://api.printful.com/v2/orders/{order_id}`
<details>
    <summary>Request body</summary>

```
{
  "items": [
        {
            "source": "catalog",
            "catalog_variant_id": 4011,
            "quantity": 1,
            "placements": [
                {
                    "placement": "front",
                    "technique": "dtg",
                    "layers": [
                        {
                            "type": "file",
                            "url": "https://www.printful.com/static/images/layout/printful-logo.png",
                        }
                    ]
                }
            ]
        }
    ]
}
```

</details>

Endpoint `GET https://api.printful.com/v2/orders/{order_id}`
<details>
    <summary>Response body</summary>

```
{
    "data": {
        "id": 94188292,
        "external_id": null,
        "store_id": 11229252,
        "shipping": "STANDARD",
        "status": "draft",
        "created_at": "2023-10-18T12:35:07Z",
        "updated_at": "2023-10-18T12:35:07Z",
        "recipient": {
            "name": "John Smith",
            "company": "John Smith Inc",
            "address1": "19749 Dearborn St",
            "address2": "string",
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": "string",
            "email": "dev@printful.com",
            "tax_number": "123.456.789-10"
        },
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
            "subtotal": null,
            "discount": "0.00",
            "shipping": "0.00",
            "tax": "0.00",
            "vat": "0.00",
            "total": null
        },
        "order_items": [ <--------Changed field
            {
                "id": 66655731,
                "external_id": null,
                "type": "order_item",
                "source": "catalog",
                "catalog_variant_id": 4011,
                "quantity": 1,
                "name": null,
                "price": "10.50",
                "retail_price": null,
                "currency": "EUR",
                "retail_currency": "USD",
                "_links": {
                    "self": {
                        "href": "https://api.printful.test/v2/orders/94188292/order-items/66655730"
                    }
                }
            }
        ],
        "customization": null,
        "_links": {
            "self": {
                "href": "https://api.printful.test/v2/orders/94188292"
            },
            "order_confirmation": {
                "href": "https://api.printful.test/v2/orders/94188292/confirmation"
            },
            "order_items": {
                "href": "https://api.printful.test/v2/orders/94188292/order-items"
            },
            "shipments": {
                "href": "https://api.printful.test/v2/orders/94188292/shipments"
            }
        }
    },
    "extra": [],
}
```

</details>

# Utilising options

Options are the special properties which are used to customise the products. Depending on the product and the technique different options might be available.
You can check the available options for a product in [Retrieve a single catalog product](#tag/Catalog-v2/operation/getProductById). Currently, there are 3 types of options each referencing a different ascpect of the design:
* `product_options` - options which affect the entire product e.g. if the product should contain inside pocket or not.
* `placement_options` - options which are affecting the specific placement
* `layer_options` - options which are affecting the specific layer e.g. thread colors

### Custom border color option

Stickers can have a different border color which can be set by using the `thread_colors_outline` option.
This option is available in `product_options` for stickers. To showcase the usage we will use the order flow
which will create order with a sticker that will have a red border color:

Endpoint `POST https://api.printful.com/v2/orders`
<details>
    <summary>Request body</summary>

```
{
    "recipient": {
        "name": "John Smith",
        "company": "John Smith Inc",
        "address1": "19749 Dearborn St",
        "address2": "string",
        "city": "Chatsworth",
        "state_code": "CA",
        "state_name": "California",
        "country_code": "US",
        "country_name": "United States",
        "zip": "91311"
    },
    "items": [
        {
            "quantity": 1,
            "variant_id": 10163,
            "source": "catalog",
            "placements": [
                {
                    "placement": "default",
                    "technique": "digital",
                    "layers": [
                        {
                            "type": "file",
                            "url": "https://www.printful.com/static/images/layout/printful-logo.png",
                            "position": {
                                "width": 3,
                                "height": 3,
                                "top": 0,
                                "left": 0
                            }
                        }
                    ]
                }
            ],
            "product_options": [
                {
                    "name": "custom_border_color",
                    "value": "#FFFF00"
                }
            ]
        }
    ]
}
```

</details>

# Creating Orders for Knitwear Products 

Knitwear products can be configured with some unique options that affect the color of the final knitted product.

These options are `trim_color`, `color_reduction_mode`, `base_color`, they only apply to
knitwear and function differently to other similar options.

Note that `yarn_colors` can be applied on each layer of a product.


Endpoint `POST https://api.printful.com/v2/orders`
<details>
    <summary>Request body</summary>

```
{
  "recipient": {
    "name": "John Smith",
    "company": "John Smith Inc",
    "address1": "19749 Dearborn St",
    "city": "Chatsworth",
    "state_code": "CA",
    "country_code": "US",
    "country_name": "United States",
    "zip": "91311"
  },
  "order_items": [
    {
      "source": "catalog",
      "quantity": 1,
      "catalog_variant_id": 19634,
      "placements": [
        {
          "placement": "sleeve_right",
          "technique": "knitting",
          "layers": [
            {
              "type": "file",
              "url": "https://picsum.photos/200",
              "layer_options": [
                {
                  "name": "yarn_colors",
                  "value": [
                    "#090909",
                    "#48542e"
                  ]
                }
              ]
            }
          ]
        }
      ],
      "product_options": [
        {
          "name": "trim_color",
          "value": "#dcd3cc"
        },
        {
          "name": "color_reduction_mode",
          "value": "pixelated"
        },
        {
          "name": "base_color",
          "value": "#dda032"
        }
      ]
    }
  ]
}
```

</details>



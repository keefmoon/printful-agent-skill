# Examples

# Catalog API examples

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

## Filter products by single category ID

The IDs of the categories may be retrieved using the `GET /categories` endpoint. We're going to list products belonging
to the "Samsung cases" category (ID: 62).

Request URL: https://api.printful.com/products?category_id=62

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": [
        {
            "id": 267,
            "type": "PHONE-CASE",
            "type_name": "Samsung Case",
            "title": "Samsung Case",
            "brand": null,
            "model": "Samsung Case",
            "image": "https://s3-printful.stage.printful.dev/upload/product-catalog-img/7c/7c7a0c9bba1ab7ead3c03753adc262b6_l",
            "variant_count": 13,
            "currency": "USD",
            "options": [],
            "dimensions": null,
            "is_discontinued": false,
            "avg_fulfillment_time": null,
            "techniques": [
                {
                    "key": "UV",
                    "display_name": "UV printing",
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
            "description": "This sleek Samsung case protects your phone from scratches, dust, oil, and dirt. It has a solid back and flexible sides that make it easy to take on and off, with precisely aligned cuts and holes. \r\n\r\n• BPA free Hybrid Thermoplastic Polyurethane (TPU) and Polycarbonate (PC) material\r\n• Solid polycarbonate back\r\n• 0.02″ (0.5 mm) raised bezel\r\n• See-through sides\r\n• Wireless charging compatible\r\n• Easy to take on and off"
        }
    ],
    "extra": []
}
```

</details>

## Filter products by multiple category IDs

The following request will fetch iPhone cases (ID: 50) and Samsung cases (ID: 62).

Request URL: https://api.printful.com/products?category_id=50,62

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": [
        {
            "id": 424,
            "type": "PHONE-CASE",
            "type_name": "Eco iPhone Case",
            "title": "Biodegradable iPhone Case",
            "brand": null,
            "model": "Eco iPhone Case",
            "image": "https://s3-printful.stage.printful.dev/upload/product-catalog-img/07/07eabbd80051c972d900332d5523dc5a_l",
            "variant_count": 14,
            "currency": "USD",
            "options": [],
            "dimensions": null,
            "is_discontinued": false,
            "avg_fulfillment_time": null,
            "techniques": [
                {
                    "key": "UV",
                    "display_name": "UV printing",
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
            "description": "Protect your phone and the environment all in one go—this phone case is eco-friendly and 100% biodegradable. Cover your phone with a unique case to protect it from bumps and scratches in style.\r\n\r\n• 100% biodegradable material\r\n• Components: soil (30%), onions (7.5%), carrots (7.5%), pepper (7.5%), sawdust (1.5%), rice (18%), soybeans (18%), wheat (10%)\r\n• Anti-shock protection\r\n• Thickness over 1.8mm\r\n• Decomposes in ~1 year\r\n• Packaged in a degradable and protective CPE 07 bag and shipped in a carton box\r\n• The SE case fits the 2020 iPhone SE model\r\n• Blank product sourced from China "
        },
        {
            "id": 181,
            "type": "PHONE-CASE",
            "type_name": "iPhone Case",
            "title": "iPhone Case",
            "brand": null,
            "model": "iPhone Case",
            "image": "https://s3-printful.stage.printful.dev/products/181/product_1570615955.jpg",
            "variant_count": 17,
            "currency": "USD",
            "options": [],
            "dimensions": {
                "default": "2.93x6.1"
            },
            "is_discontinued": false,
            "avg_fulfillment_time": null,
            "techniques": [
                {
                    "key": "UV",
                    "display_name": "UV printing",
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
            "description": "This sleek iPhone case protects your phone from scratches, dust, oil, and dirt. It has a solid back and flexible sides that make it easy to take on and off, with precisely aligned port openings. \r\n\r\n• BPA free Hybrid Thermoplastic Polyurethane (TPU) and Polycarbonate (PC) material\r\n• Solid polycarbonate back\r\n• Flexible, see-through polyurethane sides\r\n• .5 mm raised bezel\r\n• Precisely aligned port openings\r\n• Easy to take on and off\r\n• Wireless charging compatible\r\n• The SE case fits the 2020 iPhone SE model\r\n• Blank product sourced from China"
        },
        {
            "id": 267,
            "type": "PHONE-CASE",
            "type_name": "Samsung Case",
            "title": "Samsung Case",
            "brand": null,
            "model": "Samsung Case",
            "image": "https://s3-printful.stage.printful.dev/upload/product-catalog-img/7c/7c7a0c9bba1ab7ead3c03753adc262b6_l",
            "variant_count": 13,
            "currency": "USD",
            "options": [],
            "dimensions": null,
            "is_discontinued": false,
            "avg_fulfillment_time": null,
            "techniques": [
                {
                    "key": "UV",
                    "display_name": "UV printing",
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
            "description": "This sleek Samsung case protects your phone from scratches, dust, oil, and dirt. It has a solid back and flexible sides that make it easy to take on and off, with precisely aligned cuts and holes. \r\n\r\n• BPA free Hybrid Thermoplastic Polyurethane (TPU) and Polycarbonate (PC) material\r\n• Solid polycarbonate back\r\n• 0.02″ (0.5 mm) raised bezel\r\n• See-through sides\r\n• Wireless charging compatible\r\n• Easy to take on and off"
        }
    ],
    "extra": []
}
```

</details>

## Using size guides

In this example, we'll fetch the size guides for the
"Women's Basic Organic T-Shirt | SOL'S 02077" product (ID: 561).

We use the default `en_US` locale and don't provide the `unit` parameter, so the measurement values will be returned in
inches.

URL: https://api.printful.com/products/561/sizes

<details>
    <summary>Whole response body</summary>

```
{
    "code": 200,
    "result": {
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
        ]
    },
    "extra": []
}
```

</details>

Now, we'll take a closer look at the objects related to all three types of size tables.

### Measure yourself size table

This object provides all measurements for the end customers to be able to measure themselves and see what size they
should buy.

<details>
    <summary>The corresponding response fragment</summary>

```
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

```
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

```
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

## Product containing Unlimited Color option

The following request will fetch Bella Canvas 3001 product (ID: 71).

Request URL: https://api.printful.com/products/71

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "product": {
            "id": 71,
            "main_category_id": 24,
            "type": "T-SHIRT",
            "description": "This t-shirt is everything you've dreamed of and more. It feels soft and lightweight, with the right amount of stretch. It's comfortable and flattering for all. \n\n• 100% combed and ring-spun cotton (Heather colors contain polyester)\n• Fabric weight: 4.2 oz/yd² (142 g/m²)\n• Pre-shrunk fabric\n• Side-seamed construction\n• Shoulder-to-shoulder taping\n• Blank product sourced from Guatemala, Nicaragua, Mexico, Honduras, or the US",
            "type_name": "T-Shirt",
            "title": "Unisex Staple T-Shirt | Bella + Canvas 3001",
            "brand": "Bella + Canvas",
            "model": "3001 Unisex Short Sleeve Jersey T-Shirt",
            "image": "https://s3-printful.stage.printful.dev/upload/product-catalog-img/20/2079a3ee4cc472ad952fe16654f274cd_l",
            "variant_count": 378,
            "currency": "EUR",
            "options": [
                ...
            ],
            "dimensions": null,
            "is_discontinued": false,
            "avg_fulfillment_time": null,
            "techniques": [
                {
                    "key": "EMBROIDERY",
                    "display_name": "Embroidery",
                    "is_default": false
                },
                {
                    "key": "DTG",
                    "display_name": "DTG printing",
                    "is_default": true
                }
            ],
            "files": [
                {
                    "id": "embroidery_chest_left",
                    "type": "embroidery_chest_left",
                    "title": "Left chest",
                    "additional_price": "2.60",
                    "options": [
                        {
                            "id": "full_color",
                            "type": "bool",
                            "title": "Unlimited color",
                            "additional_price": 3.25
                        }
                    ]
                },
                {
                    "id": "embroidery_chest_center",
                    "type": "embroidery_chest_center",
                    "title": "Center chest",
                    "additional_price": "2.60",
                    "options": [
                        {
                            "id": "full_color",
                            "type": "bool",
                            "title": "Unlimited color",
                            "additional_price": 3.25
                        }
                    ]
                },
                {
                    "id": "embroidery_sleeve_left_top",
                    "type": "embroidery_sleeve_left_top",
                    "title": "Left sleeve top",
                    "additional_price": "2.60",
                    "options": []
                },
                {
                    "id": "embroidery_sleeve_right_top",
                    "type": "embroidery_sleeve_right_top",
                    "title": "Right sleeve top",
                    "additional_price": "2.60",
                    "options": []
                },
                {
                    "id": "default",
                    "type": "front",
                    "title": "Front print",
                    "additional_price": null,
                    "options": []
                },
                {
                    "id": "front_large",
                    "type": "front_large",
                    "title": "Large front print",
                    "additional_price": "5.25",
                    "options": []
                },
                {
                    "id": "back",
                    "type": "back",
                    "title": "Back print",
                    "additional_price": "5.25",
                    "options": []
                },
                {
                    "id": "label_outside",
                    "type": "label_outside",
                    "title": "Outside label",
                    "additional_price": "2.20",
                    "options": []
                },
                {
                    "id": "label_inside",
                    "type": "label_inside",
                    "title": "Inside label",
                    "additional_price": "2.20",
                    "options": []
                },
                {
                    "id": "sleeve_left",
                    "type": "sleeve_left",
                    "title": "Left sleeve",
                    "additional_price": "2.20",
                    "options": []
                },
                {
                    "id": "sleeve_right",
                    "type": "sleeve_right",
                    "title": "Right sleeve",
                    "additional_price": "2.20",
                    "options": []
                },
                {
                    "id": "preview",
                    "type": "mockup",
                    "title": "Mockup",
                    "additional_price": null,
                    "options": []
                }
            ],
            "catalog_categories": {
                "0": 24,
                "1": 1,
                "2": 2,
                "3": 6,
                "4": 8,
                "5": 32,
                "6": 85,
                "7": 89,
                "10": 155,
                "11": 164,
                "12": 188,
                "13": 213,
                "14": 226,
                "15": 227,
                "16": 233,
                "17": 234,
                "18": 260
            }
        },
        "variants": [
            ...
        ]
    },
    "extra": [],
}
```

</details>

# Products API examples

## Create a new Sync Product

### Using multiple placements

Request body:

```
{
    "sync_product": {
        "name": "API product Bella",
        "thumbnail": "https://example.com/image.jpg"
    },
    "sync_variants": [
        {
            "retail_price": "21.00",
            "variant_id": 4011,
            "files": [
                {
                    "url": "https://example.com/image.jpg"
                },
                {
                    "type": "back",
                    "url": "https://example.com/image.jpg"
                }
            ]
        },
        {
            "retail_price": "21.00",
            "variant_id": 4012,
            "files": [
                {
                    "url": "https://example.com/image.jpg"
                },
                {
                    "type": "back",
                    "url": "https://example.com/image.jpg"
                }
            ]
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 79348732,
        "external_id": "5bd9947c709b34",
        "name": "API product Bella",
        "variants": 2,
        "synced": 2
    }
}
```
</details>

### Using inside label

Create a new Sync Product with native inside label.

Request body:

```
{
    "sync_product": {
        "name": "API product custom"
    },
    "sync_variants": [
        {
            "retail_price": "19.00",
            "variant_id": 9575,
            "files": [
                {
                    "type": "front",
                    "url": "https://picsum.photos/200/300"
                },
                {
                    "type": "label_inside",
                    "url": "https://picsum.photos/200/300",
                    "options": [{
                        "id": "template_type",
                        "value": "native"
                    }]
                }
            ],
            "options": [
                {
                    "id": "embroidery_type",
                    "value": "flat"
                },
                {
                    "id": "thread_colors",
                    "value": []
                },
                {
                    "id": "thread_colors_3d",
                    "value": []
                },
                {
                    "id": "thread_colors_chest_left",
                    "value": []
                }
            ]
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 166266931,
        "external_id": "5e8db0013cf026",
        "name": "API product custom",
        "variants": 1,
        "synced": 1,
        "thumbnail_url": null
    }
}
```
</details>

## Modify a Sync Product

Update a Sync Product's Name and Thumbnail.

Request body:

```
{
    "sync_product": {
        "name": "API product new name",
        "thumbnail": "https://example.com/image.jpg"
    }
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 79348721,
        "external_id": "e9460f6c67",
        "name": "API product new name",
        "variants": 2,
        "synced": 2
    }
}
```
</details>

Update a Sync Product and one of its Sync Variants.

Request body:

```
{
    "sync_product": {
        "name": "API product new name",
        "thumbnail": "https://example.com/image.jpg"
    },
    "sync_variants": [
            {
                "id": 866914574
            },
            {
                "id": 866914580,
                "retail_price": 21,
                "files": [
                    {
                        "url": "https://example.com/image.jpg"
                    },
                    {
                        "type": "back",
                        "url": "https://example.com/image.jpg"
                    }
                ]
            }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 79348721,
        "external_id": "e9460f6c67",
        "name": "API product",
        "variants": 2,
        "synced": 2
    }
}
```
</details>

## Modify a Sync Variant

Update price of an existing Sync Variant.

Request body:

```
{
    "retail_price": "29.00"
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 866914574,
        "external_id": "5bd967595a1174",
        "sync_product_id": 79348721,
        "name": "API product",
        "synced": true,
        "variant_id": 4011,
        "retail_price": "29.00",
        "currency": "",
        "product": {
            "variant_id": 4011,
            "product_id": 71,
            "image": "https://s3.dev.printful.com/products/71/4012_1517927381.jpg",
            "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / S)"
        },
        "files": [
            {
                "id": 76564075,
                "type": "default",
                "hash": "7d6a2367c1e338750e68dc66b20cba1a",
                "url": "https://example.com/image.jpg",
                "filename": "76564075.jpg",
                "mime_type": "image/jpeg",
                "size": 8245,
                "width": 200,
                "height": 300,
                "dpi": null,
                "status": "ok",
                "created": 1539341673,
                "thumbnail_url": "https://s3.dev.printful.com/files/7d6/7d6a2367c1e338750e68dc66b20cba1a_thumb.png",
                "preview_url": "https://s3.dev.printful.com/files/7d6/7d6a2367c1e338750e68dc66b20cba1a_preview.png",
                "visible": true
            },
            {
                "id": 76564075,
                "type": "back",
                "hash": "7d6a2367c1e338750e68dc66b20cba1a",
                "url": "https://example.com/image.jpg",
                "filename": "76564075.jpg",
                "mime_type": "image/jpeg",
                "size": 8245,
                "width": 200,
                "height": 300,
                "dpi": null,
                "status": "ok",
                "created": 1539341673,
                "thumbnail_url": "https://s3.dev.printful.com/files/7d6/7d6a2367c1e338750e68dc66b20cba1a_thumb.png",
                "preview_url": "https://s3.dev.printful.com/files/7d6/7d6a2367c1e338750e68dc66b20cba1a_preview.png",
                "visible": true
            }
        ],
        "options": [
            {
                "id": "embroidery_type",
                "value": "flat"
            },
            {
                "id": "thread_colors",
                "value": []
            },
            {
                "id": "thread_colors_3d",
                "value": []
            },
            {
                "id": "thread_colors_chest_left",
                "value": []
            }
        ]
    }
}
```

</details>

Add a native inside label to an existing Sync Variant.

Request body:

```
{
    "files": [
        {
            "type": "label_inside",
            "url": "https://picsum.photos/200/300",
            "options": [{
                "id": "template_type",
                "value": "native"
            }]
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 1817548049,
        "external_id": "5e8dbb006e62d5",
        "sync_product_id": 162979476,
        "name": "lucia - White / XL",
        "synced": true,
        "variant_id": 4014,
        "retail_price": "19.00",
        "currency": "USD",
        "product": {
            "variant_id": 4014,
            "product_id": 71,
            "image": "https://s3.dev.printful.com/products/71/4014_1581412553.jpg",
            "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / XL)"
        },
        "files": [
            {
                "id": 185425196,
                "type": "label_inside",
                "hash": null,
                "url": "https://picsum.photos/200/300",
                "filename": null,
                "mime_type": null,
                "size": 0,
                "width": null,
                "height": null,
                "dpi": null,
                "status": "waiting",
                "created": 1586351368,
                "thumbnail_url": null,
                "preview_url": null,
                "visible": true
            },
        ],
        "options": [
            {
            "id": "embroidery_type",
            "value": "flat"
            },
            {
                "id": "thread_colors",
                "value": []
            },
            {
                "id": "text_thread_colors",
                "value": []
            },
            {
                "id": "thread_colors_3d",
                "value": []
            },
            {
                "id": "thread_colors_chest_left",
                "value": []
            },
            {
                "id": "text_thread_colors_chest_left",
                "value": []
            },
            {
                "id": "thread_colors_chest_center",
                "value": []
            },
            {
                "id": "text_thread_colors_chest_center",
                "value": []
            }
        ]
    }
}
```

</details>

Update the variant t-shirt to have only front print instead of both front and back prints.

Request body:

```
{
    "files": [
        {
            "type": "default",
            "url": "https://example.com/image.jpg"
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 866914574,
        "external_id": "5bd967595a1174",
        "sync_product_id": 79348721,
        "name": "API product",
        "synced": true,
        "variant_id": 4011,
        "retail_price": "29.00",
        "currency": "",
        "product": {
            "variant_id": 4011,
            "product_id": 71,
            "image": "https://s3.dev.printful.com/products/71/4012_1517927381.jpg",
            "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / S)"
        },
        "files": [
            {
                "id": 76564159,
                "type": "default",
                "hash": "ebd559858e5703088de8900ce99c37d3",
                "url": "https://example.com/image.jpg",
                "filename": "76564159.jpg",
                "mime_type": "image/jpeg",
                "size": 11246,
                "width": 200,
                "height": 300,
                "dpi": null,
                "status": "ok",
                "created": 1540797879,
                "thumbnail_url": "https://s3.dev.printful.com/files/ebd/ebd559858e5703088de8900ce99c37d3_thumb.png",
                "preview_url": "https://s3.dev.printful.com/files/ebd/ebd559858e5703088de8900ce99c37d3_preview.png",
                "visible": true
            }
        ],
        "options": [
            {
                "id": "embroidery_type",
                "value": "flat"
            },
            {
                "id": "thread_colors",
                "value": []
            },
            {
                "id": "thread_colors_3d",
                "value": []
            },
            {
                "id": "thread_colors_chest_left",
                "value": []
            }
        ]
    }
}
```

</details>

## Create a new Sync Variant

Request body:

```
{
    "external_id": "my-external-id",
    "retail_price": "19.00",
    "variant_id": 4011,
    "files": [
        {
            "type": "default",
            "url": "https://example.com/image.jpg"
        },
        {
            "type": "back",
            "url": "https://example.com/image.jpg"
        }
    ],
    "options": [
        {
            "id": "embroidery_type",
            "value": "flat"
        },
        {
            "id": "thread_colors",
            "value": []
        },
        {
            "id": "thread_colors_3d",
            "value": []
        },
        {
            "id": "thread_colors_chest_left",
            "value": []
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 866914592,
        "external_id": "my-external-id",
        "sync_product_id": 79348732,
        "name": "API product Bella",
        "synced": true,
        "variant_id": 4011,
        "retail_price": "19.00",
        "currency": "USD",
        "product": {
            "variant_id": 4011,
            "product_id": 71,
            "image": "https://s3.dev.printful.com/products/71/4012_1517927381.jpg",
            "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / S)"
        },
        "files": [
            {
                "id": 76564159,
                "type": "default",
                "hash": "ebd559858e5703088de8900ce99c37d3",
                "url": "https://example.com/image.jpg",
                "filename": "76564159.jpg",
                "mime_type": "image/jpeg",
                "size": 11246,
                "width": 200,
                "height": 300,
                "dpi": null,
                "status": "ok",
                "created": 1540797879,
                "thumbnail_url": "https://s3.dev.printful.com/files/ebd/ebd559858e5703088de8900ce99c37d3_thumb.png",
                "preview_url": "https://s3.dev.printful.com/files/ebd/ebd559858e5703088de8900ce99c37d3_preview.png",
                "visible": true
            },
            {
                "id": 76564159,
                "type": "back",
                "hash": "ebd559858e5703088de8900ce99c37d3",
                "url": "https://example.com/image.jpg",
                "filename": "76564159.jpg",
                "mime_type": "image/jpeg",
                "size": 11246,
                "width": 200,
                "height": 300,
                "dpi": null,
                "status": "ok",
                "created": 1540797879,
                "thumbnail_url": "https://s3.dev.printful.com/files/ebd/ebd559858e5703088de8900ce99c37d3_thumb.png",
                "preview_url": "https://s3.dev.printful.com/files/ebd/ebd559858e5703088de8900ce99c37d3_preview.png",
                "visible": true
            }
        ],
        "options": [
            {
                "id": "embroidery_type",
                "value": "flat"
            },
            {
                "id": "thread_colors",
                "value": []
            },
            {
                "id": "thread_colors_3d",
                "value": []
            },
            {
                "id": "thread_colors_chest_left",
                "value": []
            }
        ]
    }
}
```

</details>


Create a new Sync Variant with a native inside label.

__Please note that the inside label type must be specified in the file options.__

Request body:

```
{
    "retail_price": "19.00",
    "variant_id": 4025,
    "files": [
        {
            "type": "default",
            "url": "https://example.com/image.jpg"
        },
        {
            "type": "label_inside",
            "url": "https://example.com/image.jpg",
            "options": [{
                "id": "template_type",
                "value": "native"
            }]
        }
    ],
    "options": [
        {
            "id": "embroidery_type",
            "value": "flat"
        },
        {
            "id": "thread_colors",
            "value": []
        },
        {
            "id": "thread_colors_3d",
            "value": []
        },
        {
            "id": "thread_colors_chest_left",
            "value": []
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 1817548049,
        "external_id": "5e8dbb006e62d5",
        "sync_product_id": 162979476,
        "name": "lucia - White / XL",
        "synced": true,
        "variant_id": 4014,
        "retail_price": "19.00",
        "currency": "USD",
        "product": {
            "variant_id": 4014,
            "product_id": 71,
            "image": "https://s3.dev.printful.com/products/71/4014_1581412553.jpg",
            "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / XL)"
        },
        "files": [
            {
            "id": 185425195,
            "type": "default",
            "hash": null,
            "url": "https://example.com/image.jpg",
            "filename": null,
            "mime_type": null,
            "size": 0,
            "width": null,
            "height": null,
            "dpi": null,
            "status": "waiting",
            "created": 1586346752,
            "thumbnail_url": null,
            "preview_url": null,
            "visible": true
        },
        {
            "id": 185425195,
            "type": "label_inside",
            "hash": null,
            "url": "https://example.com/image.jpg",
            "filename": null,
            "mime_type": null,
            "size": 0,
            "width": null,
            "height": null,
            "dpi": null,
            "status": "waiting",
            "created": 1586346752,
            "thumbnail_url": null,
            "preview_url": null,
            "visible": true
        }
        ],
        "options": [
            {
            "id": "embroidery_type",
            "value": "flat"
            },
            {
                "id": "thread_colors",
                "value": []
            },
            {
                "id": "text_thread_colors",
                "value": []
            },
            {
                "id": "thread_colors_3d",
                "value": []
            },
            {
                "id": "thread_colors_chest_left",
                "value": []
            },
            {
                "id": "text_thread_colors_chest_left",
                "value": []
            },
            {
                "id": "thread_colors_chest_center",
                "value": []
            },
            {
                "id": "text_thread_colors_chest_center",
                "value": []
            }
        ]
    }
}
```

</details>

## Filter products by single category ID

The IDs of the categories may be retrieved using the GET /categories endpoint. We're going to list products belonging to the "T-shirts" category (ID: 24).

Request URL: https://api.printful.com/store/products?category_id=24

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": [
        {
            "id": 277353234,
            "external_id": "62f35549aeefa2",
            "name": "API product Bella Test 1",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353233,
            "external_id": "62f26ed43060d9",
            "name": "API product Bella Test 1",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353218,
            "external_id": "62e139cbdf6946",
            "name": "API product Bella Test 1",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353216,
            "external_id": "62e0fd3e89b655",
            "name": "API product Bella Test 1",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        }
    ],
    "extra": [],
    "paging": {
        "total": 194,
        "offset": 0,
        "limit": 20
    },
}
```

</details>

## Filter products by multiple category IDs

The following request will fetch T-shirts cases (ID: 24) and Dad hats (ID: 42).

Request URL: https://api.printful.com/store/products?category_id=24,42

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": [
        {
            "id": 277353234,
            "external_id": "62f35549aeefa2",
            "name": "API product Bella Test 1",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353233,
            "external_id": "62f26ed43060d9",
            "name": "API product Bella Test 1",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353218,
            "external_id": "62e139cbdf6946",
            "name": "API product Bella Test 1",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353216,
            "external_id": "62e0fd3e89b655",
            "name": "API product Bella Test 1",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353215,
            "external_id": "62de751d178f83",
            "name": "API product custom",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353214,
            "external_id": "62de40d6dfc335",
            "name": "API product custom",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353213,
            "external_id": "62de40c4a73325",
            "name": "API product custom",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353212,
            "external_id": "62de40b67313e3",
            "name": "API product custom",
            "variants": 1,
            "synced": 1,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353211,
            "external_id": "62de40ac1c6694",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353210,
            "external_id": "62dd5e06eff211",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353209,
            "external_id": "62dd5db6314647",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353208,
            "external_id": "62dd5d6ab15749",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353207,
            "external_id": "62dd5d29d70bd6",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353206,
            "external_id": "62dd5cba7b4dc2",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353205,
            "external_id": "62dd5c2d1df034",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": null,
            "is_ignored": false
        },
        {
            "id": 277353204,
            "external_id": "62dd5b3ca4f9a8",
            "name": "API product Bella Test 1",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://picsum.photos/200/300",
            "is_ignored": false
        },
        {
            "id": 277353203,
            "external_id": "62dd5ad498ea59",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": "https://s3-printful.stage.printful.dev/",
            "is_ignored": false
        },
        {
            "id": 277353202,
            "external_id": "62dd5a839ad745",
            "name": "API product custom",
            "variants": 2,
            "synced": 2,
            "thumbnail_url": null,
            "is_ignored": false
        }
    ],
    "extra": [],
    "paging": {
        "total": 194,
        "offset": 0,
        "limit": 20
    },
}
```

</details>

# Product Templates API

## Get Template List

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "items": [
            {
                "id": 24966053,
                "product_id": 328,
                "external_product_id": null,
                "title": "All-Over Print Men's Athletic T-shirt",
                "available_variant_ids": [
                    9952,
                    9958
                ],
                "option_data": [
                    {
                        "id": "stitch_color",
                        "value": "white"
                    },
                    {
                        "id": "text_thread_colors_front",
                        "value": [
                            "#000000"
                        ]
                    }
                ],
                "colors": [
                    {
                        "color_name": "White",
                        "color_codes": [
                            "#ffffff"
                        ]
                    }
                ],
                "sizes": [
                    "XS",
                    "S"
                ],
                "mockup_file": {
                    "imageURL": "https://example.com/upload/product-templates/d1/d1341a6efb49f59cc58172ce1c15eb20_l",
                    "thumbnailURL": "https://example.com/upload/product-templates/d1/d1341a6efb49f59cc58172ce1c15eb20_t",
                    "imageSVG": null,
                    "imageDownloadURL": null,
                    "status": "done",
                    "pusherKey": null
                },
                "created_at": 1644848286,
                "updated_at": 1644849302
            }
        ]
    },
    "extra": [],
    "paging": {
        "total": 2558,
        "offset": 0,
        "limit": 1
    },
    "debug": []
}
```
</details>

## Get Template Details

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 25029900,
        "product_id": 19,
        "external_product_id": null,
        "title": "White glossy mug",
        "available_variant_ids": [
            1320
        ],
        "option_data": [
            {
                "id": "text_thread_colors_default",
                "value": [
                    "#EAD6BB",
                    "#EAD6BB"
                ]
            }
        ],
        "colors": [
            {
                "color_name": "White",
                "color_codes": []
            }
        ],
        "sizes": [
            "11oz"
        ],
        "mockup_file": {
            "imageURL": "https://example.com/upload/product-templates/e8/e88de2eb2c0cc4933bcbc8866ef5ad47_l",
            "thumbnailURL": "https://example.com/upload/product-templates/e8/e88de2eb2c0cc4933bcbc8866ef5ad47_t",
            "imageSVG": null,
            "imageDownloadURL": null,
            "status": "done",
            "pusherKey": null
        },
        "created_at": 1645021359,
        "updated_at": 1645021359
    },
    "extra": [],
    "debug": []
}
```
</details>

# Orders API examples
## Using a catalog variant
Create an order containing an item, which shall be constructed on-the-fly and based on a print file and a `variant_id` from [Printful Catalog](https://developers.printful.com/docs#tag/Catalog-API).

Request body:
```
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
            "variant_id": 1,
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
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 21510,
        "external_id": null,
        "status": "draft",
        "shipping": "STANDARD",
        "created": 1390825082,
        "updated": 1390825082,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "items": [
            {
                "id": 17621,
                "external_id": null,
                "variant_id": 1,
                "quantity": 1,
                "price": "13.00",
                "retail_price": null,
                "name": "Unframed Poster 18×24",
                "product": {
                    "variant_id": 1,
                    "product_id": 1,
                    "image": "https://www.printful.com/storage/products/poster_18x24.jpg",
                    "name": "Unframed Poster 18×24"
                },
                "files": [
                    {
                        "id": 11818,
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
                        "created": 1390823900,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": []
            }
        ],
        "costs": {
            "subtotal": "13.00",
            "discount": "0.00",
            "shipping": "7.95",
            "tax": "1.17",
            "total": "22.12"
        },
        "retail_costs": {
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "total": null
        },
        "shipments": []
    }
}
```
</details>

## Using a sync variant
Create an order containing an item, which is based on a pre-configured sync variant from the authorized Printful store and is referenced by its sync variant ID. Please note, that the existing sync variant must be configured (synced) for this to work.

Request body:
```
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
            "sync_variant_id": 1,
            "quantity": 1
        }
    ]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 11360229,
        "external_id": null,
        "store": 952434,
        "status": "draft",
        "error": null,
        "shipping": "PRINTFUL_FAST",
        "created": 1539584164,
        "updated": 1539584164,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "notes": null,
        "items": [
            {
                "id": 7253102,
                "external_id": null,
                "variant_id": 4011,
                "sync_variant_id": 866913817,
                "external_variant_id": "5bc04fbe956147",
                "quantity": 1,
                "price": "12.95",
                "retail_price": null,
                "name": "Short-Sleeve Unisex T-Shirt - S",
                "product": {
                    "variant_id": 4011,
                    "product_id": 71,
                    "image": "https://www.printful.com/products/71/4012_1517927381.jpg",
                    "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / S)"
                },
                "files": [
                    {
                        "id": 76564071,
                        "type": "default",
                        "hash": "1c43709da13e3480049379d41e473ad9",
                        "url": null,
                        "filename": "printfile-23aeb205.png",
                        "mime_type": "image/png",
                        "size": 18660,
                        "width": 1800,
                        "height": 2400,
                        "dpi": 150,
                        "status": "ok",
                        "created": 1539329978,
                        "thumbnail_url": "https://www.printful.com/files/1c4/1c43709da13e3480049379d41e473ad9_thumb.png",
                        "preview_url": "https://www.printful.com/files/1c4/1c43709da13e3480049379d41e473ad9_preview.png",
                        "visible": false
                    }
                ],
                "options": [
                    {
                        "id": "embroidery_type",
                        "value": "flat"
                    },
                    {
                        "id": "thread_colors",
                        "value": []
                    },
                    {
                        "id": "thread_colors_3d",
                        "value": []
                    },
                    {
                        "id": "thread_colors_chest_left",
                        "value": []
                    }
                ],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false
            }
        ],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": true,
        "costs": {
            "currency": "USD",
            "subtotal": "12.95",
            "discount": "0.00",
            "shipping": "2.60",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "1.35",
            "vat": "0.00",
            "total": "16.90"
        },
        "retail_costs": {
            "currency": "USD",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard?order_id=11360229"
    },
    "extra": []
}
```
</details>

## Using sync variant with external ID
Create an order containing an item, which is based on a pre-configured sync variant from the authorized Printful store and is referenced by its external ID. Please note, that the existing Sync Variant must be configured (synced) for this to work.

Request body:
```
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
            "external_variant_id": "5bc04fbe956148",
            "quantity": 1
        }
    ]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 11360229,
        "external_id": null,
        "store": 952434,
        "status": "draft",
        "error": null,
        "shipping": "PRINTFUL_FAST",
        "created": 1539584164,
        "updated": 1539584164,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "notes": null,
        "items": [
            {
                "id": 7253102,
                "external_id": null,
                "variant_id": 4011,
                "sync_variant_id": 866913817,
                "external_variant_id": "5bc04fbe956147",
                "quantity": 1,
                "price": "12.95",
                "retail_price": null,
                "name": "Short-Sleeve Unisex T-Shirt - S",
                "product": {
                    "variant_id": 4011,
                    "product_id": 71,
                    "image": "https://www.printful.com/products/71/4012_1517927381.jpg",
                    "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (White / S)"
                },
                "files": [
                    {
                        "id": 76564071,
                        "type": "default",
                        "hash": "1c43709da13e3480049379d41e473ad9",
                        "url": null,
                        "filename": "printfile-23aeb205.png",
                        "mime_type": "image/png",
                        "size": 18660,
                        "width": 1800,
                        "height": 2400,
                        "dpi": 150,
                        "status": "ok",
                        "created": 1539329978,
                        "thumbnail_url": "https://www.printful.com/files/1c4/1c43709da13e3480049379d41e473ad9_thumb.png",
                        "preview_url": "https://www.printful.com/files/1c4/1c43709da13e3480049379d41e473ad9_preview.png",
                        "visible": false
                    }
                ],
                "options": [
                    {
                        "id": "embroidery_type",
                        "value": "flat"
                    },
                    {
                        "id": "thread_colors",
                        "value": []
                    },
                    {
                        "id": "thread_colors_3d",
                        "value": []
                    },
                    {
                        "id": "thread_colors_chest_left",
                        "value": []
                    }
                ],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false
            }
        ],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": true,
        "costs": {
            "currency": "USD",
            "subtotal": "12.95",
            "discount": "0.00",
            "shipping": "2.60",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "1.35",
            "vat": "0.00",
            "total": "16.90"
        },
        "retail_costs": {
            "currency": "USD",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard?order_id=11360229"
    }
}
```

</details>

## Using a product template

To create an order from a template, you need to know the IDs or External Product IDs of the product template(s) you want
to use. You can find them by calling `GET /product-templates` or in the address bar when viewing the template in
dashboard.

### Specifying one variant from a product template ID

Request body:

```
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
            "variant_id": 11325,
            "quantity": 1,
            "product_template_id": 123456789
        }
    ]
}
```

<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 83445633,
        "external_id": null,
        "store": 4485069,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (2-4 business days after fulfillment)",
        "created": 1667914568,
        "updated": 1667914568,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "Brīvības iela 1",
            "address2": null,
            "city": "Riga",
            "state_code": null,
            "state_name": null,
            "country_code": "LV",
            "country_name": "Latvia",
            "zip": "1010",
            "phone": null,
            "email": null,
            "tax_number": null
        },
        "notes": null,
        "items": [
            {
                "id": 56514363,
                "external_id": null,
                "variant_id": 11325,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "19.00",
                "retail_price": null,
                "name": "Men's Curved Hem T-Shirt | Bella + Canvas 3003 (Black / S)",
                "product": {
                    "variant_id": 11325,
                    "product_id": 415,
                    "image": "https://s3-printful.stage.printful.dev/products/415/11325_1591685336.jpg",
                    "name": "BELLA + CANVAS 3003 Fast Fashion Jersey Curved Hem Tee (Black / S)"
                },
                "files": [
                    {
                        "id": 487204563,
                        "type": "default",
                        "hash": "8cc82d028560643ca41df0315db64ec1",
                        "url": null,
                        "filename": "Motivational_posters_4_Won-with-it-(20)-(1).jpg",
                        "mime_type": "image/jpeg",
                        "size": 1727730,
                        "width": 7201,
                        "height": 9001,
                        "dpi": 300,
                        "status": "ok",
                        "created": 1667907555,
                        "thumbnail_url": "https://s3-printful.stage.printful.dev/files/8cc/8cc82d028560643ca41df0315db64ec1_thumb.png",
                        "preview_url": "https://s3-printful.stage.printful.dev/files/8cc/8cc82d028560643ca41df0315db64ec1_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487204565,
                        "type": "sleeve_right",
                        "hash": "ecf7982f2671ccfd2038a2228fda24dc",
                        "url": null,
                        "filename": "ed176bfb8272eac737256d6186ce403f.png",
                        "mime_type": "image/png",
                        "size": 4012,
                        "width": 600,
                        "height": 525,
                        "dpi": 150,
                        "status": "ok",
                        "created": 1667907619,
                        "thumbnail_url": "https://s3-printful.stage.printful.dev/printfile-preview/487204565/ed176bfb8272eac737256d6186ce403f_thumb.png",
                        "preview_url": "https://s3-printful.stage.printful.dev/printfile-preview/487204565/ed176bfb8272eac737256d6186ce403f_preview.png",
                        "visible": false,
                        "is_temporary": false
                    },
                    {
                        "id": 487208045,
                        "type": "preview",
                        "hash": null,
                        "url": null,
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1667914568,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 123456789
            }
        ],
        "branding_items": [],
        "incomplete_items": [],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "GBP",
            "subtotal": "19.00",
            "discount": "0.00",
            "shipping": "3.39",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "0.00",
            "vat": "4.71",
            "total": "27.10"
        },
        "retail_costs": {
            "currency": "GBP",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard?order_id=83445633"
    }
}
```

</details>

### Specifying multiple variants from a product template ID

Request body:

```
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
            "variant_id": 6882,
            "quantity": 1,
            "product_template_id": 111222333
        },
        {
            "variant_id": 1350,
            "quantity": 2,
            "product_template_id": 11235813
        },
        {
            "variant_id": 4398,
            "quantity": 1,
            "product_template_id": 11235813
        }
    ]
}
```

<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 83445637,
        "external_id": null,
        "store": 4485069,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (2-4 business days after fulfillment)",
        "created": 1667915130,
        "updated": 1667915130,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "Brīvības iela 1",
            "address2": null,
            "city": "Riga",
            "state_code": null,
            "state_name": null,
            "country_code": "LV",
            "country_name": "Latvia",
            "zip": "1010",
            "phone": null,
            "email": null,
            "tax_number": null
        },
        "notes": null,
        "items": [
            {
                "id": 56514369,
                "external_id": null,
                "variant_id": 6882,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "18.75",
                "retail_price": null,
                "name": "Premium Luster Photo Paper Framed Poster (in) (Black / 8″×10″)",
                "product": {
                    "variant_id": 6882,
                    "product_id": 172,
                    "image": "https://s3-printful.stage.printful.dev/products/172/6882_1527683092.jpg",
                    "name": "Premium Luster Photo Paper Framed Poster (Black/8″×10″)"
                },
                "files": [
                    {
                        "id": 487203842,
                        "type": "default",
                        "hash": "8cc82d028560643ca41df0315db64ec1",
                        "url": null,
                        "filename": "Motivational_posters_4_Won-with-it-(10)-(6)-(19).jpg",
                        "mime_type": "image/jpeg",
                        "size": 1727730,
                        "width": 7201,
                        "height": 9001,
                        "dpi": 300,
                        "status": "ok",
                        "created": 1667906016,
                        "thumbnail_url": "https://s3-printful.stage.printful.dev/files/8cc/8cc82d028560643ca41df0315db64ec1_thumb.png",
                        "preview_url": "https://s3-printful.stage.printful.dev/files/8cc/8cc82d028560643ca41df0315db64ec1_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487208085,
                        "type": "preview",
                        "hash": null,
                        "url": null,
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1667915129,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 111222333
            },
            {
                "id": 56514370,
                "external_id": null,
                "variant_id": 1350,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 2,
                "price": "23.50",
                "retail_price": null,
                "name": "Enhanced Matte Paper Framed Poster (in) (Black / 12″×16″)",
                "product": {
                    "variant_id": 1350,
                    "product_id": 2,
                    "image": "https://s3-printful.stage.printful.dev/products/2/1350_1527683296.jpg",
                    "name": "Enhanced Matte Paper Framed Poster (Black/12″×16″)"
                },
                "files": [
                    {
                        "id": 487207998,
                        "type": "default",
                        "hash": "d4024345dec7d649058a50948935ad05",
                        "url": null,
                        "filename": "d81aec33ca3a7ebf24c4ada1647e3ed0-(1).png",
                        "mime_type": "image/png",
                        "size": 773995,
                        "width": 1650,
                        "height": 1650,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667911881,
                        "thumbnail_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487208083,
                        "type": "preview",
                        "hash": null,
                        "url": null,
                        "filename": "enhanced-matte-paper-framed-poster-(in)-black-12x16-transparent-636a5d67f10a6.png",
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1667915109,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 11235813
            },
            {
                "id": 56514371,
                "external_id": null,
                "variant_id": 4398,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "24.75",
                "retail_price": null,
                "name": "Enhanced Matte Paper Framed Poster (in) (Black / 12″×18″)",
                "product": {
                    "variant_id": 4398,
                    "product_id": 2,
                    "image": "https://s3.staging.printful.com/products/2/4398_1527683145.jpg",
                    "name": "Enhanced Matte Paper Framed Poster (Black/12″×18″)"
                },
                "files": [
                    {
                        "id": 487207998,
                        "type": "default",
                        "hash": "d4024345dec7d649058a50948935ad05",
                        "url": null,
                        "filename": "d81aec33ca3a7ebf24c4ada1647e3ed0-(1).png",
                        "mime_type": "image/png",
                        "size": 773995,
                        "width": 1650,
                        "height": 1650,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667911881,
                        "thumbnail_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487208084,
                        "type": "preview",
                        "hash": null,
                        "url": null,
                        "filename": "enhanced-matte-paper-framed-poster-(in)-black-12x18-transparent-636a5d690c2c8.png",
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1667915109,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 11235813
            }
        ],
        "branding_items": [],
        "incomplete_items": [],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "GBP",
            "subtotal": "90.50",
            "discount": "0.00",
            "shipping": "16.54",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "0.00",
            "vat": "22.49",
            "total": "129.53"
        },
        "retail_costs": {
            "currency": "GBP",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard?order_id=83445637"
    }
}
```

</details>

### Using External Product ID

The Product Template may be specified by the associated External Product ID. You can mix it with Product Template IDs
for different items but for one item, only one of the identifier fields must be present.

The External Product ID is used only to specify the template. The Item will be associated with the template and only its
ID will be present in the response.

Request body:

```
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
            "variant_id": 6882,
            "quantity": 1,
            "product_template_id": 111222333
        },
        {
            "variant_id": 1350,
            "quantity": 2,
            "external_product_id": "fibonacci"
        },
        {
            "variant_id": 4398,
            "quantity": 1,
            "external_product_id": "fibonacci"
        }
    ]
}
```

<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 83445641,
        "external_id": null,
        "store": 4485069,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (2-4 business days after fulfillment)",
        "created": 1667916052,
        "updated": 1667916052,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "Brīvības iela 1",
            "address2": null,
            "city": "Riga",
            "state_code": null,
            "state_name": null,
            "country_code": "LV",
            "country_name": "Latvia",
            "zip": "1010",
            "phone": null,
            "email": null,
            "tax_number": null
        },
        "notes": null,
        "items": [
            {
                "id": 56514375,
                "external_id": null,
                "variant_id": 6882,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "18.75",
                "retail_price": null,
                "name": "Premium Luster Photo Paper Framed Poster (in) (Black / 8″×10″)",
                "product": {
                    "variant_id": 6882,
                    "product_id": 172,
                    "image": "https://s3.staging.printful.com/products/172/6882_1527683092.jpg",
                    "name": "Premium Luster Photo Paper Framed Poster (Black/8″×10″)"
                },
                "files": [
                    {
                        "id": 487203842,
                        "type": "default",
                        "hash": "8cc82d028560643ca41df0315db64ec1",
                        "url": null,
                        "filename": "Motivational_posters_4_Won-with-it-(10)-(6)-(19).jpg",
                        "mime_type": "image/jpeg",
                        "size": 1727730,
                        "width": 7201,
                        "height": 9001,
                        "dpi": 300,
                        "status": "ok",
                        "created": 1667906016,
                        "thumbnail_url": "https://s3.staging.printful.com/files/8cc/8cc82d028560643ca41df0315db64ec1_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/8cc/8cc82d028560643ca41df0315db64ec1_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487208085,
                        "type": "preview",
                        "hash": "b0c646ddb59f66d6d043021d0f7c4dd5",
                        "url": null,
                        "filename": "premium-luster-photo-paper-framed-poster-(in)-black-8x10-transparent-636a5d7e97726.png",
                        "mime_type": "image/png",
                        "size": 169640,
                        "width": 1000,
                        "height": 1000,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667915129,
                        "thumbnail_url": "https://s3.staging.printful.com/files/b0c/b0c646ddb59f66d6d043021d0f7c4dd5_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/b0c/b0c646ddb59f66d6d043021d0f7c4dd5_preview.png",
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 111222333
            },
            {
                "id": 56514376,
                "external_id": null,
                "variant_id": 1350,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 2,
                "price": "23.50",
                "retail_price": null,
                "name": "Enhanced Matte Paper Framed Poster (in) (Black / 12″×16″)",
                "product": {
                    "variant_id": 1350,
                    "product_id": 2,
                    "image": "https://s3.staging.printful.com/products/2/1350_1527683296.jpg",
                    "name": "Enhanced Matte Paper Framed Poster (Black/12″×16″)"
                },
                "files": [
                    {
                        "id": 487207998,
                        "type": "default",
                        "hash": "d4024345dec7d649058a50948935ad05",
                        "url": null,
                        "filename": "d81aec33ca3a7ebf24c4ada1647e3ed0-(1).png",
                        "mime_type": "image/png",
                        "size": 773995,
                        "width": 1650,
                        "height": 1650,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667911881,
                        "thumbnail_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487208083,
                        "type": "preview",
                        "hash": "74563a236db28cccc22f3a5554ede2fa",
                        "url": null,
                        "filename": "enhanced-matte-paper-framed-poster-(in)-black-12x16-transparent-636a5d67f10a6.png",
                        "mime_type": "image/png",
                        "size": 274306,
                        "width": 1000,
                        "height": 1000,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667915109,
                        "thumbnail_url": "https://s3.staging.printful.com/files/745/74563a236db28cccc22f3a5554ede2fa_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/745/74563a236db28cccc22f3a5554ede2fa_preview.png",
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 11235813
            },
            {
                "id": 56514377,
                "external_id": null,
                "variant_id": 4398,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "24.75",
                "retail_price": null,
                "name": "Enhanced Matte Paper Framed Poster (in) (Black / 12″×18″)",
                "product": {
                    "variant_id": 4398,
                    "product_id": 2,
                    "image": "https://s3.staging.printful.com/products/2/4398_1527683145.jpg",
                    "name": "Enhanced Matte Paper Framed Poster (Black/12″×18″)"
                },
                "files": [
                    {
                        "id": 487207998,
                        "type": "default",
                        "hash": "d4024345dec7d649058a50948935ad05",
                        "url": null,
                        "filename": "d81aec33ca3a7ebf24c4ada1647e3ed0-(1).png",
                        "mime_type": "image/png",
                        "size": 773995,
                        "width": 1650,
                        "height": 1650,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667911881,
                        "thumbnail_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/d40/d4024345dec7d649058a50948935ad05_preview.png",
                        "visible": true,
                        "is_temporary": false
                    },
                    {
                        "id": 487208084,
                        "type": "preview",
                        "hash": "d1021840b0e9514038e9440edbfef4ec",
                        "url": null,
                        "filename": "enhanced-matte-paper-framed-poster-(in)-black-12x18-transparent-636a5d690c2c8.png",
                        "mime_type": "image/png",
                        "size": 277513,
                        "width": 1000,
                        "height": 1000,
                        "dpi": null,
                        "status": "ok",
                        "created": 1667915109,
                        "thumbnail_url": "https://s3.staging.printful.com/files/d10/d1021840b0e9514038e9440edbfef4ec_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/d10/d1021840b0e9514038e9440edbfef4ec_preview.png",
                        "visible": false,
                        "is_temporary": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock": false,
                "product_template_id": 11235813
            }
        ],
        "branding_items": [],
        "incomplete_items": [],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "GBP",
            "subtotal": "90.50",
            "discount": "0.00",
            "shipping": "16.54",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "0.00",
            "vat": "22.49",
            "total": "129.53"
        },
        "retail_costs": {
            "currency": "GBP",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard?order_id=83445641"
    }
}
```

</details>

## Using customized retail prices

Create an order with external order ID, custom item names, and retail price information. Order with external ID and
customized item names and prices.

Request body:

```
{
    "external_id": "9887112",
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
            "variant_id": 2,
            "quantity": 1,
            "name": "Niagara Falls poster",
            "retail_price": "19.99",
            "files": [
                {
                    "url": "http://example.com/files/posters/poster_2.jpg"
                }
            ]
        },
        {
            "variant_id": 1,
            "quantity": 3,
            "name": "Grand Canyon poster",
            "retail_price": "17.99",
            "files": [
            {
                "url": "http://example.com/files/posters/poster_3.jpg"
            }
            ]
        }
    ],
    "retail_costs": {
        "shipping": "24.50"
    }
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 21509,
        "external_id": "9887112",
        "status": "draft",
        "shipping": "STANDARD",
        "created": 1390825006,
        "updated": 1390825006,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "items": [
            {
                "id": 17619,
                "external_id": null,
                "variant_id": 2,
                "quantity": 1,
                "price": "18.00",
                "retail_price": "19.99",
                "name": "Niagara Falls poster",
                "product": {
                    "variant_id": 2,
                    "product_id": 1,
                    "image": "https://www.printful.com/storage/products/poster_24x36.jpg",
                    "name": "Unframed Poster 24×36"
                },
                "files": [
                    {
                        "id": 11819,
                        "type": "default",
                        "hash": null,
                        "url": "http://example.com/files/posters/poster_2.jpg",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1390824712,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": []
            },
            {
                "id": 17620,
                "external_id": null,
                "variant_id": 1,
                "quantity": 3,
                "price": "13.00",
                "retail_price": "17.99",
                "name": "Grand Canyon poster",
                "product": {
                    "variant_id": 1,
                    "product_id": 1,
                    "image": "https://www.printful.com/storage/products/poster_18x24.jpg",
                    "name": "Unframed Poster 18×24"
                },
                "files": [
                    {
                        "id": 11820,
                        "type": "default",
                        "hash": null,
                        "url": "http://example.com/files/posters/poster_3.jpg",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1390824712,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": []
            }
        ],
        "costs": {
            "subtotal": "57.00",
            "discount": "0.00",
            "shipping": "9.95",
            "tax": "5.13",
            "total": "72.08"
        },
        "retail_costs": {
            "subtotal": "73.96",
            "discount": "0.00",
            "shipping": "24.50",
            "tax": "0.00",
            "total": "98.46"
        },
        "shipments": []
    }
}
```
</details>

## Using multiple print placements
Create an order from a catalog variant that uses both front and back print files specified.

Request body:
```
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
            "variant_id": 1118,
            "quantity": 1,
            "files": [
                {
                    "type": "front",
                    "url": "http://example.com/files/tshirts/shirt_front.png"
                },
                {
                    "type": "back",
                    "url": "http://example.com/files/tshirts/shirt_back.png"
                }
            ],
            "options": []
        }
    ]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 21516,
        "external_id": null,
        "status": "draft",
        "shipping": "STANDARD",
        "created": 1390825437,
        "updated": 1390825437,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "items": [
            {
                "id": 17627,
                "external_id": null,
                "variant_id": 1118,
                "quantity": 1,
                "price": "20.50",
                "retail_price": null,
                "name": "Alternative 1070 Short Sleeve Men T-Shirt (Black / M)",
                "product": {
                    "variant_id": 1118,
                    "product_id": 14,
                    "image": "https://www.printful.com/storage/products/14/1095.jpg",
                    "name": "Alternative 1070 Short Sleeve Men T-Shirt (Black / M)"
                },
                "files": [
                    {
                        "id": 11822,
                        "type": "default",
                        "hash": null,
                        "url": "http://example.com/files/tshirts/shirt_front.png",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1390825349,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    },
                    {
                        "id": 11823,
                        "type": "back",
                        "hash": null,
                        "url": "http://example.com/files/tshirts/shirt_back.png",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1390825349,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": []
            }
        ],
        "costs": {
            "subtotal": "20.50",
            "discount": "0.00",
            "shipping": "5.50",
            "tax": "1.85",
            "total": "27.85"
        },
        "retail_costs": {
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "total": null
        },
        "shipments": []
    }
}
```
</details>

## Using all-over products
Create an order containing a pair of leggings with the "Stitch color" option specified.

Request body:
```
{
    "recipient": {
        "name": "John Doe",
        "address1": "19749 Dearborn St",
        "city": "Chatsworth",
        "state_code": "CA",
        "country_code": "US",
        "zip": "91311"
    },
    "items": [{
        "variant_id": 7983,
        "quantity": 1,
        "name": "Leggings",
        "retail_price": "19.99",
        "files": [{
            "url": "https://example.com/id/858/2000/2009.jpg"
        },
        {
        "url": "https://example.com/id/858/2000/2009.jpg",
        "type": "label_inside"
        }
        ],
        "options": [{"id":"stitch_color", "value": "white"}]
    }]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 22481169,
        "external_id": null,
        "store": 1633247,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (3-4 business days after fulfillment)",
        "created": 1575645857,
        "updated": 1575645857,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "notes": null,
        "items": [
            {
                "id": 13516941,
                "external_id": null,
                "variant_id": 9144,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "26.99",
                "retail_price": null,
                "name": "All-Over Print Men's Leggings (XS)",
                "product": {
                    "variant_id": 9144,
                    "product_id": 288,
                    "image": "https://www.printful.com/products/288/9144_1532344185.jpg",
                    "name": "All-Over Print Men's Leggings (XS)"
                },
                "files": [
                    {
                        "id": 138304686,
                        "type": "default",
                        "hash": "c27a71ab4008c83eba9b554775aa12ca",
                        "url": "http://example.com/files/leggings/leggings_mockup.jpg",
                        "filename": "QmTQo4cxDZ5MoszQAK93JyhFedeMuj7j4x5P7tQnvRi4A5.png",
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1575645853,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": [
                    {
                        "id": "stitch_color",
                        "value": "black"
                    }
                ],
                "sku": null,
                "discontinued": false,
                "out_of_stock_eu": false,
                "out_of_stock": false
            }
        ],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "EUR",
            "subtotal": "26.99",
            "discount": "0.00",
            "shipping": "3.65",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "2.82",
            "vat": "0.00",
            "total": "33.46"
        },
        "retail_costs": {
            "currency": "EUR",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard/default/orders?order_id=22481169"
    },
    "extra": []
}
```
</details>

## Using embroidery products
Create an order containing a hat with the embroidery type and thread colors specified.

Request body:
```
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
            "variant_id": 8746,
            "quantity": 1,
            "files": [
                {
                    "url": "http://example.com/files/hats/hats_mockup.jpg"
                }
            ],
            "options": [
                {
                    "id": "embroidery_type",
                    "value": "flat"
                },
                {
                    "id": "thread_colors",
                    "value": ["#FFFFFF", "#A67843"]
                }
            ]
        }
    ]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 22481172,
        "external_id": null,
        "store": 1387102,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (3-4 business days after fulfillment)",
        "created": 1575647137,
        "updated": 1575647137,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "notes": null,
        "items": [
            {
                "id": 13516944,
                "external_id": null,
                "variant_id": 8746,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "12.57",
                "retail_price": null,
                "name": "Yupoong 6606 Retro Trucker Cap (White)",
                "product": {
                    "variant_id": 8746,
                    "product_id": 252,
                    "image": "https://www.printful.com/products/252/8746_1543592826.jpg",
                    "name": "Yupoong 6606 Retro Trucker Cap (White)"
                },
                "files": [
                    {
                        "id": 138304687,
                        "type": "default",
                        "hash": "c27a71ab4008c83eba9b554775aa12ca",
                        "url": "http://example.com/files/hats/hats_mockup.jpg",
                        "filename": "QmTQo4cxDZ5MoszQAK93JyhFedeMuj7j4x5P7tQnvRi4A5.png",
                        "mime_type": "image/png",
                        "size": 37257,
                        "width": 2480,
                        "height": 2480,
                        "dpi": 300,
                        "status": "ok",
                        "created": 1575646661,
                        "thumbnail_url": "https://www.printful.com/files/c27/c27a71ab4008c83eba9b554775aa12ca_thumb.png",
                        "preview_url": "https://www.printful.com/files/c27/c27a71ab4008c83eba9b554775aa12ca_preview.png",
                        "visible": true
                    }
                ],
                "options": [
                    {
                        "id": "embroidery_type",
                        "value": "flat"
                    },
                    {
                        "id": "thread_colors",
                        "value": [
                            "#FFFFFF",
                            "#A67843"
                        ]
                    },
                    {
                        "id": "text_thread_colors",
                        "value": []
                    },
                    {
                        "id": "thread_colors_3d",
                        "value": []
                    }
                ],
                "sku": null,
                "discontinued": false,
                "out_of_stock_eu": false,
                "out_of_stock": false
            }
        ],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "EUR",
            "subtotal": "12.57",
            "discount": "0.00",
            "shipping": "3.65",
            "digitization": "5.75",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "1.91",
            "vat": "0.00",
            "total": "23.88"
        },
        "retail_costs": {
            "currency": "EUR",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.com/dashboard/default/orders?order_id=22481172"
    },
    "extra": []
}
```
</details>

## Using full color option

Request body:
```
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
            "variant_id": 8746,
            "quantity": 1,
            "files": [
                {
                    "url": "http://example.com/files/hats/hats_mockup.jpg",
                    "type": "embroidery_front",
                    "options": [
                        {
                            "id": "full_color",
                            "value": true
                        }
                    ]
                }
            ]
        }
    ]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 77329346,
        "external_id": null,
        "store": 7158238,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (3-4 business days after fulfillment)",
        "created": 1657290197,
        "updated": 1657290197,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null,
            "tax_number": null
        },
        "notes": null,
        "items": [
            {
                "id": 51123823,
                "external_id": null,
                "variant_id": 8746,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "18.50",
                "retail_price": null,
                "name": "Retro Trucker Hat | Yupoong 6606 (White)",
                "product": {
                    "variant_id": 8746,
                    "product_id": 252,
                    "image": "https://s3.staging.printful.com/products/252/8746_1585041861.jpg",
                    "name": "Yupoong 6606 Retro Trucker Cap (White)"
                },
                "files": [
                    {
                        "id": 438932922,
                        "type": "default",
                        "hash": null,
                        "url": "http://example.com/files/hats/hats_mockup.jpg",
                        "filename": "hats_mockup.jpg",
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "failed",
                        "created": 1657290187,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": false
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock_eu": false,
                "out_of_stock": false
            }
        ],
        "branding_items": [],
        "incomplete_items": [],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "CAD",
            "subtotal": "18.50",
            "discount": "0.00",
            "shipping": "4.99",
            "digitization": "9.60",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "2.93",
            "vat": "0.00",
            "total": "36.02"
        },
        "retail_costs": {
            "currency": "CAD",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.test/dashboard?order_id=77329346"
    },
}
```
</details>

## T-shirt with an inside label
Create an order containing a t-shirt with a native inside label. Please note that the inside label type must be specified in the file options.

Request body:
```
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
            "id": 16303890,
            "variant_id": 6950,
            "quantity": 1,
            "files": [
                {
                    "type": "front",
                    "url": "http://www.example.com/media/prints/38/large.jpg"
                },
                {
                    "type": "label_inside",
                    "url": "http://www.example.com/media/image_123.jpg",
                    "options": [
                        {
                            "id": "template_type",
                            "value": "native"
                        }
                    ]
                }
            ]
        }
    ]
}
```
<details>
  <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 30931362,
        "external_id": null,
        "store": 493,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (3-4 business days after fulfillment)",
        "created": 1586183759,
        "updated": 1586183759,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null
        },
        "notes": null,
        "items": [
            {
                "id": 17915289,
                "external_id": null,
                "variant_id": 6950,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "20.51",
                "retail_price": null,
                "name": "Unisex Premium T-Shirt | Bella + Canvas 3001 (Athletic Heather / L)",
                "product": {
                    "variant_id": 6950,
                    "product_id": 71,
                    "image": "https://s3.dev.printful.com/products/71/6950_1581408454.jpg",
                    "name": "Bella + Canvas 3001 Unisex Short Sleeve Jersey T-Shirt with Tear Away Label (Athletic Heather / L)"
                },
                "files": [
                    {
                        "id": 183409147,
                        "type": "default",
                        "hash": null,
                        "url": "http://www.example.com/media/prints/38/large.jpg",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1586183758,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    },
                    {
                        "id": 183409148,
                        "type": "label_inside",
                        "hash": null,
                        "url": "http://www.example.com/media/image_123.jpg",
                        "filename": null,
                        "mime_type": null,
                        "size": 0,
                        "width": null,
                        "height": null,
                        "dpi": null,
                        "status": "waiting",
                        "created": 1586183758,
                        "thumbnail_url": null,
                        "preview_url": null,
                        "visible": true
                    }
                ],
                "options": [],
                "sku": null,
                "discontinued": false,
                "out_of_stock_eu": false,
                "out_of_stock": false
            }
        ],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "CAD",
            "subtotal": "20.51",
            "discount": "0.00",
            "shipping": "5.70",
            "digitization": "0.00",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "2.15",
            "vat": "0.00",
            "total": "28.36"
        },
        "retail_costs": {
            "currency": "CAD",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.test/dashboard?order_id=30931362"
    },
    "extra": [],
}
```
</details>

## Embroidery patch
Create an order containing the embroidery patch.

Request body:

```
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
            "variant_id": 12983,
            "quantity": 1,
            "files": [
                {
                    "url": "https://pbs.twimg.com/profile_images/1073247505381552129/53OmqmtE_400x400.jpg",
                    "type": "embroidery_patch_front"
                }
            ],
            "options": [
                {
                    "id": "thread_colors_outline",
                    "value": [
                        "#A67843"
                    ]
                },
                {
                    "id": "thread_colors_patch_front",
                    "value": [
                        "#96A1A8",
                        "#3399FF",
                        "#E25C27",
                        "#CC3333",
                        "#333366",
                        "#000000"
                    ]
                }
            ]
        }
    ]
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "id": 77329716,
        "external_id": null,
        "store": 7158238,
        "status": "draft",
        "error": null,
        "errorCode": 0,
        "shipping": "STANDARD",
        "shipping_service_name": "Flat Rate (3-4 business days after fulfillment)",
        "created": 1658333148,
        "updated": 1658333148,
        "recipient": {
            "name": "John Doe",
            "company": null,
            "address1": "19749 Dearborn St",
            "address2": null,
            "city": "Chatsworth",
            "state_code": "CA",
            "state_name": "California",
            "country_code": "US",
            "country_name": "United States",
            "zip": "91311",
            "phone": null,
            "email": null,
            "tax_number": null
        },
        "notes": null,
        "items": [
            {
                "id": 51124091,
                "external_id": null,
                "variant_id": 12983,
                "sync_variant_id": null,
                "external_variant_id": null,
                "quantity": 1,
                "price": "11.50",
                "retail_price": null,
                "name": "Embroidered Patches (White / Circular ⌀3 in)",
                "product": {
                    "variant_id": 12983,
                    "product_id": 516,
                    "image": "https://s3.staging.printful.com/products/516/12983_1631709382.jpg",
                    "name": "Embroidered Patches (White / Circular ⌀3 in)"
                },
                "files": [
                    {
                        "id": 438932260,
                        "type": "embroidery_patch_front",
                        "hash": "9740034e67529e49e4db9329c733b0ea",
                        "url": "https://pbs.twimg.com/profile_images/1073247505381552129/53OmqmtE_400x400.jpg",
                        "filename": "53OmqmtE_400x400.jpg",
                        "mime_type": "image/jpeg",
                        "size": 10045,
                        "width": 400,
                        "height": 400,
                        "dpi": null,
                        "status": "ok",
                        "created": 1655901818,
                        "thumbnail_url": "https://s3.staging.printful.com/files/974/9740034e67529e49e4db9329c733b0ea_thumb.png",
                        "preview_url": "https://s3.staging.printful.com/files/974/9740034e67529e49e4db9329c733b0ea_preview.png",
                        "visible": true
                    }
                ],
                "options": [
                    {
                        "id": "embroidery_type",
                        "value": "flat"
                    },
                    {
                        "id": "thread_colors",
                        "value": [
                            "#96A1A8",
                            "#3399FF",
                            "#E25C27",
                            "#CC3333",
                            "#333366",
                            "#000000"
                        ]
                    },
                    {
                        "id": "text_thread_colors",
                        "value": []
                    },
                    {
                        "id": "thread_colors_3d",
                        "value": []
                    },
                    {
                        "id": "thread_colors_patch_front",
                        "value": []
                    },
                    {
                        "id": "text_thread_colors_patch_front",
                        "value": []
                    },
                    {
                        "id": "thread_colors_outline",
                        "value": [
                            "#A67843"
                        ]
                    }
                ],
                "sku": null,
                "discontinued": false,
                "out_of_stock_eu": false,
                "out_of_stock": false
            }
        ],
        "branding_items": [],
        "incomplete_items": [],
        "is_sample": false,
        "needs_approval": false,
        "not_synced": false,
        "has_discontinued_items": false,
        "can_change_hold": false,
        "costs": {
            "currency": "CAD",
            "subtotal": "11.50",
            "discount": "0.00",
            "shipping": "5.09",
            "digitization": "9.60",
            "additional_fee": "0.00",
            "fulfillment_fee": "0.00",
            "tax": "2.20",
            "vat": "0.00",
            "total": "28.39"
        },
        "retail_costs": {
            "currency": "CAD",
            "subtotal": null,
            "discount": null,
            "shipping": null,
            "tax": null,
            "vat": null,
            "total": null
        },
        "shipments": [],
        "gift": null,
        "packing_slip": null,
        "dashboard_url": "https://www.printful.test/dashboard?order_id=77329716"
    },
}
```

</details>

## Using Knitwear Product

In v1 of the API the `color_reduction_mode` is not available to be specified
and it will always default to `solid`.

Create an order with an order for a knitwear product.
```
{
  "recipient": {
    "name": "john smith",
    "company": "john smith inc",
    "address1": "19749 dearborn st",
    "city": "chatsworth",
    "state_code": "ca",
    "country_code": "us",
    "country_name": "united states",
    "zip": "91311"
  },
  "items": [
    {
      "variant_id": 19633,
      "quantity": 1,
      "files": [
        {
          "type": "front",
          "url": "https://picsum.photos/200"
        }
      ],
      "options": [
        {
          "id": "yarn_colors",
          "value": [
            "#090909",
            "#48542e"
          ]
        },
        {
          "id": "trim_color",
          "value": "#dcd3cc"
        },
        {
          "id": "base_color",
          "value": "#dda032"
        }
      ]
    }
  ]
}
```


<details>
    <summary>Response data</summary>

```
{
  "code": 200,
  "result": {
    "id": 108346939,
    "external_id": null,
    "store": 2036805,
    "status": "draft",
    "error": null,
    "errorCode": 0,
    "shipping": "STANDARD",
    "shipping_service_name": "Flat Rate (3-4 business days after fulfillment)",
    "created": 1722332904,
    "updated": 1722332904,
    "recipient": {
      "name": "John Smith",
      "company": "John Smith Inc",
      "address1": "19749 Dearborn St",
      "address2": null,
      "city": "Chatsworth",
      "state_code": "CA",
      "state_name": "California",
      "country_code": "US",
      "country_name": "United States",
      "zip": "91311",
      "phone": null,
      "email": null,
      "tax_number": null
    },
    "notes": null,
    "incomplete_items": [],
    "is_sample": false,
    "needs_approval": false,
    "not_synced": false,
    "has_discontinued_items": false,
    "can_change_hold": false,
    "costs": {
      "currency": "AUD",
      "subtotal": 65.5,
      "discount": 0,
      "shipping": 0,
      "digitization": 0,
      "additional_fee": 0,
      "fulfillment_fee": 0,
      "retail_delivery_fee": "0.00",
      "tax": "6.84",
      "vat": 0,
      "total": 72.34
    },
    "dashboard_url": "https://www.printful.test/dashboard?order_id=108346939",
    "items": [
      {
        "id": 85614675,
        "external_id": null,
        "variant_id": 19633,
        "sync_variant_id": null,
        "external_variant_id": null,
        "quantity": 1,
        "price": 65.5,
        "retail_price": null,
        "name": "Classic Fit Knitted Crew Neck Sweater (Custom / 3XS)",
        "product": {
          "variant_id": 19633,
          "product_id": 769,
          "image": "https://s3.staging.printful.com/products/769/19633_1716883244.jpg",
          "name": "Classic Fit Knitted Crew Neck Sweater (Custom / 3XS)"
        },
        "files": [
          {
            "id": 165809524,
            "type": "default",
            "hash": "2cfa55e156067d0ef050343f80f81e1d",
            "url": "https://picsum.photos/200",
            "filename": "165809524.jpg",
            "mime_type": "image/jpeg",
            "size": 9289,
            "width": 200,
            "height": 200,
            "dpi": 72,
            "status": "ok",
            "created": 1579677111,
            "thumbnail_url": "https://s3.staging.printful.com/files/2cf/2cfa55e156067d0ef050343f80f81e1d_thumb.png",
            "preview_url": "https://s3.staging.printful.com/files/2cf/2cfa55e156067d0ef050343f80f81e1d_preview.png",
            "visible": true,
            "is_temporary": false,
            "message": ""
          }
        ],
        "options": [
          {
            "id": "base_color",
            "value": "#dda032"
          },
          {
            "id": "yarn_colors",
            "value": [ "#090909", "#48542e" ]
          },
          {
            "id": "trim_color",
            "value": "#dcd3cc"
          }
        ],
        "sku": null,
        "discontinued": false,
        "out_of_stock": false
      }
    ],
    "branding_items": [],
    "shipments": [],
    "packing_slip": null,
    "retail_costs": {
      "currency": "AUD",
      "subtotal": null,
      "discount": null,
      "shipping": null,
      "tax": null,
      "vat": null,
      "total": null
    },
    "gift": null
  },
  "extra": [],
  "debug": []
}
```

</details>

The resulting mockup should look as follows:
<br>
![Image](images/mockups/knitwear_example.png)

# File Library API examples

## Add a new file

Add file to the print file library, file name will be detected from URL. After creation file is not processed instantly.

Request body:

```
{
    "url": "http://www.example.com/files/tshirts/example.png"
}
```

Response data:

```
{
    "code": 200,
    "result": {
        "id": 11816,
        "type": "default",
        "hash": null,
        "url": "http://www.example.com/files/tshirts/example.png",
        "filename": null,
        "mime_type": null,
        "size": 0,
        "width": null,
        "height": null,
        "dpi": null,
        "status": "waiting",
        "created": 1390819101,
        "thumbnail_url": null,
        "preview_url": null,
        "visible": true
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
    "visible": 0
}
```

## Suggest thread colors

Get suggested thread colors for provided image URL

Request body:

```
{
    "file_url": "http://www.example.com/files/tshirts/example.png"
}
```

Response data:

```
{
    "code": 200,
    "result": {
        "thread_colors": [
            "#FFFFFF",
            "#000000",
            "#96A1A8",
            "#CC3333",
            "#E25C27"
        ]
    },
}
```

# Ecommerce Platform Sync API examples

## Modify a Sync Variant

Links Sync Variant with T-shirt with front and back images.

Request body:

```
{
    "variant_id": 1118,
    "files": [
        {
            "url": "http://example.com/files/tshirts/shirt_front.png"
        },
        {
            "type": "back",
            "url": "http://example.com/files/tshirts/shirt_back.png"
        }
    ],
    "options": []
}
```

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "sync_variant": {
            "id": 4699529,
            "external_id": "1117068664",
            "sync_product_id": 1055835,
            "name": "Michael Jackson T-Shirt - Medium",
            "synced": true,
            "variant_id": 1118,
            "product": {
                "variant_id": 1118,
                "product_id": 14,
                "image": "https://files.cdn.printful.com/products/14/1095.jpg",
                "name": "Alternative 1070 Short Sleeve Men T-Shirt (Black / M)"
            },
            "files": [
                {
                    "id": 304995,
                    "type": "default",
                    "hash": null,
                    "url": "http://example.com/files/tshirts/shirt_front.png",
                    "filename": null,
                    "mime_type": null,
                    "size": 0,
                    "width": null,
                    "height": null,
                    "dpi": null,
                    "status": "waiting",
                    "created": 1426839420,
                    "thumbnail_url": null,
                    "preview_url": null,
                    "visible": true
                },
                {
                    "id": 304996,
                    "type": "back",
                    "hash": null,
                    "url": "http://example.com/files/tshirts/shirt_back.png",
                    "filename": null,
                    "mime_type": null,
                    "size": 0,
                    "width": null,
                    "height": null,
                    "dpi": null,
                    "status": "waiting",
                    "created": 1426839420,
                    "thumbnail_url": null,
                    "preview_url": null,
                    "visible": true
                }
            ],
            "options": []
        },
        "sync_product": {
            "id": 1055835,
            "external_id": "409040684",
            "name": "Michael Jackson T-Shirt",
            "variants": 1,
            "synced": 1
        }
    }
}
```

</details>

# Mockup Generator API examples

## Retrieve product variant print files with default technique

URI: `GET /mockup-generator/printfiles/162`

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "product_id": 162,
        "available_placements": {
            "front": "Front print",
            "back": "Back print",
            "label_outside": "Outside label",
            "label_inside": "Inside label",
            "sleeve_left": "Left sleeve",
            "sleeve_right": "Right sleeve"
        },
        "printfiles": [
            {
                "printfile_id": 1,
                "width": 1800,
                "height": 2400,
                "dpi": 150,
                "fill_mode": "fit",
                "can_rotate": false
            },
            ...
        ],
        "variant_printfiles": [
            {
                "variant_id": 6584,
                "placements": {
                    "front": 1,
                    "back": 1,
                    "label_outside": 90,
                    "label_inside": 71,
                    "sleeve_left": 130,
                    "sleeve_right": 130
                }
            },
            ...
        ],
        "option_groups": [
            "Couple's",
            "Flat",
            "Flat 2",
            "Flat 3",
            "Flat Lifestyle",
            "Holiday season",
            "Labels",
            "Men's",
            "Men's 2",
            "Men's 3",
            "Men's 4",
            "On Hanger",
            "Spring/summer vibes",
            "Women's",
            "Women's 2",
            "Women's 3",
            "Women's 4"
        ],
        "options": [
            "Back",
            "Back 2",
            "Front",
            "Front 2",
            "Inside label",
            "Left",
            "Left Front",
            "Outside label",
            "Right",
            "Right Back",
            "Right Front"
        ]
    }
}
```

</details>

## Retrieve product variant printfiles with specified technique

URI: `GET /mockup-generator/printfiles/162?technique=EMBROIDERY`

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "product_id": 162,
        "available_placements": {
            "embroidery_chest_center": "Center chest",
            "embroidery_chest_left": "Left chest"
        },
        "printfiles": [
            {
                "printfile_id": 136,
                "width": 1200,
                "height": 1200,
                "dpi": 300,
                "fill_mode": "fit",
                "can_rotate": false
            }
        ],
        "variant_printfiles": [
            {
                "variant_id": 6584,
                "placements": {
                    "embroidery_chest_center": 136,
                    "embroidery_chest_left": 136
                }
            },
            ...
        ],
        "option_groups": [
            "Couple's",
            "Flat",
            "Flat 2",
            "Flat 3",
            "Flat Lifestyle",
            "Holiday season",
            "Men's",
            "Men's 2",
            "Men's 3",
            "Men's 4",
            "On Hanger",
            "Spring/summer vibes",
            "Women's",
            "Women's 2",
            "Women's 3",
            "Women's 4"
        ],
        "options": [
            "Front",
            "Front 2",
            "Zoomed-in",
            "Zoomed-in 2"
        ]
    }
}
```

</details>

## Get layout templates with default technique

URI: `GET /mockup-generator/templates/162`

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "version": 100,
        "variant_mapping": [
            {
                "variant_id": 6584,
                "templates": [
                    {
                        "placement": "front",
                        "template_id": 7597
                    },
                    {
                        "placement": "back",
                        "template_id": 7617
                    },
                    {
                        "placement": "label_outside",
                        "template_id": 3521
                    },
                    {
                        "placement": "label_inside",
                        "template_id": 47233
                    },
                    {
                        "placement": "sleeve_left",
                        "template_id": 7684
                    },
                    {
                        "placement": "sleeve_right",
                        "template_id": 7685
                    }
                ]
            },
            ...
        ],
        "templates": [
            {
                "template_id": 7597,
                "image_url": "https://printful-mockups-dev.s3.amazonaws.com/56-bella-canvas-3413/medium/onman/front/03_bc3413_onman_front_shadows_whitebg.png?v=1646372077",
                "background_url": "https://printful-mockups-dev.s3.amazonaws.com/56-bella-canvas-3413/medium/onman/front/01_bc3413_onman_front_solidblack.png?v=1646372077",
                "background_color": null,
                "printfile_id": 1,
                "template_width": 1000,
                "template_height": 1000,
                "print_area_width": 299,
                "print_area_height": 399,
                "print_area_top": 228,
                "print_area_left": 350,
                "is_template_on_front": true,
                "orientation": "any"
            },
            ...
        ],
        "min_dpi": 75,
        "conflicting_placements": [
            {
                "placement": "back",
                "conflicts": [
                    "label_outside"
                ]
            },
            {
                "placement": "label_outside",
                "conflicts": [
                    "back",
                    "label_inside"
                ]
            },
            {
                "placement": "label_inside",
                "conflicts": [
                    "label_outside"
                ]
            }
        ]
    }
}
```

</details>

## Get layout templates with specified technique

URI: `GET /mockup-generator/templates/162?technique=EMBROIDERY`

<details>
    <summary>Response data</summary>

```
{
    "code": 200,
    "result": {
        "version": 100,
        "variant_mapping": [
            {
                "variant_id": 6584,
                "templates": [
                    {
                        "placement": "embroidery_chest_center",
                        "template_id": 53432
                    },
                    {
                        "placement": "embroidery_chest_left",
                        "template_id": 12221
                    }
                ]
            },
            ...
        ],
        "templates": [
            {
                "template_id": 53432,
                "image_url": "https://printful-mockups-dev.s3.amazonaws.com/56-bella-canvas-3413/medium/onman/embroidery_chest_left/zoomed/04_bc3413_onman_front_zoomed_shadows.png?v=1646372077",
                "background_url": "https://printful-mockups-dev.s3.amazonaws.com/56-bella-canvas-3413/medium/onman/embroidery_chest_left/zoomed/01_bc3413_onman_front_zoomed_solidblack.png?v=1646372077",
                "background_color": null,
                "printfile_id": 136,
                "template_width": 1000,
                "template_height": 1000,
                "print_area_width": 173,
                "print_area_height": 173,
                "print_area_top": 464,
                "print_area_left": 425,
                "is_template_on_front": true,
                "orientation": "any"
            },
            ...
        ],
        "min_dpi": 150,
        "conflicting_placements": [
            {
                "placement": "embroidery_chest_center",
                "conflicts": [
                    "embroidery_chest_left",
                    "embroidery_large_center"
                ]
            },
            {
                "placement": "embroidery_chest_left",
                "conflicts": [
                    "embroidery_chest_center",
                    "embroidery_large_center"
                ]
            }
        ]
    }
}
```

</details>


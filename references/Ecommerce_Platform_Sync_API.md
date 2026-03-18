# Ecommerce Platform Sync API

The ecommerce platform sync API allows you to automatically assign Printful
products and print files to the products in your online store (Shopify, Woocommerce, etc.)
that is linked to Printful.

### Sync Products & Sync Variants explained

Each product in your store can contain one or multiple variants
(some ecommerce platforms would call these options) that the customer can purchase
(imagine multiple sizes or colors of the same t-shirt design). When you link your
ecommerce store to Printful, we create a copy of your product and variant lists
on our side - we call it Sync Products and Sync Variants.

Similar to your store, products that are sold by Printful also consist of multiple variants.
Each t-shirt model is available in many sizes and colors.

![Image](images/variants.png?center)

The purpose of Sync Variants is to let you link each variant from your store that
will be fulfilled by Printful with a design file(s) and specific variant from Printful
product catalogue. When synced products are ordered, we'll know which Printful product
needs to be printed, and the order is imported into Printful for fulfillment.

You can configure each Sync Variant in the Printful Dashboard manually. However,
that can be quite a tedious and repetitive task if your store sells hundreds of products.
This API is designed to help you automate this process.

<div class="alert alert-info">
<strong>Remember</strong><br>
Product data is not imported to Printful immediately after the product is 
created/updated in your ecommerce platform. Depending on the platform, it can take from
a couple of seconds up to a few hours for the products to be available on Printful.
Before the products are imported, you will not be able to update product information 
through this API.
</div>

### External ID

External ID is a feature that allows you to reference Sync Products and Sync Variants
by using the ID from your store.

When requesting Sync Products and Sync Variants, you can use both the Printful ID
and your External ID (if you prefix it with the `@` symbol).

```
GET /sync/products/11001  - reference by Printful Sync Product ID
GET /sync/products/@988123  - reference by Shopify (or other platform's) Product ID
GET /sync/variant/123456  - reference by Printful Sync Variant ID
GET /sync/variant/@123123  - reference by Shopify (or other platform's)  Variant ID
```

### Specifying products

To specify the exact variant of the product, you have to use the `variant_id` attribute of the order item. Each
available unique item (including size/color) has its own Variant ID that can be acquired through
the [Catalog API](#tag/Catalog-API).

### Adding print files

There are two ways to assign a print file to the item.
One is to specify the File ID if the file already exists in the file library of the authorized store:

```json
...
"files":
  [
    {
      "id": 12345
    },
  ],
...
```

Second, and the most convenient method is to specify the file URL.
If a file with the same URL already exists, it will be reused:

```json
...
"files":
  [
    {
      "url": "http://example.com/t-shirts/123/front.pdf"
    },
  ],
...
```

### Specifying multiple files per item

Each item in the order has to be linked with one or multiple files. The available file types for each product are
available from the [Catalog API](#tag/Catalog-API).

You can add one file for each type by specifying the `type` attribute. For the `default` type, this attribute can be
skipped.

```json
...
"files":
  [
    {
      "type": "default",
      "url": "http://example.com/t-shirts/123/front.pdf"
    },
    {
      "type": "back",
      "url": "http://example.com/t-shirts/123/back.pdf"
    }
  ],
...
```

Remember that using additional files can increase the price of the item.

[See examples](#tag/Examples/Ecommerce-Platform-Sync-API-examples/Modify-a-Sync-Variant)


## Endpoints

### GET /sync/products
**Summary:** Get list of Sync Products

Returns list of Sync Product objects from your store.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStoreSyncProducts",
  "security": [
    {
      "OAuth": [
        "sync_products/read"
      ]
    }
  ],
  "parameters": [
    {
      "name": "search",
      "in": "query",
      "required": false,
      "description": "Product search needle",
      "schema": {
        "type": "string"
      }
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
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Store-Information-API/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "query",
      "name": "status",
      "schema": {
        "type": "string",
        "enum": [
          "all",
          "synced",
          "unsynced",
          "ignored",
          "imported",
          "discontinued",
          "out_of_stock"
        ]
      },
      "description": "Parameter used to filter results by status/group of Sync Products"
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
                    "description": "Array of SyncProduct",
                    "items": {
                      "type": "object",
                      "title": "SyncProduct",
                      "description": "Information about the SyncProduct",
                      "required": [
                        "name"
                      ],
                      "properties": {
                        "id": {
                          "description": "SyncProduct ID",
                          "type": "integer",
                          "example": 13,
                          "readOnly": true
                        },
                        "external_id": {
                          "description": "Product ID from the Ecommerce platform",
                          "type": "string",
                          "example": "4235234213"
                        },
                        "name": {
                          "description": "Product name",
                          "type": "string",
                          "example": "T-shirt"
                        },
                        "variants": {
                          "description": "Total number of Sync Variants belonging to this product",
                          "type": "integer",
                          "example": 10,
                          "readOnly": true
                        },
                        "synced": {
                          "description": "Number of synced Sync Variants belonging to this product",
                          "type": "integer",
                          "example": 10,
                          "readOnly": true
                        },
                        "thumbnail": {
                          "type": "string",
                          "description": "Thumbnail image URL. Although we do not limit thumbnail image size, we recommend to keep it reasonably small.",
                          "maxLength": 250,
                          "example": "\u200bhttp://your-domain.com/path/to/thumbnail.png",
                          "writeOnly": true
                        },
                        "thumbnail_url": {
                          "description": "Thumbnail image for the product",
                          "type": "string",
                          "example": "\u200bhttps://your-domain.com/path/to/image.png",
                          "readOnly": true
                        },
                        "is_ignored": {
                          "description": "Indicates if this Sync Product is ignored",
                          "type": "boolean"
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
      "source": "curl --location --request GET 'https://api.printful.com/sync/products' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### GET /sync/products/{id}
**Summary:** Get a Sync Product

Get information about a single Sync Product and its Sync Variants

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStoreSyncProductById",
  "security": [
    {
      "OAuth": [
        "sync_products/read"
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
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Sync Product ID (integer) or External ID (if prefixed with @)"
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
                    "title": "SyncProductInfo",
                    "properties": {
                      "sync_product": {
                        "type": "object",
                        "title": "SyncProduct",
                        "description": "Information about the SyncProduct",
                        "required": [
                          "name"
                        ],
                        "properties": {
                          "id": {
                            "description": "SyncProduct ID",
                            "type": "integer",
                            "example": 13,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Product ID from the Ecommerce platform",
                            "type": "string",
                            "example": "4235234213"
                          },
                          "name": {
                            "description": "Product name",
                            "type": "string",
                            "example": "T-shirt"
                          },
                          "variants": {
                            "description": "Total number of Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "synced": {
                            "description": "Number of synced Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "thumbnail": {
                            "type": "string",
                            "description": "Thumbnail image URL. Although we do not limit thumbnail image size, we recommend to keep it reasonably small.",
                            "maxLength": 250,
                            "example": "\u200bhttp://your-domain.com/path/to/thumbnail.png",
                            "writeOnly": true
                          },
                          "thumbnail_url": {
                            "description": "Thumbnail image for the product",
                            "type": "string",
                            "example": "\u200bhttps://your-domain.com/path/to/image.png",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "description": "Indicates if this Sync Product is ignored",
                            "type": "boolean"
                          }
                        }
                      },
                      "sync_variants": {
                        "type": "array",
                        "description": "Array of Sync Variants available for the selected product",
                        "items": {
                          "type": "object",
                          "title": "SyncVariant",
                          "description": "Information about the SyncVariant",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Sync Variant ID",
                              "example": 10,
                              "readOnly": true
                            },
                            "external_id": {
                              "type": "string",
                              "description": "Variant ID from the Ecommerce platform",
                              "example": "12312414"
                            },
                            "sync_product_id": {
                              "type": "integer",
                              "description": "Sync Product ID that this variant belongs to",
                              "example": 71,
                              "readOnly": true
                            },
                            "name": {
                              "type": "string",
                              "description": "Sync Variant name",
                              "example": "Red T-Shirt",
                              "readOnly": true
                            },
                            "synced": {
                              "type": "boolean",
                              "description": "Indicates if this Sync Variant is properly linked with Printful product",
                              "example": true,
                              "readOnly": true
                            },
                            "variant_id": {
                              "type": "integer",
                              "description": "Printful Variant ID that this Sync Variant is synced to",
                              "example": 3001
                            },
                            "retail_price": {
                              "type": "string",
                              "description": "Retail price that this item is sold for",
                              "example": "29.99"
                            },
                            "currency": {
                              "type": "string",
                              "description": "Currency in which prices are returned",
                              "example": "USD",
                              "readOnly": true
                            },
                            "is_ignored": {
                              "type": "boolean",
                              "description": "Indicates if this Sync Variant is ignored",
                              "example": true
                            },
                            "sku": {
                              "type": "string",
                              "description": "SKU of this Sync Variant",
                              "example": "SKU1234",
                              "nullable": true
                            },
                            "product": {
                              "allOf": [
                                {
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
                                {
                                  "readOnly": true
                                }
                              ]
                            },
                            "files": {
                              "type": "array",
                              "description": "Array of attached printfiles / preview images",
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
                                    "properties": {
                                      "stitch_count_tier": {
                                        "type": "string",
                                        "description": "Stitch count tier see https://help.printful.com/hc/en-us/articles/4909652626204-What-s-the-new-large-embroidery-pricing-",
                                        "nullable": true,
                                        "readOnly": true,
                                        "example": "stitch_tier_1"
                                      }
                                    }
                                  }
                                ]
                              }
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for the configured product/variant [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "SyncVariantOption",
                                "description": "Additional options for the configured product/variant",
                                "required": [
                                  "id",
                                  "value"
                                ],
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option id",
                                    "example": "embroidery_type"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "flat"
                                  }
                                }
                              }
                            },
                            "main_category_id": {
                              "type": "integer",
                              "description": "Printful Variant catalog category ID",
                              "example": 24,
                              "readOnly": true,
                              "nullable": true
                            },
                            "warehouse_product_id": {
                              "type": "integer",
                              "description": "Warehousing product ID. If the sync variant is connected with a warehousing item, this is the ID of corresponding warehouse product.",
                              "example": 3002,
                              "readOnly": true,
                              "nullable": true
                            },
                            "warehouse_product_variant_id": {
                              "type": "integer",
                              "description": "Warehousing variant ID. If the sync variant is connected with a warehousing item, this is its ID.",
                              "example": 3002,
                              "readOnly": true,
                              "nullable": true
                            },
                            "size": {
                              "type": "string",
                              "description": "The size of the associated Catalog Variant",
                              "example": "XS",
                              "readOnly": true,
                              "nullable": true
                            },
                            "color": {
                              "type": "string",
                              "description": "The color of the associated Catalog Variant",
                              "example": "White",
                              "readOnly": true,
                              "nullable": true
                            },
                            "availability_status": {
                              "description": "Indicates the status of the Sync Variant.",
                              "type": "string",
                              "enum": [
                                "active",
                                "discontinued",
                                "out_of_stock",
                                "temporary_out_of_stock"
                              ]
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
      "source": "curl --location --request GET 'https://api.printful.com/sync/products/161636640' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### DELETE /sync/products/{id}
**Summary:** Delete a Sync Product

Deletes a Sync Product with all of its Sync Variants

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteStoreSyncProduct",
  "security": [
    {
      "OAuth": [
        "sync_products"
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
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Sync Product ID (integer) or External ID (if prefixed with @)"
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
                    "title": "SyncProductInfo",
                    "properties": {
                      "sync_product": {
                        "type": "object",
                        "title": "SyncProduct",
                        "description": "Information about the SyncProduct",
                        "required": [
                          "name"
                        ],
                        "properties": {
                          "id": {
                            "description": "SyncProduct ID",
                            "type": "integer",
                            "example": 13,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Product ID from the Ecommerce platform",
                            "type": "string",
                            "example": "4235234213"
                          },
                          "name": {
                            "description": "Product name",
                            "type": "string",
                            "example": "T-shirt"
                          },
                          "variants": {
                            "description": "Total number of Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "synced": {
                            "description": "Number of synced Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "thumbnail": {
                            "type": "string",
                            "description": "Thumbnail image URL. Although we do not limit thumbnail image size, we recommend to keep it reasonably small.",
                            "maxLength": 250,
                            "example": "\u200bhttp://your-domain.com/path/to/thumbnail.png",
                            "writeOnly": true
                          },
                          "thumbnail_url": {
                            "description": "Thumbnail image for the product",
                            "type": "string",
                            "example": "\u200bhttps://your-domain.com/path/to/image.png",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "description": "Indicates if this Sync Product is ignored",
                            "type": "boolean"
                          }
                        }
                      },
                      "sync_variants": {
                        "type": "array",
                        "description": "Array of Sync Variants available for the selected product",
                        "items": {
                          "type": "object",
                          "title": "SyncVariant",
                          "description": "Information about the SyncVariant",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Sync Variant ID",
                              "example": 10,
                              "readOnly": true
                            },
                            "external_id": {
                              "type": "string",
                              "description": "Variant ID from the Ecommerce platform",
                              "example": "12312414"
                            },
                            "sync_product_id": {
                              "type": "integer",
                              "description": "Sync Product ID that this variant belongs to",
                              "example": 71,
                              "readOnly": true
                            },
                            "name": {
                              "type": "string",
                              "description": "Sync Variant name",
                              "example": "Red T-Shirt",
                              "readOnly": true
                            },
                            "synced": {
                              "type": "boolean",
                              "description": "Indicates if this Sync Variant is properly linked with Printful product",
                              "example": true,
                              "readOnly": true
                            },
                            "variant_id": {
                              "type": "integer",
                              "description": "Printful Variant ID that this Sync Variant is synced to",
                              "example": 3001
                            },
                            "retail_price": {
                              "type": "string",
                              "description": "Retail price that this item is sold for",
                              "example": "29.99"
                            },
                            "currency": {
                              "type": "string",
                              "description": "Currency in which prices are returned",
                              "example": "USD",
                              "readOnly": true
                            },
                            "is_ignored": {
                              "type": "boolean",
                              "description": "Indicates if this Sync Variant is ignored",
                              "example": true
                            },
                            "sku": {
                              "type": "string",
                              "description": "SKU of this Sync Variant",
                              "example": "SKU1234",
                              "nullable": true
                            },
                            "product": {
                              "allOf": [
                                {
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
                                {
                                  "readOnly": true
                                }
                              ]
                            },
                            "files": {
                              "type": "array",
                              "description": "Array of attached printfiles / preview images",
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
                                    "properties": {
                                      "stitch_count_tier": {
                                        "type": "string",
                                        "description": "Stitch count tier see https://help.printful.com/hc/en-us/articles/4909652626204-What-s-the-new-large-embroidery-pricing-",
                                        "nullable": true,
                                        "readOnly": true,
                                        "example": "stitch_tier_1"
                                      }
                                    }
                                  }
                                ]
                              }
                            },
                            "options": {
                              "type": "array",
                              "description": "Array of additional options for the configured product/variant [See examples](#tag/Common/Options)",
                              "items": {
                                "type": "object",
                                "title": "SyncVariantOption",
                                "description": "Additional options for the configured product/variant",
                                "required": [
                                  "id",
                                  "value"
                                ],
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "description": "Option id",
                                    "example": "embroidery_type"
                                  },
                                  "value": {
                                    "type": "string",
                                    "description": "Option value",
                                    "example": "flat"
                                  }
                                }
                              }
                            },
                            "main_category_id": {
                              "type": "integer",
                              "description": "Printful Variant catalog category ID",
                              "example": 24,
                              "readOnly": true,
                              "nullable": true
                            },
                            "warehouse_product_id": {
                              "type": "integer",
                              "description": "Warehousing product ID. If the sync variant is connected with a warehousing item, this is the ID of corresponding warehouse product.",
                              "example": 3002,
                              "readOnly": true,
                              "nullable": true
                            },
                            "warehouse_product_variant_id": {
                              "type": "integer",
                              "description": "Warehousing variant ID. If the sync variant is connected with a warehousing item, this is its ID.",
                              "example": 3002,
                              "readOnly": true,
                              "nullable": true
                            },
                            "size": {
                              "type": "string",
                              "description": "The size of the associated Catalog Variant",
                              "example": "XS",
                              "readOnly": true,
                              "nullable": true
                            },
                            "color": {
                              "type": "string",
                              "description": "The color of the associated Catalog Variant",
                              "example": "White",
                              "readOnly": true,
                              "nullable": true
                            },
                            "availability_status": {
                              "description": "Indicates the status of the Sync Variant.",
                              "type": "string",
                              "enum": [
                                "active",
                                "discontinued",
                                "out_of_stock",
                                "temporary_out_of_stock"
                              ]
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
      "source": "curl --location --request DELETE 'https://api.printful.com/sync/products/161636640' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### GET /sync/variant/{id}
**Summary:** Get a Sync Variant

Get information about a single Sync Variant

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStoreSyncVariantById",
  "security": [
    {
      "OAuth": [
        "sync_products/read"
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
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Sync Variant ID (integer) or External ID (if prefixed with @)"
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
                    "title": "SyncVariantInfo",
                    "properties": {
                      "sync_variant": {
                        "type": "object",
                        "title": "SyncVariant",
                        "description": "Information about the SyncVariant",
                        "properties": {
                          "id": {
                            "type": "integer",
                            "description": "Sync Variant ID",
                            "example": 10,
                            "readOnly": true
                          },
                          "external_id": {
                            "type": "string",
                            "description": "Variant ID from the Ecommerce platform",
                            "example": "12312414"
                          },
                          "sync_product_id": {
                            "type": "integer",
                            "description": "Sync Product ID that this variant belongs to",
                            "example": 71,
                            "readOnly": true
                          },
                          "name": {
                            "type": "string",
                            "description": "Sync Variant name",
                            "example": "Red T-Shirt",
                            "readOnly": true
                          },
                          "synced": {
                            "type": "boolean",
                            "description": "Indicates if this Sync Variant is properly linked with Printful product",
                            "example": true,
                            "readOnly": true
                          },
                          "variant_id": {
                            "type": "integer",
                            "description": "Printful Variant ID that this Sync Variant is synced to",
                            "example": 3001
                          },
                          "retail_price": {
                            "type": "string",
                            "description": "Retail price that this item is sold for",
                            "example": "29.99"
                          },
                          "currency": {
                            "type": "string",
                            "description": "Currency in which prices are returned",
                            "example": "USD",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "type": "boolean",
                            "description": "Indicates if this Sync Variant is ignored",
                            "example": true
                          },
                          "sku": {
                            "type": "string",
                            "description": "SKU of this Sync Variant",
                            "example": "SKU1234",
                            "nullable": true
                          },
                          "product": {
                            "allOf": [
                              {
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
                              {
                                "readOnly": true
                              }
                            ]
                          },
                          "files": {
                            "type": "array",
                            "description": "Array of attached printfiles / preview images",
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
                                  "properties": {
                                    "stitch_count_tier": {
                                      "type": "string",
                                      "description": "Stitch count tier see https://help.printful.com/hc/en-us/articles/4909652626204-What-s-the-new-large-embroidery-pricing-",
                                      "nullable": true,
                                      "readOnly": true,
                                      "example": "stitch_tier_1"
                                    }
                                  }
                                }
                              ]
                            }
                          },
                          "options": {
                            "type": "array",
                            "description": "Array of additional options for the configured product/variant [See examples](#tag/Common/Options)",
                            "items": {
                              "type": "object",
                              "title": "SyncVariantOption",
                              "description": "Additional options for the configured product/variant",
                              "required": [
                                "id",
                                "value"
                              ],
                              "properties": {
                                "id": {
                                  "type": "string",
                                  "description": "Option id",
                                  "example": "embroidery_type"
                                },
                                "value": {
                                  "type": "string",
                                  "description": "Option value",
                                  "example": "flat"
                                }
                              }
                            }
                          },
                          "main_category_id": {
                            "type": "integer",
                            "description": "Printful Variant catalog category ID",
                            "example": 24,
                            "readOnly": true,
                            "nullable": true
                          },
                          "warehouse_product_id": {
                            "type": "integer",
                            "description": "Warehousing product ID. If the sync variant is connected with a warehousing item, this is the ID of corresponding warehouse product.",
                            "example": 3002,
                            "readOnly": true,
                            "nullable": true
                          },
                          "warehouse_product_variant_id": {
                            "type": "integer",
                            "description": "Warehousing variant ID. If the sync variant is connected with a warehousing item, this is its ID.",
                            "example": 3002,
                            "readOnly": true,
                            "nullable": true
                          },
                          "size": {
                            "type": "string",
                            "description": "The size of the associated Catalog Variant",
                            "example": "XS",
                            "readOnly": true,
                            "nullable": true
                          },
                          "color": {
                            "type": "string",
                            "description": "The color of the associated Catalog Variant",
                            "example": "White",
                            "readOnly": true,
                            "nullable": true
                          },
                          "availability_status": {
                            "description": "Indicates the status of the Sync Variant.",
                            "type": "string",
                            "enum": [
                              "active",
                              "discontinued",
                              "out_of_stock",
                              "temporary_out_of_stock"
                            ]
                          }
                        }
                      },
                      "sync_product": {
                        "type": "object",
                        "title": "SyncProduct",
                        "description": "Information about the SyncProduct",
                        "required": [
                          "name"
                        ],
                        "properties": {
                          "id": {
                            "description": "SyncProduct ID",
                            "type": "integer",
                            "example": 13,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Product ID from the Ecommerce platform",
                            "type": "string",
                            "example": "4235234213"
                          },
                          "name": {
                            "description": "Product name",
                            "type": "string",
                            "example": "T-shirt"
                          },
                          "variants": {
                            "description": "Total number of Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "synced": {
                            "description": "Number of synced Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "thumbnail": {
                            "type": "string",
                            "description": "Thumbnail image URL. Although we do not limit thumbnail image size, we recommend to keep it reasonably small.",
                            "maxLength": 250,
                            "example": "\u200bhttp://your-domain.com/path/to/thumbnail.png",
                            "writeOnly": true
                          },
                          "thumbnail_url": {
                            "description": "Thumbnail image for the product",
                            "type": "string",
                            "example": "\u200bhttps://your-domain.com/path/to/image.png",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "description": "Indicates if this Sync Product is ignored",
                            "type": "boolean"
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
      "source": "curl --location --request GET 'https://api.printful.com/sync/variant/1781126748' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### PUT /sync/variant/{id}
**Summary:** Modify a Sync Variant

Modifies an existing Sync Variant.

Please note that in the request body you only need to specify the fields that need to be changed. See examples for more insights.

 **Rate limiting:** Up to 10 requests per 60 seconds. A 60 seconds lockout is applied if request count is exceeded.

[See examples](#tag/Examples/Ecommerce-Platform-Sync-API-examples/Modify-a-Sync-Variant)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "updateStoreSyncVariant",
  "security": [
    {
      "OAuth": [
        "sync_products"
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
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Sync Variant ID (integer) or External ID (if prefixed with @)"
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
    "description": "PUT request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "properties": {
            "variant_id": {
              "type": "integer",
              "description": "Printful Variant ID that this Sync Variant is synced to",
              "example": 3001
            },
            "retail_price": {
              "type": "string",
              "description": "Retail price that this item is sold for",
              "example": "29.99"
            },
            "sku": {
              "type": "string",
              "description": "SKU of this Sync Variant",
              "example": "SKU1234",
              "nullable": true
            },
            "is_ignored": {
              "type": "boolean",
              "description": "If is set to true, indicates the Sync Variant has been marked as ignored by Printful for order imports. This also means that Printful will not handle the stock for Shopify stores that have marked this Sync Variant as ignored.",
              "example": false
            },
            "files": {
              "type": "array",
              "description": "Array of attached printfiles/preview images",
              "items": {
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
              }
            },
            "options": {
              "type": "array",
              "description": "Array of additional options for the configured product/variant [See examples](#tag/Common/Options)",
              "items": {
                "type": "object",
                "title": "SyncVariantOption",
                "description": "Additional options for the configured product/variant",
                "required": [
                  "id",
                  "value"
                ],
                "properties": {
                  "id": {
                    "type": "string",
                    "description": "Option id",
                    "example": "embroidery_type"
                  },
                  "value": {
                    "type": "string",
                    "description": "Option value",
                    "example": "flat"
                  }
                }
              }
            },
            "warehouse_product_variant_id": {
              "type": "integer",
              "description": "Warehousing variant ID. If the sync variant is connected with a warehousing item, this is its ID.",
              "example": 3002,
              "readOnly": true,
              "nullable": true
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
                    "title": "SyncVariantInfo",
                    "properties": {
                      "sync_variant": {
                        "type": "object",
                        "title": "SyncVariant",
                        "description": "Information about the SyncVariant",
                        "properties": {
                          "id": {
                            "type": "integer",
                            "description": "Sync Variant ID",
                            "example": 10,
                            "readOnly": true
                          },
                          "external_id": {
                            "type": "string",
                            "description": "Variant ID from the Ecommerce platform",
                            "example": "12312414"
                          },
                          "sync_product_id": {
                            "type": "integer",
                            "description": "Sync Product ID that this variant belongs to",
                            "example": 71,
                            "readOnly": true
                          },
                          "name": {
                            "type": "string",
                            "description": "Sync Variant name",
                            "example": "Red T-Shirt",
                            "readOnly": true
                          },
                          "synced": {
                            "type": "boolean",
                            "description": "Indicates if this Sync Variant is properly linked with Printful product",
                            "example": true,
                            "readOnly": true
                          },
                          "variant_id": {
                            "type": "integer",
                            "description": "Printful Variant ID that this Sync Variant is synced to",
                            "example": 3001
                          },
                          "retail_price": {
                            "type": "string",
                            "description": "Retail price that this item is sold for",
                            "example": "29.99"
                          },
                          "currency": {
                            "type": "string",
                            "description": "Currency in which prices are returned",
                            "example": "USD",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "type": "boolean",
                            "description": "Indicates if this Sync Variant is ignored",
                            "example": true
                          },
                          "sku": {
                            "type": "string",
                            "description": "SKU of this Sync Variant",
                            "example": "SKU1234",
                            "nullable": true
                          },
                          "product": {
                            "allOf": [
                              {
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
                              {
                                "readOnly": true
                              }
                            ]
                          },
                          "files": {
                            "type": "array",
                            "description": "Array of attached printfiles / preview images",
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
                                  "properties": {
                                    "stitch_count_tier": {
                                      "type": "string",
                                      "description": "Stitch count tier see https://help.printful.com/hc/en-us/articles/4909652626204-What-s-the-new-large-embroidery-pricing-",
                                      "nullable": true,
                                      "readOnly": true,
                                      "example": "stitch_tier_1"
                                    }
                                  }
                                }
                              ]
                            }
                          },
                          "options": {
                            "type": "array",
                            "description": "Array of additional options for the configured product/variant [See examples](#tag/Common/Options)",
                            "items": {
                              "type": "object",
                              "title": "SyncVariantOption",
                              "description": "Additional options for the configured product/variant",
                              "required": [
                                "id",
                                "value"
                              ],
                              "properties": {
                                "id": {
                                  "type": "string",
                                  "description": "Option id",
                                  "example": "embroidery_type"
                                },
                                "value": {
                                  "type": "string",
                                  "description": "Option value",
                                  "example": "flat"
                                }
                              }
                            }
                          },
                          "main_category_id": {
                            "type": "integer",
                            "description": "Printful Variant catalog category ID",
                            "example": 24,
                            "readOnly": true,
                            "nullable": true
                          },
                          "warehouse_product_id": {
                            "type": "integer",
                            "description": "Warehousing product ID. If the sync variant is connected with a warehousing item, this is the ID of corresponding warehouse product.",
                            "example": 3002,
                            "readOnly": true,
                            "nullable": true
                          },
                          "warehouse_product_variant_id": {
                            "type": "integer",
                            "description": "Warehousing variant ID. If the sync variant is connected with a warehousing item, this is its ID.",
                            "example": 3002,
                            "readOnly": true,
                            "nullable": true
                          },
                          "size": {
                            "type": "string",
                            "description": "The size of the associated Catalog Variant",
                            "example": "XS",
                            "readOnly": true,
                            "nullable": true
                          },
                          "color": {
                            "type": "string",
                            "description": "The color of the associated Catalog Variant",
                            "example": "White",
                            "readOnly": true,
                            "nullable": true
                          },
                          "availability_status": {
                            "description": "Indicates the status of the Sync Variant.",
                            "type": "string",
                            "enum": [
                              "active",
                              "discontinued",
                              "out_of_stock",
                              "temporary_out_of_stock"
                            ]
                          }
                        }
                      },
                      "sync_product": {
                        "type": "object",
                        "title": "SyncProduct",
                        "description": "Information about the SyncProduct",
                        "required": [
                          "name"
                        ],
                        "properties": {
                          "id": {
                            "description": "SyncProduct ID",
                            "type": "integer",
                            "example": 13,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Product ID from the Ecommerce platform",
                            "type": "string",
                            "example": "4235234213"
                          },
                          "name": {
                            "description": "Product name",
                            "type": "string",
                            "example": "T-shirt"
                          },
                          "variants": {
                            "description": "Total number of Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "synced": {
                            "description": "Number of synced Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "thumbnail": {
                            "type": "string",
                            "description": "Thumbnail image URL. Although we do not limit thumbnail image size, we recommend to keep it reasonably small.",
                            "maxLength": 250,
                            "example": "\u200bhttp://your-domain.com/path/to/thumbnail.png",
                            "writeOnly": true
                          },
                          "thumbnail_url": {
                            "description": "Thumbnail image for the product",
                            "type": "string",
                            "example": "\u200bhttps://your-domain.com/path/to/image.png",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "description": "Indicates if this Sync Product is ignored",
                            "type": "boolean"
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
      "source": "curl --location --request PUT 'https://api.printful.com/sync/variant/1781126748' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--header 'Content-Type: application/json' \\\n--data-raw '{\n    \"id\": 1781126748,\n    \"files\": [\n        {\n            \"url\": \"picsum.photos/200\"\n        }\n    ]\n}'"
    }
  ]
}
```
</details>

---

### DELETE /sync/variant/{id}
**Summary:** Delete a Sync Variant

Deletes configuraton information (`variant_id`, print files and options) and disables automatic order importing for this Sync Variant.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteStoreSyncVariant",
  "security": [
    {
      "OAuth": [
        "sync_products"
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
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Sync Variant ID (integer) or External ID (if prefixed with @)"
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
                    "title": "SyncVariantInfo",
                    "properties": {
                      "sync_variant": {
                        "type": "object",
                        "title": "SyncVariant",
                        "description": "Information about the SyncVariant",
                        "properties": {
                          "id": {
                            "type": "integer",
                            "description": "Sync Variant ID",
                            "example": 10,
                            "readOnly": true
                          },
                          "external_id": {
                            "type": "string",
                            "description": "Variant ID from the Ecommerce platform",
                            "example": "12312414"
                          },
                          "sync_product_id": {
                            "type": "integer",
                            "description": "Sync Product ID that this variant belongs to",
                            "example": 71,
                            "readOnly": true
                          },
                          "name": {
                            "type": "string",
                            "description": "Sync Variant name",
                            "example": "Red T-Shirt",
                            "readOnly": true
                          },
                          "synced": {
                            "type": "boolean",
                            "description": "Indicates if this Sync Variant is properly linked with Printful product",
                            "example": true,
                            "readOnly": true
                          },
                          "variant_id": {
                            "type": "integer",
                            "description": "Printful Variant ID that this Sync Variant is synced to",
                            "example": 3001
                          },
                          "retail_price": {
                            "type": "string",
                            "description": "Retail price that this item is sold for",
                            "example": "29.99"
                          },
                          "currency": {
                            "type": "string",
                            "description": "Currency in which prices are returned",
                            "example": "USD",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "type": "boolean",
                            "description": "Indicates if this Sync Variant is ignored",
                            "example": true
                          },
                          "sku": {
                            "type": "string",
                            "description": "SKU of this Sync Variant",
                            "example": "SKU1234",
                            "nullable": true
                          },
                          "product": {
                            "allOf": [
                              {
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
                              {
                                "readOnly": true
                              }
                            ]
                          },
                          "files": {
                            "type": "array",
                            "description": "Array of attached printfiles / preview images",
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
                                  "properties": {
                                    "stitch_count_tier": {
                                      "type": "string",
                                      "description": "Stitch count tier see https://help.printful.com/hc/en-us/articles/4909652626204-What-s-the-new-large-embroidery-pricing-",
                                      "nullable": true,
                                      "readOnly": true,
                                      "example": "stitch_tier_1"
                                    }
                                  }
                                }
                              ]
                            }
                          },
                          "options": {
                            "type": "array",
                            "description": "Array of additional options for the configured product/variant [See examples](#tag/Common/Options)",
                            "items": {
                              "type": "object",
                              "title": "SyncVariantOption",
                              "description": "Additional options for the configured product/variant",
                              "required": [
                                "id",
                                "value"
                              ],
                              "properties": {
                                "id": {
                                  "type": "string",
                                  "description": "Option id",
                                  "example": "embroidery_type"
                                },
                                "value": {
                                  "type": "string",
                                  "description": "Option value",
                                  "example": "flat"
                                }
                              }
                            }
                          },
                          "main_category_id": {
                            "type": "integer",
                            "description": "Printful Variant catalog category ID",
                            "example": 24,
                            "readOnly": true,
                            "nullable": true
                          },
                          "warehouse_product_id": {
                            "type": "integer",
                            "description": "Warehousing product ID. If the sync variant is connected with a warehousing item, this is the ID of corresponding warehouse product.",
                            "example": 3002,
                            "readOnly": true,
                            "nullable": true
                          },
                          "warehouse_product_variant_id": {
                            "type": "integer",
                            "description": "Warehousing variant ID. If the sync variant is connected with a warehousing item, this is its ID.",
                            "example": 3002,
                            "readOnly": true,
                            "nullable": true
                          },
                          "size": {
                            "type": "string",
                            "description": "The size of the associated Catalog Variant",
                            "example": "XS",
                            "readOnly": true,
                            "nullable": true
                          },
                          "color": {
                            "type": "string",
                            "description": "The color of the associated Catalog Variant",
                            "example": "White",
                            "readOnly": true,
                            "nullable": true
                          },
                          "availability_status": {
                            "description": "Indicates the status of the Sync Variant.",
                            "type": "string",
                            "enum": [
                              "active",
                              "discontinued",
                              "out_of_stock",
                              "temporary_out_of_stock"
                            ]
                          }
                        }
                      },
                      "sync_product": {
                        "type": "object",
                        "title": "SyncProduct",
                        "description": "Information about the SyncProduct",
                        "required": [
                          "name"
                        ],
                        "properties": {
                          "id": {
                            "description": "SyncProduct ID",
                            "type": "integer",
                            "example": 13,
                            "readOnly": true
                          },
                          "external_id": {
                            "description": "Product ID from the Ecommerce platform",
                            "type": "string",
                            "example": "4235234213"
                          },
                          "name": {
                            "description": "Product name",
                            "type": "string",
                            "example": "T-shirt"
                          },
                          "variants": {
                            "description": "Total number of Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "synced": {
                            "description": "Number of synced Sync Variants belonging to this product",
                            "type": "integer",
                            "example": 10,
                            "readOnly": true
                          },
                          "thumbnail": {
                            "type": "string",
                            "description": "Thumbnail image URL. Although we do not limit thumbnail image size, we recommend to keep it reasonably small.",
                            "maxLength": 250,
                            "example": "\u200bhttp://your-domain.com/path/to/thumbnail.png",
                            "writeOnly": true
                          },
                          "thumbnail_url": {
                            "description": "Thumbnail image for the product",
                            "type": "string",
                            "example": "\u200bhttps://your-domain.com/path/to/image.png",
                            "readOnly": true
                          },
                          "is_ignored": {
                            "description": "Indicates if this Sync Product is ignored",
                            "type": "boolean"
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
      "source": "curl --location --request DELETE 'https://api.printful.com/sync/variant/1781126748' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---


# Products API

**The Products API resource lets you create, modify and delete products in a Printful store based on the Manual orders /
API platform** (you can create such store by going to the Stores section at your Printful dashboard.)

**Important**: Jewelry products are not supported via API.

<div class="alert alert-info" style="word-wrap: break-word; padding: 16px; border-radius: 0; cursor: default; color: #31708f; background-color: #d9edf7; border-color: #bce8f1;">
To configure products and variants at a Printful store based on Shopify, WooCommerce or another supported integration platform, please see <a href="#tag/Ecommerce-Platform-Sync-API">Ecommerce Platform Sync API</a>.
</div>

To manage Warehouse products, please see <a href="#tag/Warehouse-Products-API">Warehouse Products API</a>.

### The basics

Each product in your Printful store must contain one or multiple variants (imagine multiple sizes or colors of the same
t-shirt design). Furthermore, for each variant, you have to specify both a blank product variant from our Printful
Catalog and a print file. These two properties together with price and External ID (more on that later) will allow the
variant to be purchasable. Please, see the following sections for more details. Finally, please note that for technical
reasons a product in your Printful store is called a Sync Product and a variant of that product is called a Sync Variant. The maximum supported amount of Sync Variants a Sync Product can have is 100.

### Assigning a blank product variant
Printful has a substantial catalog of blank products and variants, where each variant (e.g. size and color combination
of a particular product) has a unique ID, which we call variant_id. You can browse through the catalog via Catalog API
to find a specific variant_id. Moreover, when creating a Sync Product at your Printful store, each of its Sync Variants
must be associated with a variant_id from the Printful Catalog. Furthermore, to assign a specific variant_id to a
specific Sync Variant, simply add it to the HTTP request body (see examples at the specific endpoint).

### Assigning a single print file
There are two ways to assign a print file to a Sync Variant. One is to specify the File ID if the file already exists in the File library of the authorized store;

### Limitations

**Important**: The Products API is not intended and will never support creating and managing products in external platforms such as Shopify, WooCommerce and others. For managing your products from external platforms please refer to [Ecommerce Platform Sync API](#tag/Ecommerce-Platform-Sync-API)

```
{
    ...
    "files": [
        {
            "id": 12345
        }
    ],
    ...
}
```
The second and most convenient method is to specify the file URL. If a file with the same URL already exists, it will be reused.

```
{
    ...
    "files": [
        {
            "url": "http://example.com/t-shirts/123/front.pdf"
        }
    ],
    ...
}
```
Moreover, each Sync Variant has to be linked with one or multiple print files. The available file types for each product are available from the Printful Catalogue API. You can add one file for each type by specifying the type attribute. For the
default type, this attribute can be skipped.

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
    }
],
...
```
Remember that using additional files can increase the price of the item.

### External ID
When creating a Sync Product and/or Sync Variant you can specify an External ID, which you can then use as a reference when managing or even ordering the specific Sync Product or Sync Variant. In particular, when requesting a specific Sync Product
and Sync Variant, you can use either the internal Printful ID or your External ID (prefixed with an @ symbol) at the request URL.

### Native inside label
Printful previously allowed customers to upload a fully customized inside label. Since these labels had to contain specific information about fabric composition, manufacturing, etc. to meet the legal requirements, users usually encountered issues to
get their labels printed.

Inside labels are printed on the inside of the garment and require the removal of the original manufacturer's tag. They're only available for apparel with tear-away labels. An inside label must include the country of manufacturing origin, original
garment size, and material information. To use our native label template you only need to upload a graphic (such as your brand's logo). The mandatory content will be generated and placed automatically.

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
Printful previously supported fully customized inside labels. These have now been deprecated. The ability to create orders with fully customized inside labels has been limited to only users who were actively using them in their stores before April
2020. This feature is no longer accessible to new users.

[See examples](#tag/Examples/Products-API-examples)


## Endpoints

### GET /store/products
**Summary:** Get Sync Products

Returns a list of Sync Product objects from your custom Printful store.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getSyncProducts",
  "security": [
    {
      "OAuth": [
        "sync_products/read"
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
    },
    {
      "in": "query",
      "name": "category_id",
      "schema": {
        "type": "string"
      },
      "required": false,
      "description": "A comma-separated list of Category IDs of the Products that are to be returned"
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
      "source": "curl --location --request GET 'https://api.printful.com/store/products' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n// set some paging info\n$offset = 0;\n$limit = 20;\n\n/** @var SyncProductsResponse $list */\n$list = $productsApi->getProducts($offset, $limit);\n"
    }
  ]
}
```
</details>

---

### POST /store/products
**Summary:** Create a new Sync Product

Creates a new Sync Product together with its Sync Variants ([See examples](#tag/Products-API/operation/createSyncProduct)).

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createSyncProduct",
  "security": [
    {
      "OAuth": [
        "sync_products"
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
          "title": "Product",
          "description": "Information about the Sync Product",
          "required": [
            "sync_product",
            "sync_variants"
          ],
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
              "description": "Information about the Sync Variants",
              "items": {
                "allOf": [
                  {
                    "required": [
                      "files",
                      "variant_id"
                    ]
                  },
                  {
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
                ]
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
      "source": "curl --location --request POST 'https://api.printful.com/store/products' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"sync_product\": {\n        \"name\": \"T-Shirt\",\n        \"thumbnail\": \"https://picsum.photos/200/300\"\n    },\n    \"sync_variants\": [\n        {\n            \"retail_price\": 21.00,\n            \"variant_id\": 4011,\n            \"files\": [\n                {\n                    \"url\": \"https://picsum.photos/200/300?image=1\"\n                },\n                {\n                    \"type\": \"back\",\n                    \"url\": \"https://picsum.photos/200/300?image=1\"\n                }\n            ]\n        },\n        {\n            \"retail_price\": 21.00,\n            \"variant_id\": 4012,\n            \"files\": [\n                {\n                    \"url\": \"https://picsum.photos/200/300?image=1\"\n                },\n                {\n                    \"type\": \"back\",\n                    \"url\": \"https://picsum.photos/200/300?image=1\"\n                }\n            ]\n        }\n    ]\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n// create product request\n$productRequest = new SyncProductRequest;\n\n// set id in my store for this product (optional)\n$productRequest->externalId = 1;\n\n// set product name\n$productRequest->name = 'My new shirt';\n\n// set thumbnail url\n$productRequest->thumbnail = 'https://www.my-webshop.com/shirt.jpg';\n\n// create creationParams\n$creationParams = new SyncProductCreationParameters($productRequest);\n\n// create variant\n$syncVariantRequest = new SyncVariantRequest;\n\n// set id in my store for this variant (optional)\n$syncVariantRequest->externalId = 1;\n\n// set variant in from Printful Catalog(https://www.printful.com/docs/catalog)\n$syncVariantRequest->variantId = 4011; // Bella + Canvas 3001, S, White\n\n// set retail price that this item is sold for (optional)\n$syncVariantRequest->retailPrice = 21.00;\n\n// create print file\n// $file->id or $file->url is required\n$file = new SyncVariantRequestFile();\n\n// file id from my File library in Printful (https://www.printful.com/docs/files)\n// $file->id = 1;\n\n// set print file url\n$file->url = 'https://www.my-webshop.com/shirt.jpg';\n\n// set print file placement on item. If not set, default placement for this product will be used\n$file->type = 'front';\n\n// add print file to variant. You can add multiple files for single variant (type must be unique)\n$syncVariantRequest->addFile($file);\n\n// create variant option\n$option = new SyncVariantRequestOption;\n$option->id = 'embroidery_type';\n$option->value = 'flat';\n\n$syncVariantRequest->addOption($option);\n\n// add variant to creation params (you can create and add multiple variants)\n$creationParams->addSyncVariant($syncVariantRequest);\n\n$product = $productsApi->createProduct($creationParams);\n"
    }
  ]
}
```
</details>

---

### GET /store/products/{id}
**Summary:** Get a Sync Product

Get information about a single Sync Product and its Sync Variants.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getSyncProductById",
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
      "source": "curl --location --request GET 'https://api.printful.com/store/products/{sync_product_id}' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n// product id in your Printful store\n$productId = 1;\n\n// get product by id\n$response = $productsApi->getProduct($productId);\n"
    }
  ]
}
```
</details>

---

### DELETE /store/products/{id}
**Summary:** Delete a Sync Product

Deletes a Sync Product with all of its Sync Variants

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteSyncProduct",
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
      "source": "curl --location --request DELETE 'https://api.printful.com/store/products/161636638' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// product id in Printful store, which we want to delete\n$id = 1;\n\n// you can also use your external id\n// $externalId = 32142;\n// $id = '@'.$externalId;\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n$productsApi->deleteProduct($id);\n"
    }
  ]
}
```
</details>

---

### PUT /store/products/{id}
**Summary:** Modify a Sync Product

Modifies an existing Sync Product with its Sync Variants.

Please note that in the request body you only need to specify the fields that need to be changed. Furthermore, if you want to update existing sync variants, 
then in the sync variants array you must specify the IDs of all existing sync variants. All omitted existing sync variants will be deleted. All new sync 
variants without an ID will be created. See examples for more insights.

**Rate limiting:** Up to 10 requests per 60 seconds. A 60 seconds lockout is applied if request count is exceeded.

[See examples](#tag/Examples/Products-API-examples/Modify-a-Sync-Product)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "updateSyncProduct",
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
  "requestBody": {
    "description": "PUT request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
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
              "description": "Information about the Sync Variants",
              "items": {
                "allOf": [
                  {
                    "type": "object"
                  },
                  {
                    "anyOf": [
                      {
                        "required": [
                          "files",
                          "variant_id"
                        ]
                      },
                      {
                        "required": [
                          "id"
                        ]
                      }
                    ],
                    "properties": {
                      "id": {
                        "type": "integer",
                        "description": "Sync Variant ID. Please specify the IDs of all Sync Variants you wish to keep.",
                        "example": 10,
                        "readOnly": false
                      }
                    }
                  },
                  {
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
                ]
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
      "source": "curl --location --request PUT 'https://api.printful.com/store/products/161636640' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"sync_product\": {\n        \"name\": \"New product name\"\n    },\n    \"sync_variants\": [\n        {\n            \"id\": 1781126748,\n            \"retail_price\": \"18.00\",\n            \"variant_id\": 4011\n        },\n        {\n            \"id\": 1781126749\n        }\n    ]\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// product id in Printful store, which we want to update\n$productId = 1;\n\n// you can also use your external id\n// $productId = '@' . 32142;\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n// build update params manually\n$params = new SyncProductUpdateParameters();\n\n// build product request manually\n$productRequest = new SyncProductRequest();\n\n// in this example we only need to update this one field, so we omit rest\n$productRequest->name = 'New product name';\n\n// in this example we only need to update products info, so we omit variant list info\n$params->syncProduct = $productRequest;\n\n// preform update\n$product = $productsApi->updateProduct($productId, $params);\n"
    }
  ]
}
```
</details>

---

### GET /store/variants/{id}
**Summary:** Get a Sync Variant

Get information about a single Sync Variant.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getSyncVariantById",
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
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
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
  }
}
```
</details>

---

### DELETE /store/variants/{id}
**Summary:** Delete a Sync Variant

Deletes a single Sync Variant.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteSyncVariant",
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
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
                "type": "array",
                "maxItems": 0,
                "minItems": 0
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
      "source": "curl --location --request DELETE 'https://api.printful.com/store/variants/1781126754' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// variant id in Printful store, which we want to delete\n// remember that you can not delete products last variant, you \n// should delete whole product instead\n$id = 1;\n\n// you can also use your external id\n// $id = '@' . 32142;\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n$productsApi->deleteVariant($id);\n"
    }
  ]
}
```
</details>

---

### PUT /store/variants/{id}
**Summary:** Modify a Sync Variant

Modifies an existing Sync Variant.

Please note that in the request body you only need to specify the fields that need to be changed. See examples for more insights.

[See examples](#tag/Examples/Products-API-examples/Modify-a-Sync-Variant)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "updateSyncVariant",
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
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "allOf": [
            {
              "type": "object",
              "properties": {
                "id": {
                  "type": "integer",
                  "description": "Sync Variant ID. Please specify the IDs of all Sync Variants you wish to keep.",
                  "example": 10,
                  "readOnly": false
                }
              }
            },
            {
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
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
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
      "source": "curl --location --request PUT 'https://api.printful.com/store/variants/161636640' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    {\n        \"retail_price\": 25.00,\n        \"variant_id\": 4011\n    }\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// variant id in Printful store, which we want to update\n$variantId = 1;\n\n// you can also use your external id\n// $variantId = '@' . 32142;\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n$variantRequest = new SyncVariantRequest;\n\n// in this example we only update retail price, so we omit everything else\n$variantRequest->retailPrice = 22.00;\n\n$updatedVariant = $productsApi->updateVariant($variantId, $variantRequest);\n"
    }
  ]
}
```
</details>

---

### POST /store/products/{id}/variants
**Summary:** Create a new Sync Variant

Creates a new Sync Variant for an existing Sync Product ([See examples](#tag/Examples/Products-API-examples/Create-a-new-Sync-Variant)).

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createSyncVariant",
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
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "allOf": [
            {
              "required": [
                "files",
                "variant_id"
              ]
            },
            {
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
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
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
      "source": "curl --location --request POST 'https://api.printful.com/store/products/161636640/variants' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"retail_price\": 19.00,\n    \"variant_id\": 4011,\n    \"files\": [\n        {\n            \"type\": \"back\",\n            \"url\": \"https://picsum.photos/200/300?image=2\"\n        }\n    ],\n    \"options\": [\n        {\n            \"id\": \"embroidery_type\",\n            \"value\": \"flat\"\n        }\n    ]\n}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n// product id in your Printful store to which variant will be added\n$productId = 1;\n\n// you can also use your external id\n// $productId = '@'.32142;\n\n// create ApiClient\n$pf = new PrintfulApiClient($apiKey);\n\n// create Products Api object\n$productsApi = new PrintfulProducts($pf);\n\n$variantRequest = SyncVariantRequest::fromArray([\n    'external_id' => 1, // set id in my store for this variant (optional)\n    'retail_price' => 21.00, // set retail price that this item is sold for (optional)\n    'variant_id' => 4011, // set variant in from Printful Catalog(https://www.printful.com/docs/catalog)\n    'files' => [\n        [\n            'url' => 'https://www.my-webshop.com/shirt.jpg',\n        ],\n        [\n            'type' => 'back', // set print file placement on item. If not set, default placement for this product will be used\n            'url' => 'https://www.my-webshop.com/shirt.jpg',\n        ],\n    ],\n    'options' => [\n        [\n            'id' => 'embroidery_type',\n            'value' => 'flat',\n        ],\n    ],\n]);\n\n$variant = $productsApi->createVariant($productId, $variantRequest);\n"
    }
  ]
}
```
</details>

---


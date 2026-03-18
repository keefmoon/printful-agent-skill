# Catalog API

Printful has a substantial catalog of blank Products and Variants. A Product can describe a specific type, model and manufacturer of the item, while the Variant specifies the more detailed attributes of the product like the exact size/color of a
T-shirt or the dimensions of a poster. Moreover, each item in the Printful Catalog has a unique Variant ID. When managing Sync Products or orders, you will need to specify the Variant ID of the specific blank item, hence you can use this API resource
to find the needed Variant ID.

<div class="alert alert-info">
It is critically important to always refer to the Variant IDs (<strong>NOT Product IDs</strong>) when creating products or orders. Mixing up and using the Product ID instead of the Variant ID can lead to an entirely different product created or item ordered. 
The Product entity is only meant to allow of easier browsing of what Printful offers.
</div>

You can also use this API resource to find out the types of print files a product can be configured for as well as the
additional price each print file would cost (e.g. the back print or inside label print for T-shirts). Moreover, some
product types allow for additional options (e.g. embroidery type and thread colors) - these options are listed in the
responses as well.

<div class="alert alert-info">
Please note that the current Catalog API does not reflect the discounted pricing available in the Printful subscription plans.
</div>

**Important**: Jewelry products are not supported via API.

**Rate limiting**: For unauthenticated usages, up to 30 requests per 60 seconds. A 60 seconds lockout is applied if
request count is exceeded.

### Size guides

The [Get Product Size Guide](#operation/getProductSizeGuideById) endpoint will return size guide data for the selected
product.

There are three types of size tables available, as described by the following table:

| Table type                    | API name           | Description                                                                                 |
|-------------------------------|--------------------|---------------------------------------------------------------------------------------------|
| Measure yourself              | `measure_yourself` | Measurements of the product to measure the body provided by the supplier.                   |
| Product measurements          | `product_measure`  | Measurements of the product provided by the supplier.                                       |
| International size conversion | `international`    | International size conversion – e.g. US, EU or UK sizes corresponding to the product sizes. |

Not each table type might be available for the selected product.

[See examples](#tag/Examples/Catalog-API-examples/Using-size-guides)

## Endpoints

### GET /products
**Summary:** Get Products

Returns list of Products available in the Printful

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProducts",
  "parameters": [
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
                    "type": "array",
                    "items": {
                      "type": "object",
                      "title": "Product",
                      "description": "Information about the Product that the Variant belongs to",
                      "properties": {
                        "id": {
                          "description": "Product ID",
                          "type": "integer",
                          "example": 13
                        },
                        "main_category_id": {
                          "description": "Main category of product",
                          "type": "integer",
                          "example": 24
                        },
                        "type": {
                          "description": "Product type identifier",
                          "type": "string",
                          "example": "T-SHIRT"
                        },
                        "type_name": {
                          "description": "Product type name",
                          "type": "string",
                          "example": "T-Shirt"
                        },
                        "title": {
                          "description": "Product title",
                          "type": "string",
                          "example": "Unisex Staple T-Shirt | Bella + Canvas 3001"
                        },
                        "brand": {
                          "description": "Brand name",
                          "type": "string",
                          "example": "Gildan"
                        },
                        "model": {
                          "description": "Model name",
                          "type": "string",
                          "example": "2200 Ultra Cotton Tank Top"
                        },
                        "image": {
                          "description": "URL of a sample image for this product",
                          "type": "string",
                          "example": "https://files.cdn.printful.com/products/12/product_1550594502.jpg"
                        },
                        "variant_count": {
                          "description": "Number of available variants for this product",
                          "type": "integer",
                          "example": 30
                        },
                        "currency": {
                          "description": "Currency in which prices are returned",
                          "type": "string",
                          "example": "EUR"
                        },
                        "files": {
                          "description": "Definitions of Print/Mockup file categories that can be attached to this product",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "FileType",
                            "properties": {
                              "id": {
                                "description": "Deprecated file type identifier. Please use type field instead!",
                                "deprecated": true,
                                "type": "string",
                                "example": "default"
                              },
                              "type": {
                                "description": "File type identifier - use this to specify a file's purpose when creating an order",
                                "type": "string",
                                "example": "front"
                              },
                              "title": {
                                "description": "Display name",
                                "type": "string",
                                "example": "Front print"
                              },
                              "additional_price": {
                                "description": "Additional price when this print file type is used",
                                "type": "string",
                                "example": "2.22"
                              },
                              "options": {
                                "description": "Additional options available to product files",
                                "type": "array",
                                "items": {
                                  "type": "object",
                                  "title": "CatalogFileOption",
                                  "properties": {
                                    "id": {
                                      "type": "string",
                                      "description": "File option identifier. Use this to specify which option you are adding to your file in a request.",
                                      "example": "full_color"
                                    },
                                    "type": {
                                      "type": "string",
                                      "example": "bool",
                                      "description": "The type of the value property when using this option in a request."
                                    },
                                    "title": {
                                      "type": "string",
                                      "example": "Unlimited color"
                                    },
                                    "additional_price": {
                                      "type": "number",
                                      "description": "Additional cost this will add to the item.",
                                      "example": 3.25
                                    }
                                  }
                                }
                              }
                            }
                          }
                        },
                        "options": {
                          "description": "Definitions of additional options that are available for this product [See examples](#tag/Common/Options)",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "OptionType",
                            "properties": {
                              "id": {
                                "description": "Option identifier - use this to specify the option when creating an order",
                                "type": "string",
                                "example": "embroidery_type"
                              },
                              "title": {
                                "description": "Display name",
                                "type": "string",
                                "example": "Embroidery type"
                              },
                              "type": {
                                "description": "Data type of this option (currently only 'bool' is supported)",
                                "type": "string",
                                "example": "radio"
                              },
                              "values": {
                                "description": "Possible option values - [key, value] map",
                                "type": "object",
                                "additionalProperties": true,
                                "example": {
                                  "flat": "Flat Embroidery",
                                  "3d": "3D Puff",
                                  "both": "Partial 3D Puff"
                                }
                              },
                              "additional_price": {
                                "description": "Additional price when this option is used",
                                "type": "string"
                              },
                              "additional_price_breakdown": {
                                "description": "Additional price breakdown by type - [key, value] map",
                                "type": "object",
                                "additionalProperties": true,
                                "example": {
                                  "flat": "0.00",
                                  "3d": "0.00",
                                  "both": "0.00"
                                }
                              }
                            }
                          }
                        },
                        "is_discontinued": {
                          "description": "If product is disabled in push",
                          "type": "boolean",
                          "example": false
                        },
                        "avg_fulfillment_time": {
                          "description": "Average number of days for order to be fulfilled",
                          "type": "number",
                          "example": 4.3
                        },
                        "description": {
                          "description": "Product description",
                          "type": "string"
                        },
                        "techniques": {
                          "description": "Available techniques",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "AvailableTechnique",
                            "properties": {
                              "key": {
                                "description": "The technique key to be used in the API",
                                "type": "string",
                                "example": "DTG"
                              },
                              "display_name": {
                                "description": "The human-readable technique name",
                                "type": "string",
                                "example": "DTG printing"
                              },
                              "is_default": {
                                "description": "Whether the technique is the default one",
                                "type": "boolean"
                              }
                            }
                          }
                        },
                        "origin_country": {
                          "description": "The origin country for inside label",
                          "type": "string",
                          "nullable": true,
                          "example": "Nicaragua"
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
      "source": "curl --location --request GET 'https://api.printful.com/products'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\nuse Printful\\PrintfulApiClient;\n\nrequire_once __DIR__ . '../vendor/autoload.php';\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n$products = $pf->get('products');\n"
    }
  ]
}
```
</details>

---

### GET /products/variant/{id}
**Summary:** Get Variant

Returns information about a specific Variant and its Product

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getVariantById",
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Variant id"
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
                    "title": "VariantInfo",
                    "properties": {
                      "variant": {
                        "type": "object",
                        "title": "Variant",
                        "properties": {
                          "id": {
                            "description": "Variant ID, use this to specify the product when creating orders",
                            "type": "integer",
                            "example": 100
                          },
                          "product_id": {
                            "description": "ID of the product that this variant belongs to",
                            "type": "integer",
                            "example": 12
                          },
                          "name": {
                            "description": "Display name",
                            "type": "string",
                            "example": "Gildan 64000 Unisex Softstyle T-Shirt with Tear Away (Black / 2XL)"
                          },
                          "size": {
                            "description": "Item size",
                            "type": "string",
                            "example": "2XL"
                          },
                          "color": {
                            "description": "Item color",
                            "type": "string",
                            "example": "Black"
                          },
                          "color_code": {
                            "description": "Hexadecimal RGB color code. May not exactly reflect the real-world color",
                            "type": "string",
                            "example": "#14191e"
                          },
                          "color_code2": {
                            "description": "Secondary hexadecimal RGB color code. May not exactly reflect the real-world color",
                            "type": "string"
                          },
                          "image": {
                            "description": "URL of a preview image for this variant",
                            "type": "string",
                            "example": "https://files.cdn.printful.com/products/12/629_1517916489.jpg"
                          },
                          "price": {
                            "description": "Variant's price (can change depending on print files and optional settings)",
                            "type": "string",
                            "example": "9.85"
                          },
                          "in_stock": {
                            "description": "Stock availability of this variant",
                            "type": "boolean",
                            "example": true
                          },
                          "availability_regions": {
                            "description": "Map of [region code, region name] of regions where the variant is available for fulfillment",
                            "type": "object",
                            "additionalProperties": true,
                            "example": {
                              "US": "USA",
                              "EU": "Europe"
                            }
                          },
                          "availability_status": {
                            "description": "Detailed stock status per region",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "AvailabilityStatus",
                              "properties": {
                                "region": {
                                  "description": "Region code",
                                  "type": "string",
                                  "example": "US"
                                },
                                "status": {
                                  "description": "Stock status. Possible values include: 'in_stock' - available for fulfillment, 'stocked_on_demand' - available for fulfillment, 'discontinued' - permanently unavailable, 'out_of_stock' - temporarily unavailable",
                                  "type": "string",
                                  "example": "in_stock"
                                }
                              }
                            }
                          },
                          "material": {
                            "description": "A list of materials this Variant is composed of",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Material",
                              "properties": {
                                "name": {
                                  "type": "string",
                                  "description": "Material name",
                                  "example": "cotton"
                                },
                                "percentage": {
                                  "type": "number",
                                  "description": "Percentage of the material in the product",
                                  "example": 100
                                }
                              }
                            }
                          }
                        }
                      },
                      "product": {
                        "type": "object",
                        "title": "Product",
                        "description": "Information about the Product that the Variant belongs to",
                        "properties": {
                          "id": {
                            "description": "Product ID",
                            "type": "integer",
                            "example": 13
                          },
                          "main_category_id": {
                            "description": "Main category of product",
                            "type": "integer",
                            "example": 24
                          },
                          "type": {
                            "description": "Product type identifier",
                            "type": "string",
                            "example": "T-SHIRT"
                          },
                          "type_name": {
                            "description": "Product type name",
                            "type": "string",
                            "example": "T-Shirt"
                          },
                          "title": {
                            "description": "Product title",
                            "type": "string",
                            "example": "Unisex Staple T-Shirt | Bella + Canvas 3001"
                          },
                          "brand": {
                            "description": "Brand name",
                            "type": "string",
                            "example": "Gildan"
                          },
                          "model": {
                            "description": "Model name",
                            "type": "string",
                            "example": "2200 Ultra Cotton Tank Top"
                          },
                          "image": {
                            "description": "URL of a sample image for this product",
                            "type": "string",
                            "example": "https://files.cdn.printful.com/products/12/product_1550594502.jpg"
                          },
                          "variant_count": {
                            "description": "Number of available variants for this product",
                            "type": "integer",
                            "example": 30
                          },
                          "currency": {
                            "description": "Currency in which prices are returned",
                            "type": "string",
                            "example": "EUR"
                          },
                          "files": {
                            "description": "Definitions of Print/Mockup file categories that can be attached to this product",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "FileType",
                              "properties": {
                                "id": {
                                  "description": "Deprecated file type identifier. Please use type field instead!",
                                  "deprecated": true,
                                  "type": "string",
                                  "example": "default"
                                },
                                "type": {
                                  "description": "File type identifier - use this to specify a file's purpose when creating an order",
                                  "type": "string",
                                  "example": "front"
                                },
                                "title": {
                                  "description": "Display name",
                                  "type": "string",
                                  "example": "Front print"
                                },
                                "additional_price": {
                                  "description": "Additional price when this print file type is used",
                                  "type": "string",
                                  "example": "2.22"
                                },
                                "options": {
                                  "description": "Additional options available to product files",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "CatalogFileOption",
                                    "properties": {
                                      "id": {
                                        "type": "string",
                                        "description": "File option identifier. Use this to specify which option you are adding to your file in a request.",
                                        "example": "full_color"
                                      },
                                      "type": {
                                        "type": "string",
                                        "example": "bool",
                                        "description": "The type of the value property when using this option in a request."
                                      },
                                      "title": {
                                        "type": "string",
                                        "example": "Unlimited color"
                                      },
                                      "additional_price": {
                                        "type": "number",
                                        "description": "Additional cost this will add to the item.",
                                        "example": 3.25
                                      }
                                    }
                                  }
                                }
                              }
                            }
                          },
                          "options": {
                            "description": "Definitions of additional options that are available for this product [See examples](#tag/Common/Options)",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "OptionType",
                              "properties": {
                                "id": {
                                  "description": "Option identifier - use this to specify the option when creating an order",
                                  "type": "string",
                                  "example": "embroidery_type"
                                },
                                "title": {
                                  "description": "Display name",
                                  "type": "string",
                                  "example": "Embroidery type"
                                },
                                "type": {
                                  "description": "Data type of this option (currently only 'bool' is supported)",
                                  "type": "string",
                                  "example": "radio"
                                },
                                "values": {
                                  "description": "Possible option values - [key, value] map",
                                  "type": "object",
                                  "additionalProperties": true,
                                  "example": {
                                    "flat": "Flat Embroidery",
                                    "3d": "3D Puff",
                                    "both": "Partial 3D Puff"
                                  }
                                },
                                "additional_price": {
                                  "description": "Additional price when this option is used",
                                  "type": "string"
                                },
                                "additional_price_breakdown": {
                                  "description": "Additional price breakdown by type - [key, value] map",
                                  "type": "object",
                                  "additionalProperties": true,
                                  "example": {
                                    "flat": "0.00",
                                    "3d": "0.00",
                                    "both": "0.00"
                                  }
                                }
                              }
                            }
                          },
                          "is_discontinued": {
                            "description": "If product is disabled in push",
                            "type": "boolean",
                            "example": false
                          },
                          "avg_fulfillment_time": {
                            "description": "Average number of days for order to be fulfilled",
                            "type": "number",
                            "example": 4.3
                          },
                          "description": {
                            "description": "Product description",
                            "type": "string"
                          },
                          "techniques": {
                            "description": "Available techniques",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "AvailableTechnique",
                              "properties": {
                                "key": {
                                  "description": "The technique key to be used in the API",
                                  "type": "string",
                                  "example": "DTG"
                                },
                                "display_name": {
                                  "description": "The human-readable technique name",
                                  "type": "string",
                                  "example": "DTG printing"
                                },
                                "is_default": {
                                  "description": "Whether the technique is the default one",
                                  "type": "boolean"
                                }
                              }
                            }
                          },
                          "origin_country": {
                            "description": "The origin country for inside label",
                            "type": "string",
                            "nullable": true,
                            "example": "Nicaragua"
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
      "source": "curl --location --request GET 'https://api.printful.com/products/variant/4018'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\nuse Printful\\PrintfulApiClient;\n\nrequire_once __DIR__ . '../vendor/autoload.php';\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n$variant = $pf->get('products/variant/1007');\n"
    }
  ]
}
```
</details>

---

### GET /products/{id}
**Summary:** Get Product

Returns information about a specific product and a list of variants for this product.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductById",
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Product ID."
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
                    "title": "ProductInfo",
                    "properties": {
                      "product": {
                        "type": "object",
                        "title": "Product",
                        "description": "Information about the Product that the Variant belongs to",
                        "properties": {
                          "id": {
                            "description": "Product ID",
                            "type": "integer",
                            "example": 13
                          },
                          "main_category_id": {
                            "description": "Main category of product",
                            "type": "integer",
                            "example": 24
                          },
                          "type": {
                            "description": "Product type identifier",
                            "type": "string",
                            "example": "T-SHIRT"
                          },
                          "type_name": {
                            "description": "Product type name",
                            "type": "string",
                            "example": "T-Shirt"
                          },
                          "title": {
                            "description": "Product title",
                            "type": "string",
                            "example": "Unisex Staple T-Shirt | Bella + Canvas 3001"
                          },
                          "brand": {
                            "description": "Brand name",
                            "type": "string",
                            "example": "Gildan"
                          },
                          "model": {
                            "description": "Model name",
                            "type": "string",
                            "example": "2200 Ultra Cotton Tank Top"
                          },
                          "image": {
                            "description": "URL of a sample image for this product",
                            "type": "string",
                            "example": "https://files.cdn.printful.com/products/12/product_1550594502.jpg"
                          },
                          "variant_count": {
                            "description": "Number of available variants for this product",
                            "type": "integer",
                            "example": 30
                          },
                          "currency": {
                            "description": "Currency in which prices are returned",
                            "type": "string",
                            "example": "EUR"
                          },
                          "files": {
                            "description": "Definitions of Print/Mockup file categories that can be attached to this product",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "FileType",
                              "properties": {
                                "id": {
                                  "description": "Deprecated file type identifier. Please use type field instead!",
                                  "deprecated": true,
                                  "type": "string",
                                  "example": "default"
                                },
                                "type": {
                                  "description": "File type identifier - use this to specify a file's purpose when creating an order",
                                  "type": "string",
                                  "example": "front"
                                },
                                "title": {
                                  "description": "Display name",
                                  "type": "string",
                                  "example": "Front print"
                                },
                                "additional_price": {
                                  "description": "Additional price when this print file type is used",
                                  "type": "string",
                                  "example": "2.22"
                                },
                                "options": {
                                  "description": "Additional options available to product files",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "CatalogFileOption",
                                    "properties": {
                                      "id": {
                                        "type": "string",
                                        "description": "File option identifier. Use this to specify which option you are adding to your file in a request.",
                                        "example": "full_color"
                                      },
                                      "type": {
                                        "type": "string",
                                        "example": "bool",
                                        "description": "The type of the value property when using this option in a request."
                                      },
                                      "title": {
                                        "type": "string",
                                        "example": "Unlimited color"
                                      },
                                      "additional_price": {
                                        "type": "number",
                                        "description": "Additional cost this will add to the item.",
                                        "example": 3.25
                                      }
                                    }
                                  }
                                }
                              }
                            }
                          },
                          "options": {
                            "description": "Definitions of additional options that are available for this product [See examples](#tag/Common/Options)",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "OptionType",
                              "properties": {
                                "id": {
                                  "description": "Option identifier - use this to specify the option when creating an order",
                                  "type": "string",
                                  "example": "embroidery_type"
                                },
                                "title": {
                                  "description": "Display name",
                                  "type": "string",
                                  "example": "Embroidery type"
                                },
                                "type": {
                                  "description": "Data type of this option (currently only 'bool' is supported)",
                                  "type": "string",
                                  "example": "radio"
                                },
                                "values": {
                                  "description": "Possible option values - [key, value] map",
                                  "type": "object",
                                  "additionalProperties": true,
                                  "example": {
                                    "flat": "Flat Embroidery",
                                    "3d": "3D Puff",
                                    "both": "Partial 3D Puff"
                                  }
                                },
                                "additional_price": {
                                  "description": "Additional price when this option is used",
                                  "type": "string"
                                },
                                "additional_price_breakdown": {
                                  "description": "Additional price breakdown by type - [key, value] map",
                                  "type": "object",
                                  "additionalProperties": true,
                                  "example": {
                                    "flat": "0.00",
                                    "3d": "0.00",
                                    "both": "0.00"
                                  }
                                }
                              }
                            }
                          },
                          "is_discontinued": {
                            "description": "If product is disabled in push",
                            "type": "boolean",
                            "example": false
                          },
                          "avg_fulfillment_time": {
                            "description": "Average number of days for order to be fulfilled",
                            "type": "number",
                            "example": 4.3
                          },
                          "description": {
                            "description": "Product description",
                            "type": "string"
                          },
                          "techniques": {
                            "description": "Available techniques",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "AvailableTechnique",
                              "properties": {
                                "key": {
                                  "description": "The technique key to be used in the API",
                                  "type": "string",
                                  "example": "DTG"
                                },
                                "display_name": {
                                  "description": "The human-readable technique name",
                                  "type": "string",
                                  "example": "DTG printing"
                                },
                                "is_default": {
                                  "description": "Whether the technique is the default one",
                                  "type": "boolean"
                                }
                              }
                            }
                          },
                          "origin_country": {
                            "description": "The origin country for inside label",
                            "type": "string",
                            "nullable": true,
                            "example": "Nicaragua"
                          }
                        }
                      },
                      "variants": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "Variant",
                          "properties": {
                            "id": {
                              "description": "Variant ID, use this to specify the product when creating orders",
                              "type": "integer",
                              "example": 100
                            },
                            "product_id": {
                              "description": "ID of the product that this variant belongs to",
                              "type": "integer",
                              "example": 12
                            },
                            "name": {
                              "description": "Display name",
                              "type": "string",
                              "example": "Gildan 64000 Unisex Softstyle T-Shirt with Tear Away (Black / 2XL)"
                            },
                            "size": {
                              "description": "Item size",
                              "type": "string",
                              "example": "2XL"
                            },
                            "color": {
                              "description": "Item color",
                              "type": "string",
                              "example": "Black"
                            },
                            "color_code": {
                              "description": "Hexadecimal RGB color code. May not exactly reflect the real-world color",
                              "type": "string",
                              "example": "#14191e"
                            },
                            "color_code2": {
                              "description": "Secondary hexadecimal RGB color code. May not exactly reflect the real-world color",
                              "type": "string"
                            },
                            "image": {
                              "description": "URL of a preview image for this variant",
                              "type": "string",
                              "example": "https://files.cdn.printful.com/products/12/629_1517916489.jpg"
                            },
                            "price": {
                              "description": "Variant's price (can change depending on print files and optional settings)",
                              "type": "string",
                              "example": "9.85"
                            },
                            "in_stock": {
                              "description": "Stock availability of this variant",
                              "type": "boolean",
                              "example": true
                            },
                            "availability_regions": {
                              "description": "Map of [region code, region name] of regions where the variant is available for fulfillment",
                              "type": "object",
                              "additionalProperties": true,
                              "example": {
                                "US": "USA",
                                "EU": "Europe"
                              }
                            },
                            "availability_status": {
                              "description": "Detailed stock status per region",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "AvailabilityStatus",
                                "properties": {
                                  "region": {
                                    "description": "Region code",
                                    "type": "string",
                                    "example": "US"
                                  },
                                  "status": {
                                    "description": "Stock status. Possible values include: 'in_stock' - available for fulfillment, 'stocked_on_demand' - available for fulfillment, 'discontinued' - permanently unavailable, 'out_of_stock' - temporarily unavailable",
                                    "type": "string",
                                    "example": "in_stock"
                                  }
                                }
                              }
                            },
                            "material": {
                              "description": "A list of materials this Variant is composed of",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Material",
                                "properties": {
                                  "name": {
                                    "type": "string",
                                    "description": "Material name",
                                    "example": "cotton"
                                  },
                                  "percentage": {
                                    "type": "number",
                                    "description": "Percentage of the material in the product",
                                    "example": 100
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
      "source": "curl --location --request GET 'https://api.printful.com/products/71'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\nuse Printful\\PrintfulApiClient;\n\nrequire_once __DIR__ . '../vendor/autoload.php';\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n$product = $pf->get('products/10');\n"
    }
  ]
}
```
</details>

---

### GET /products/{id}/sizes
**Summary:** Get Product Size Guide

Returns information about the size guide for a specific product.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductSizeGuideById",
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Product ID."
    },
    {
      "in": "query",
      "name": "unit",
      "schema": {
        "type": "string",
        "example": "inches,cm"
      },
      "required": false,
      "description": "A comma-separated list of measurement unit in which size tables are to be returned (`inches` or `cm`).\nThe default value is determined based on the locale country. The inches are used for United States, Liberia\nand Myanmar, for other countries the unit defaults to centimeters.\n"
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
                    "title": "ProductSizeGuide",
                    "description": "Size Guide information for the Product",
                    "required": [
                      "product_id",
                      "available_sizes",
                      "size_tables"
                    ],
                    "properties": {
                      "product_id": {
                        "description": "Product ID",
                        "type": "integer",
                        "example": 13
                      },
                      "available_sizes": {
                        "description": "The sizes available for the Product",
                        "type": "array",
                        "items": {
                          "type": "string"
                        },
                        "example": [
                          "S",
                          "M",
                          "L"
                        ]
                      },
                      "size_tables": {
                        "description": "Size tables for the product",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "SizeTable",
                          "description": "Size table for the Product",
                          "required": [
                            "type",
                            "measurements"
                          ],
                          "properties": {
                            "type": {
                              "description": "Size table type",
                              "type": "string",
                              "enum": [
                                "measure_yourself",
                                "product_measure",
                                "international"
                              ]
                            },
                            "unit": {
                              "description": "The unit the size table values are in",
                              "type": "string",
                              "enum": [
                                "inches",
                                "cm"
                              ]
                            },
                            "description": {
                              "description": "The size table description (HTML)",
                              "type": "string"
                            },
                            "image_url": {
                              "description": "The URL of an image showing the measurements",
                              "type": "string"
                            },
                            "image_description": {
                              "description": "The description of the measurement image (HTML)",
                              "type": "string"
                            },
                            "measurements": {
                              "description": "The size table measurements",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Measurement",
                                "description": "The information about a single size table measurement",
                                "required": [
                                  "type",
                                  "values"
                                ],
                                "properties": {
                                  "type_label": {
                                    "description": "Measurement type",
                                    "type": "string",
                                    "example": "Length"
                                  },
                                  "unit": {
                                    "description": "The measurement unit if it's not defined on the size table level or is different",
                                    "type": "string",
                                    "example": "none"
                                  },
                                  "values": {
                                    "description": "The measurement values for each size",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "MeasurementValue",
                                      "description": "The measurement value for a specific size",
                                      "required": [
                                        "size"
                                      ],
                                      "properties": {
                                        "size": {
                                          "description": "The size with which the value is associated",
                                          "type": "string",
                                          "example": "S"
                                        },
                                        "value": {
                                          "description": "The single value associated with a size (whether this or `min_value` and `max_value` will be present)",
                                          "type": "string",
                                          "example": "23.5"
                                        },
                                        "min_value": {
                                          "description": "The lower boundary of the value range (whether this and `max_value` or `value` will be present)",
                                          "type": "string",
                                          "example": "20"
                                        },
                                        "max_value": {
                                          "description": "The upper boundary of the value range (whether this and `min_value` or `value` will be present)",
                                          "type": "string",
                                          "example": "20"
                                        }
                                      }
                                    }
                                  }
                                }
                              }
                            }
                          }
                        },
                        "example": [
                          {
                            "type": "measure_yourself",
                            "unit": "inches",
                            "description": "<p>Measurements are provided by suppliers.<br /><br />US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\\n<p>Product measurements may vary by up to 2\\\" (5 cm).&nbsp;</p>",
                            "image_url": "https://s3.staging.printful.com/upload/measure-yourself/6a/6a4fe322f592f2b91d5a735d7ff8d1c0_t?v=1652962720",
                            "image_description": "<h6><strong>A Length</strong></h6>\\n<p dir=\\\"ltr\\\"><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">o the bottom of the shirt.</span></p>\\n<h6>B Chest</h6>\\n<p dir=\\\"ltr\\\">Measure yourself around the fullest part of your chest. Keep the tape measure horizontal.</p>",
                            "measurements": [
                              {
                                "type_label": "Length",
                                "values": [
                                  {
                                    "size": "S",
                                    "value": "24"
                                  },
                                  {
                                    "size": "M",
                                    "value": "26"
                                  },
                                  {
                                    "size": "L",
                                    "value": "28"
                                  }
                                ]
                              },
                              {
                                "type_label": "Chest",
                                "values": [
                                  {
                                    "size": "S",
                                    "min_value": "14",
                                    "max_value": "16"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "18",
                                    "max_value": "20"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "22",
                                    "max_value": "24"
                                  }
                                ]
                              }
                            ]
                          },
                          {
                            "type": "product_measure",
                            "unit": "inches",
                            "description": "<p dir=\\\"ltr\\\">Measurements are provided by our suppliers. Product measurements may vary by up to 2\\\" (5 cm).</p>\\n<p dir=\\\"ltr\\\">US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\\n<p dir=\\\"ltr\\\">Pro tip! Measure one of your products at home and compare with the measurements you see in this guide.</p>",
                            "image_url": "https://s3.staging.printful.com/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
                            "image_description": "<h6><strong>A Length</strong></h6>\\n<p dir=\\\"ltr\\\"><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">o the bottom of the shirt.</span></p>\\n<h6>B Width</h6>\\n<p dir=\\\"ltr\\\">Place the end of the tape at the seam under the sleeve and pull the tape measure across the shirt to the seam under the opposite sleeve.</p>",
                            "measurements": [
                              {
                                "type_label": "Length",
                                "values": [
                                  {
                                    "size": "S",
                                    "value": "24"
                                  },
                                  {
                                    "size": "M",
                                    "value": "26"
                                  },
                                  {
                                    "size": "L",
                                    "value": "28"
                                  }
                                ]
                              },
                              {
                                "type_label": "Width",
                                "values": [
                                  {
                                    "size": "S",
                                    "min_value": "14",
                                    "max_value": "16"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "18",
                                    "max_value": "20"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "22",
                                    "max_value": "24"
                                  }
                                ]
                              }
                            ]
                          },
                          {
                            "type": "measure_yourself",
                            "unit": "cm",
                            "description": "<p>Measurements are provided by suppliers.<br /><br />US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\\n<p>Product measurements may vary by up to 2\\\" (5 cm).&nbsp;</p>",
                            "image_url": "https://s3.staging.printful.com/upload/measure-yourself/6a/6a4fe322f592f2b91d5a735d7ff8d1c0_t?v=1652962720",
                            "image_description": "<h6><strong>A Length</strong></h6>\\n<p dir=\\\"ltr\\\"><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">o the bottom of the shirt.</span></p>\\n<h6>B Chest</h6>\\n<p dir=\\\"ltr\\\">Measure yourself around the fullest part of your chest. Keep the tape measure horizontal.</p>",
                            "measurements": [
                              {
                                "type_label": "Length",
                                "values": [
                                  {
                                    "size": "S",
                                    "value": "60.96"
                                  },
                                  {
                                    "size": "M",
                                    "value": "66.04"
                                  },
                                  {
                                    "size": "L",
                                    "value": "71.12"
                                  }
                                ]
                              },
                              {
                                "type_label": "Chest",
                                "values": [
                                  {
                                    "size": "S",
                                    "min_value": "35.56",
                                    "max_value": "40.64"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "45.72",
                                    "max_value": "50.80"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "55.88",
                                    "max_value": "60.96"
                                  }
                                ]
                              }
                            ]
                          },
                          {
                            "type": "product_measure",
                            "unit": "cm",
                            "description": "<p dir=\\\"ltr\\\">Measurements are provided by our suppliers. Product measurements may vary by up to 2\\\" (5 cm).</p>\\n<p dir=\\\"ltr\\\">US customers should order a size up as the EU sizes for this supplier correspond to a smaller size in the US market.</p>\\n<p dir=\\\"ltr\\\">Pro tip! Measure one of your products at home and compare with the measurements you see in this guide.</p>",
                            "image_url": "https://s3.staging.printful.com/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
                            "image_description": "<h6><strong>A Length</strong></h6>\\n<p dir=\\\"ltr\\\"><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">Place the end of the tape beside the collar at the top of the tee (Highest Point Shoulder). Pull the tape measure t</span><span id=\\\"docs-internal-guid-a3ac3082-7fff-5f98-2623-3eb38d5f43a1\\\">o the bottom of the shirt.</span></p>\\n<h6>B Width</h6>\\n<p dir=\\\"ltr\\\">Place the end of the tape at the seam under the sleeve and pull the tape measure across the shirt to the seam under the opposite sleeve.</p>",
                            "measurements": [
                              {
                                "type_label": "Length",
                                "values": [
                                  {
                                    "size": "S",
                                    "value": "60.96"
                                  },
                                  {
                                    "size": "M",
                                    "value": "66.04"
                                  },
                                  {
                                    "size": "L",
                                    "value": "71.12"
                                  }
                                ]
                              },
                              {
                                "type_label": "Width",
                                "values": [
                                  {
                                    "size": "S",
                                    "min_value": "35.56",
                                    "max_value": "40.64"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "45.72",
                                    "max_value": "50.80"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "55.88",
                                    "max_value": "60.96"
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
                                    "min_value": "8",
                                    "max_value": "10"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "12",
                                    "max_value": "14"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "16",
                                    "max_value": "18"
                                  }
                                ]
                              },
                              {
                                "type_label": "EU size",
                                "values": [
                                  {
                                    "size": "S",
                                    "min_value": "38",
                                    "max_value": "39"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "40",
                                    "max_value": "41"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "42",
                                    "max_value": "43"
                                  }
                                ]
                              },
                              {
                                "type_label": "UK size",
                                "values": [
                                  {
                                    "size": "S",
                                    "min_value": "4",
                                    "max_value": "6"
                                  },
                                  {
                                    "size": "M",
                                    "min_value": "8",
                                    "max_value": "10"
                                  },
                                  {
                                    "size": "L",
                                    "min_value": "12",
                                    "max_value": "14"
                                  }
                                ]
                              }
                            ]
                          }
                        ]
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
      "source": "curl --location --request GET 'https://api.printful.com/products/71/sizes'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\nuse Printful\\PrintfulApiClient;\n\nrequire_once __DIR__ . '../vendor/autoload.php';\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n$product = $pf->get('products/10/sizes');"
    }
  ]
}
```
</details>

---

### GET /categories
**Summary:** Get Categories

Returns list of Catalog Categories available in the Printful

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCategories",
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
                    "type": "array",
                    "items": {
                      "type": "object",
                      "title": "Category",
                      "description": "Information about the Category",
                      "properties": {
                        "id": {
                          "description": "Category ID",
                          "type": "integer",
                          "example": 24
                        },
                        "parent_id": {
                          "description": "ID of the parent Category. If there is no parent Category, 0 is returned.",
                          "type": "integer",
                          "example": 6
                        },
                        "image_url": {
                          "description": "The URL of the Category image",
                          "type": "string",
                          "example": "https://s3.staging.printful.com/upload/catalog_category/b1/b1513c82696405fcc316fc611c57f132_t?v=1646395980"
                        },
                        "size": {
                          "description": "The size of the category image",
                          "type": "string",
                          "enum": [
                            "small",
                            "medium",
                            "large"
                          ]
                        },
                        "title": {
                          "description": "Category title",
                          "type": "string",
                          "example": "T-Shirts"
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
      "source": "curl --location --request GET 'https://api.printful.com/categories'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\nuse Printful\\PrintfulApiClient;\n\nrequire_once __DIR__ . '../vendor/autoload.php';\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n$categories = $pf->get('categories');\n"
    }
  ]
}
```
</details>

---

### GET /categories/{id}
**Summary:** Get Category

Returns information about a specific category.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCategoryById",
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Category ID"
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
                    "title": "Category",
                    "description": "Information about the Category",
                    "properties": {
                      "id": {
                        "description": "Category ID",
                        "type": "integer",
                        "example": 24
                      },
                      "parent_id": {
                        "description": "ID of the parent Category. If there is no parent Category, 0 is returned.",
                        "type": "integer",
                        "example": 6
                      },
                      "image_url": {
                        "description": "The URL of the Category image",
                        "type": "string",
                        "example": "https://s3.staging.printful.com/upload/catalog_category/b1/b1513c82696405fcc316fc611c57f132_t?v=1646395980"
                      },
                      "size": {
                        "description": "The size of the category image",
                        "type": "string",
                        "enum": [
                          "small",
                          "medium",
                          "large"
                        ]
                      },
                      "title": {
                        "description": "Category title",
                        "type": "string",
                        "example": "T-Shirts"
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
      "source": "curl --location --request GET 'https://api.printful.com/categories/24'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\nuse Printful\\PrintfulApiClient;\n\nrequire_once __DIR__ . '../vendor/autoload.php';\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n$category = $pf->get('categories/24');\n"
    }
  ]
}
```
</details>

---


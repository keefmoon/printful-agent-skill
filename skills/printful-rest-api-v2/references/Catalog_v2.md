# Catalog v2

## What is a catalog product?
A catalog product refers to a blank product and its variants. Each variant can be used as the base for custom designs. Catalog product consists of multiple catalog variants each of which might be a combination of size and color or it might be a variation with e.g. tear away label. The entire hierarchy is shown in the diagram below:

[<img src="images/catalog/catalog_product_diagram.png?center" width="300" alt="Catalog product diagram"/>](images/catalog/catalog_product_diagram.png)

## What is a catalog variant?
Catalog variant might be a combination of size and color associated with the catalog product or additional variation of a product with certain features like tear away label. It can be thought of as an actual physical blank product. For example T-shirt Bella Canvas 3001 in White color and size M is considered to be a single variant. Catalog variants can have different availability as different regions could have different stock of some product colors and sizes.

[<img src="images/catalog/catalog_variant_diagram.png?center" width="300" alt="Catalog variant diagram"/>](images/catalog/catalog_variant_diagram.png)

Printful has a substantial catalog of blank Products and Variants. A Product can describe a specific type, model and manufacturer of the item, while the Variant specifies the more detailed attributes of the product like the exact size/color of a
T-shirt or the dimensions of a poster. Moreover, each item in the Printful Catalog has a unique Variant ID. When
managing orders, you will need to specify the Variant ID of the specific blank item, hence you can use this API resource
to find the needed Variant ID.

<div class="alert alert-info">
It is critically important to always refer to the Variant IDs (<strong>NOT Product IDs</strong>) when creating products or orders. Mixing up and using the Product ID instead of the Variant ID can lead to an entirely different product created or item ordered. 
The Product entity is only meant to allow of easier browsing of what Printful offers.
</div>

You can also use this API resource to find out the types of print files. A product can be configured for as well as the
additional price each print file would cost (e.g. the back print or inside label print for T-shirts). Moreover, some
product types allow for additional options (e.g. embroidery type and thread colors) – these options are listed in the
responses as well.

**Important**: Jewelry products are not supported via API.

**Rate limiting**: Up to 120 requests per 60 seconds. A 60 seconds lockout is applied if
request count is exceeded.

## Catalog general information
### General introduction

This resource contains the list of all the products that are available in Printful for purchase and fulfillment. Apart from basic product data and availability it also contains the information about:
* [Product Prices](#operation/getProductPricesById)
* [Product Size Guides](#operation/getProductSizeGuideById)
* [Product Images](#operation/getProductImagesById)

[<img src="images/catalog/catalog_diagram.png?center" width="500" alt="Catalog diagram"/>](images/catalog/catalog_diagram.png)

### Filtering options

Filters can be used to search for particular set of products based on the provided values. Multiple filters can be used at the same time.

| Filtering option name | Description                                                    | Where to find                                                              |
|-----------------------|----------------------------------------------------------------|----------------------------------------------------------------------------|
| colors                | Filters products by variants containing specified colors       | `data[].color` [Catalog variants](#operation/getProductVariantsById)       |
| new                   | Filters products that are new                                  | -                                                                          |
| placements            | Filters products by available placements specified in the list | `data[].placements[].placement` [Catalog products](#operation/getProducts) |
| category_ids          | Filters products by specified product categories               | `data[].id` [Catalog categories](#operation/getCategories)                 |
| techniques            | Filters products by specified techniques                       | `data[].techniques[].key` [Catalog products](#operation/getProducts)       |
| types                 | Filters products by specified types of product e.g. T-Shirt.   | `data[].type` [Catalog products](#operation/getProducts)                   |
| bestseller            | Filters products that are bestsellers                          | -                                                                          |

### Sorting

Requires providing two parameters in the URL: `sort_type`
And `sort_direction`. Sort type determines a type of sorting that will be used e.g. sort by price and the sort direction determines if the result should be sorted ascending or descending.
Only one sort type can be used at a time.

| Sorting option name | Description                                                |
|---------------------|------------------------------------------------------------|
| new                 | Sorts resulting catalog products starting with new ones    |
| rating              | Sorts resulting catalog products by their user-rating      |
| price               | Sorts resulting catalog products by their base price       |
| bestseller          | Sorts resulting catalog products starting with bestsellers |

### DSR

* **What is DSR?** – DSR (Default Selling Region) is an option that lets you limit the results of a product catalog to products that could be shipped within a specified region. When choosing Worldwide, we'll display the entire Product Catalog. Delivery times depend on end-customer location and selected product.
* **Where DSR can be set and how it affects response?** – DSR can be currently set in [Catalog products](#operation/getProducts) and [Product Prices](#operation/getProductPricesById).
* **Available DSRs**

| Region name              | Region name value          |
|--------------------------|----------------------------|
| Worldwide                | `worldwide`                |
| North America            | `north_america`            |
| Canada                   | `canada`                   |
| Europe                   | `europe`                   |
| Spain                    | `spain`                    |
| Latvia                   | `latvia`                   |
| United Kingdom           | `uk`                       |
| France                   | `france`                   |
| Germany                  | `germany`                  |
| Australia                | `australia`                |
| Japan                    | `japan`                    |
| New Zealand              | `new_zealand`              |
| Italy                    | `italy`                    |
| Brazil                   | `brazil`                   |
| Southeast Asia           | `southeast_asia`           |
| Republic of Korea        | `republic_of_korea`        |

### Size guides

[<img src="images/catalog/size_guides_diagram.png?center" width="600" alt="Size guides diagram"/>](images/catalog/size_guides_diagram.png)

[Get Product Size Guide](#operation/getProductSizeGuideById) endpoint will return size guides for the specified product.

There are three types of size tables available, as described by the following table:

| Table type                    | API name           | Description                                                                                 |
|-------------------------------|--------------------|---------------------------------------------------------------------------------------------|
| Measure yourself              | `measure_yourself` | Measurements of the product to measure the body provided by the supplier.                   |
| Product measurements          | `product_measure`  | Measurements of the product provided by the supplier.                                       |
| International size conversion | `international`    | International size conversion – e.g. US, EU or UK sizes corresponding to the product sizes. |

Not each table type might be available for the selected product.
[See examples](#tag/Examples/Catalog-API-examples/Using-size-guides)

## Categories

Catalog categories are used to group products with similar characteristics such as t-shirts, hats etc.

Categories have purely informational purposes and cannot be edited by the customers. Categories can be nested e.g.
* Men’s clothing
  * All shirts
    * T-Shirts
    * Polo shirts
  * Jackets & vests

Apart from display purposes they can be also used for filtering the products in the API [see Filtering options](#filtering-options)


## Endpoints

### GET /v2/catalog-products
**Summary:** Retrieve a list of catalog products

This endpoint retrieves a list of the products available in Printful's catalog. The list is paginated and can be filtered using various filters. The information returned includes details on how each product can be designed, such as the available placements, techniques, and additional options.
For a visual representation of the design data, please see the following diagram:
[<img src="images/catalog/design_data_diagram.png?center" width="700" alt="Design data diagram"/>](images/catalog/design_data_diagram.png)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProducts",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "category_ids",
      "schema": {
        "type": "array",
        "items": {
          "type": "integer"
        }
      },
      "explode": false,
      "examples": {
        "oneID": {
          "summary": "Using a single category ID",
          "value": [
            4
          ]
        },
        "multipleIDs": {
          "summary": "Multiple category IDs",
          "value": [
            4,
            7
          ]
        }
      },
      "required": false,
      "description": "One or more category IDs to return only products in those categories. The IDs can be found in the response of the\noperation _[Get Categories](#operation/getCategories)_.\n"
    },
    {
      "in": "query",
      "name": "colors",
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "explode": false,
      "examples": {
        "oneColor": {
          "summary": "Grey.",
          "value": [
            "grey"
          ]
        },
        "oneUrlEncodedColor": {
          "summary": "Heather grey color.",
          "value": [
            "heather%20grey"
          ]
        },
        "multipleColors": {
          "summary": "Grey or heather grey color",
          "value": [
            "grey",
            "heather%20grey"
          ]
        }
      },
      "required": false,
      "description": "One or more color names to return only products with variants of one the those colors.\n"
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
      "name": "new",
      "schema": {
        "type": "boolean",
        "default": false
      },
      "examples": {
        "true": {
          "summary": "Return only new products",
          "value": true
        }
      },
      "required": false,
      "description": "If true only new Products will be returned."
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
      "in": "query",
      "name": "placements",
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "explode": false,
      "examples": {
        "onePlacement": {
          "summary": "Return products with variants that have front placement.",
          "value": [
            "front"
          ]
        },
        "multiplePlacements": {
          "summary": "Return products with variants that have front or back placement.",
          "value": [
            "front",
            "back"
          ]
        }
      },
      "required": false,
      "description": "One or more identifiers of a placement to return only products with variants that have that placement. The complete list of placements can be found [here](https://developers.printful.com/docs/#tag/Common/Placements)."
    },
    {
      "in": "query",
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
    },
    {
      "in": "query",
      "name": "sort_direction",
      "schema": {
        "type": "string",
        "default": "descending",
        "enum": [
          "ascending",
          "descending"
        ]
      },
      "required": false,
      "description": "This parameter only is used if sort_type is also present and it changes the order of the returned products.\nThe exact meaning varies depending on the value of `sort_type`:\n* `sort_type=new`\n  * ascending sorts from newest to oldest.\n  * descending sorts from oldest to newest.\n* `sort_type=rating`\n\t* ascending from lowest to highest rated.\n\t* descending from highest to lowest rated.\n* `sort_type=price`\n\t* ascending from lowest to highest price.\n\t* descending from highest to lowest price.\n* `sort_type=bestseller`\n  * ascending from non bestsellers to bestsellers.\n  * descending from bestsellers to non bestsellers.\n"
    },
    {
      "in": "query",
      "name": "sort_type",
      "schema": {
        "type": "string",
        "enum": [
          "new",
          "rating",
          "price",
          "bestseller"
        ]
      },
      "required": false,
      "description": "The sorting strategy to use when sorting the result. When it's not present, no specific order is guaranteed.\n"
    },
    {
      "in": "query",
      "name": "techniques",
      "schema": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "dtg",
            "digital",
            "cut-sew",
            "uv",
            "embroidery",
            "sublimation",
            "dtfilm"
          ]
        }
      },
      "explode": false,
      "examples": {
        "oneTechnique": {
          "summary": "Products with variants that can be printed using the technique digital.",
          "value": [
            "digital"
          ]
        },
        "multipleTechniques": {
          "summary": "Products with variants that can be printed using the techniques digital or embroidery.",
          "value": [
            "digital",
            "embroidery"
          ]
        }
      },
      "required": false,
      "description": "One or more techniques to return only products with variants that can be printed using one of the techniques."
    },
    {
      "in": "query",
      "name": "destination_country",
      "schema": {
        "type": "string"
      },
      "required": false,
      "description": "The ISO 3166-1 alpha-2 country code for the destination country. Only products shippable to the specified country will be returned.\n"
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                  "title": "Product",
                  "description": "Information about the Catalog Product",
                  "allOf": [
                    {
                      "type": "object",
                      "title": "Product",
                      "required": [
                        "id",
                        "main_category_id",
                        "type",
                        "name",
                        "brand",
                        "model",
                        "image",
                        "variant_count",
                        "is_discontinued",
                        "description",
                        "sizes",
                        "colors",
                        "techniques",
                        "placements",
                        "product_options",
                        "_links"
                      ],
                      "description": "Information about the Product",
                      "properties": {
                        "id": {
                          "description": "Product ID",
                          "type": "integer",
                          "example": 362
                        },
                        "main_category_id": {
                          "description": "Main category of product",
                          "type": "integer",
                          "example": 24
                        },
                        "type": {
                          "description": "Product type",
                          "type": "string",
                          "example": "T-Shirt"
                        },
                        "name": {
                          "description": "Product name",
                          "type": "string",
                          "example": "Unisex Organic T-Shirt | Econscious EC1000"
                        },
                        "brand": {
                          "description": "Brand name",
                          "type": "string",
                          "nullable": true,
                          "example": "Econscious"
                        },
                        "model": {
                          "description": "Model name",
                          "type": "string",
                          "nullable": true,
                          "example": "Unisex Organic T-Shirt"
                        },
                        "image": {
                          "description": "URL of a sample image for this product",
                          "type": "string",
                          "example": "https://files.cdn.printful.com/products/12/product_1550594502.jpg"
                        },
                        "variant_count": {
                          "description": "Number of available variants for this product",
                          "type": "integer",
                          "example": 10
                        },
                        "is_discontinued": {
                          "description": "Product is discontinued and can no longer be ordered",
                          "type": "boolean",
                          "example": false
                        },
                        "description": {
                          "description": "Product description",
                          "type": "string"
                        },
                        "sizes": {
                          "description": "Product sizes",
                          "type": "array",
                          "items": {
                            "type": "string",
                            "example": "M"
                          }
                        },
                        "colors": {
                          "description": "Product colors",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "required": [
                              "name",
                              "value"
                            ],
                            "properties": {
                              "name": {
                                "type": "string",
                                "description": "Name of the color",
                                "example": "Athletic Heather"
                              },
                              "value": {
                                "type": "string",
                                "description": "Value of the color in HEX",
                                "example": "#cbcbcb"
                              }
                            }
                          }
                        },
                        "techniques": {
                          "description": "Product's techniques",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "Technique",
                            "description": "Information about the available Product's technique",
                            "required": [
                              "key",
                              "display_name",
                              "is_default"
                            ],
                            "properties": {
                              "key": {
                                "description": "Technique key",
                                "type": "string",
                                "example": "embroidery"
                              },
                              "display_name": {
                                "description": "Technique display name",
                                "type": "string",
                                "example": "Embroidery"
                              },
                              "is_default": {
                                "description": "This is the default product technique",
                                "type": "boolean",
                                "example": true
                              }
                            }
                          }
                        },
                        "placements": {
                          "description": "Product's design placements",
                          "type": "array",
                          "items": {
                            "allOf": [
                              {
                                "required": [
                                  "print_area_width",
                                  "print_area_height",
                                  "placement_options",
                                  "conflicting_placements"
                                ]
                              },
                              {
                                "type": "object",
                                "title": "DesignPlacement",
                                "description": "Information about the product placements that can be used to specify the design placement",
                                "required": [
                                  "placement",
                                  "technique",
                                  "layers"
                                ],
                                "properties": {
                                  "placement": {
                                    "description": "Name of placement that can be used to place design in a correct spot on a product",
                                    "type": "string",
                                    "example": "back"
                                  },
                                  "technique": {
                                    "description": "Indicates technique for which the placements are available",
                                    "type": "string",
                                    "example": "embroidery"
                                  },
                                  "layers": {
                                    "description": "Available layers for that product",
                                    "type": "array",
                                    "items": {
                                      "oneOf": [
                                        {
                                          "type": "object",
                                          "required": [
                                            "type"
                                          ],
                                          "properties": {
                                            "type": {
                                              "description": "File type layer",
                                              "type": "string",
                                              "example": "file"
                                            },
                                            "layer_options": {
                                              "description": "Possible layer options",
                                              "type": "array",
                                              "items": {
                                                "type": "object",
                                                "title": "CatalogOption",
                                                "description": "Catalog option definition",
                                                "required": [
                                                  "name",
                                                  "techniques",
                                                  "type",
                                                  "values"
                                                ],
                                                "properties": {
                                                  "name": {
                                                    "description": "Option identifier",
                                                    "type": "string",
                                                    "example": "3d_puff"
                                                  },
                                                  "techniques": {
                                                    "description": "Available techniques for option",
                                                    "type": "array",
                                                    "items": {
                                                      "type": "string",
                                                      "example": "embroidery"
                                                    }
                                                  },
                                                  "type": {
                                                    "description": "Type of accepted value",
                                                    "type": "string",
                                                    "example": "boolean"
                                                  },
                                                  "values": {
                                                    "description": "List of available option values.",
                                                    "type": "array",
                                                    "example": [
                                                      true,
                                                      false
                                                    ]
                                                  }
                                                }
                                              },
                                              "example": [
                                                {
                                                  "name": "3d_puff",
                                                  "techniques": [
                                                    "embroidery"
                                                  ],
                                                  "type": "boolean",
                                                  "values": [
                                                    true,
                                                    false
                                                  ]
                                                }
                                              ]
                                            }
                                          }
                                        }
                                      ]
                                    }
                                  },
                                  "placement_options": {
                                    "description": "Possible placement options",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "CatalogOption",
                                      "description": "Catalog option definition",
                                      "required": [
                                        "name",
                                        "techniques",
                                        "type",
                                        "values"
                                      ],
                                      "properties": {
                                        "name": {
                                          "description": "Option identifier",
                                          "type": "string",
                                          "example": "3d_puff"
                                        },
                                        "techniques": {
                                          "description": "Available techniques for option",
                                          "type": "array",
                                          "items": {
                                            "type": "string",
                                            "example": "embroidery"
                                          }
                                        },
                                        "type": {
                                          "description": "Type of accepted value",
                                          "type": "string",
                                          "example": "boolean"
                                        },
                                        "values": {
                                          "description": "List of available option values.",
                                          "type": "array",
                                          "example": [
                                            true,
                                            false
                                          ]
                                        }
                                      }
                                    },
                                    "example": [
                                      {
                                        "name": "unlimited_color",
                                        "techniques": [
                                          "embroidery"
                                        ],
                                        "type": "boolean",
                                        "values": [
                                          true,
                                          false
                                        ]
                                      }
                                    ]
                                  }
                                }
                              },
                              {
                                "properties": {
                                  "conflicting_placements": {
                                    "description": "List of conflicting placements. Used to determine which placements can be used together.",
                                    "type": "array",
                                    "example": [
                                      "back",
                                      "label_inside"
                                    ],
                                    "items": {
                                      "type": "string"
                                    }
                                  }
                                }
                              }
                            ]
                          }
                        },
                        "product_options": {
                          "description": "Possible product options",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "CatalogOption",
                            "description": "Catalog option definition",
                            "required": [
                              "name",
                              "techniques",
                              "type",
                              "values"
                            ],
                            "properties": {
                              "name": {
                                "description": "Option identifier",
                                "type": "string",
                                "example": "3d_puff"
                              },
                              "techniques": {
                                "description": "Available techniques for option",
                                "type": "array",
                                "items": {
                                  "type": "string",
                                  "example": "embroidery"
                                }
                              },
                              "type": {
                                "description": "Type of accepted value",
                                "type": "string",
                                "example": "boolean"
                              },
                              "values": {
                                "description": "List of available option values.",
                                "type": "array",
                                "example": [
                                  true,
                                  false
                                ]
                              }
                            }
                          },
                          "example": [
                            {
                              "name": "stitch_color",
                              "techniques": [
                                "cut-sew"
                              ],
                              "type": "string",
                              "values": [
                                "White",
                                "Black"
                              ]
                            }
                          ]
                        }
                      }
                    },
                    {
                      "type": "object",
                      "properties": {
                        "_links": {
                          "type": "object",
                          "title": "Product links",
                          "description": "HATEOAS links",
                          "required": [
                            "self",
                            "variants",
                            "categories",
                            "product_prices",
                            "product_sizes",
                            "product_images",
                            "availability"
                          ],
                          "properties": {
                            "self": {
                              "description": "Link to same resource",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71"
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
                            "variants": {
                              "description": "Link to product variants",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants"
                              },
                              "allOf": [
                                {
                                  "$ref": "circular reference to #/components/schemas/HateoasLink"
                                }
                              ]
                            },
                            "categories": {
                              "description": "Link to product categories",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71/categories"
                              },
                              "allOf": [
                                {
                                  "$ref": "circular reference to #/components/schemas/HateoasLink"
                                }
                              ]
                            },
                            "product_prices": {
                              "description": "Link to product prices",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71/prices"
                              },
                              "allOf": [
                                {
                                  "$ref": "circular reference to #/components/schemas/HateoasLink"
                                }
                              ]
                            },
                            "product_sizes": {
                              "description": "Link product size guides",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71/sizes"
                              },
                              "allOf": [
                                {
                                  "$ref": "circular reference to #/components/schemas/HateoasLink"
                                }
                              ]
                            },
                            "product_images": {
                              "description": "Link product images",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71/images"
                              },
                              "allOf": [
                                {
                                  "$ref": "circular reference to #/components/schemas/HateoasLink"
                                }
                              ]
                            },
                            "availability": {
                              "description": "Link to product stock availability endpoint",
                              "type": "object",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/catalog-products/71/availability"
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
                  ]
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
                "title": "Product links",
                "description": "HATEOAS links",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
                    "type": "object",
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products"
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
                    "type": "object",
                    "description": "Link to next resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products?offset=20"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "first": {
                    "type": "object",
                    "description": "Link to first resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "last": {
                    "type": "object",
                    "description": "Link to the last resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products?offset=280"
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

### GET /v2/catalog-products/{id}
**Summary:** Retrieve a single catalog product

Returns information about a single specified catalog product. [See catalog product](#tag/Catalog-v2/What-is-a-catalog-product)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                "title": "Product",
                "description": "Information about the Catalog Product",
                "allOf": [
                  {
                    "type": "object",
                    "title": "Product",
                    "required": [
                      "id",
                      "main_category_id",
                      "type",
                      "name",
                      "brand",
                      "model",
                      "image",
                      "variant_count",
                      "is_discontinued",
                      "description",
                      "sizes",
                      "colors",
                      "techniques",
                      "placements",
                      "product_options",
                      "_links"
                    ],
                    "description": "Information about the Product",
                    "properties": {
                      "id": {
                        "description": "Product ID",
                        "type": "integer",
                        "example": 362
                      },
                      "main_category_id": {
                        "description": "Main category of product",
                        "type": "integer",
                        "example": 24
                      },
                      "type": {
                        "description": "Product type",
                        "type": "string",
                        "example": "T-Shirt"
                      },
                      "name": {
                        "description": "Product name",
                        "type": "string",
                        "example": "Unisex Organic T-Shirt | Econscious EC1000"
                      },
                      "brand": {
                        "description": "Brand name",
                        "type": "string",
                        "nullable": true,
                        "example": "Econscious"
                      },
                      "model": {
                        "description": "Model name",
                        "type": "string",
                        "nullable": true,
                        "example": "Unisex Organic T-Shirt"
                      },
                      "image": {
                        "description": "URL of a sample image for this product",
                        "type": "string",
                        "example": "https://files.cdn.printful.com/products/12/product_1550594502.jpg"
                      },
                      "variant_count": {
                        "description": "Number of available variants for this product",
                        "type": "integer",
                        "example": 10
                      },
                      "is_discontinued": {
                        "description": "Product is discontinued and can no longer be ordered",
                        "type": "boolean",
                        "example": false
                      },
                      "description": {
                        "description": "Product description",
                        "type": "string"
                      },
                      "sizes": {
                        "description": "Product sizes",
                        "type": "array",
                        "items": {
                          "type": "string",
                          "example": "M"
                        }
                      },
                      "colors": {
                        "description": "Product colors",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "required": [
                            "name",
                            "value"
                          ],
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Name of the color",
                              "example": "Athletic Heather"
                            },
                            "value": {
                              "type": "string",
                              "description": "Value of the color in HEX",
                              "example": "#cbcbcb"
                            }
                          }
                        }
                      },
                      "techniques": {
                        "description": "Product's techniques",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "Technique",
                          "description": "Information about the available Product's technique",
                          "required": [
                            "key",
                            "display_name",
                            "is_default"
                          ],
                          "properties": {
                            "key": {
                              "description": "Technique key",
                              "type": "string",
                              "example": "embroidery"
                            },
                            "display_name": {
                              "description": "Technique display name",
                              "type": "string",
                              "example": "Embroidery"
                            },
                            "is_default": {
                              "description": "This is the default product technique",
                              "type": "boolean",
                              "example": true
                            }
                          }
                        }
                      },
                      "placements": {
                        "description": "Product's design placements",
                        "type": "array",
                        "items": {
                          "allOf": [
                            {
                              "required": [
                                "print_area_width",
                                "print_area_height",
                                "placement_options",
                                "conflicting_placements"
                              ]
                            },
                            {
                              "type": "object",
                              "title": "DesignPlacement",
                              "description": "Information about the product placements that can be used to specify the design placement",
                              "required": [
                                "placement",
                                "technique",
                                "layers"
                              ],
                              "properties": {
                                "placement": {
                                  "description": "Name of placement that can be used to place design in a correct spot on a product",
                                  "type": "string",
                                  "example": "back"
                                },
                                "technique": {
                                  "description": "Indicates technique for which the placements are available",
                                  "type": "string",
                                  "example": "embroidery"
                                },
                                "layers": {
                                  "description": "Available layers for that product",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "required": [
                                          "type"
                                        ],
                                        "properties": {
                                          "type": {
                                            "description": "File type layer",
                                            "type": "string",
                                            "example": "file"
                                          },
                                          "layer_options": {
                                            "description": "Possible layer options",
                                            "type": "array",
                                            "items": {
                                              "type": "object",
                                              "title": "CatalogOption",
                                              "description": "Catalog option definition",
                                              "required": [
                                                "name",
                                                "techniques",
                                                "type",
                                                "values"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "3d_puff"
                                                },
                                                "techniques": {
                                                  "description": "Available techniques for option",
                                                  "type": "array",
                                                  "items": {
                                                    "type": "string",
                                                    "example": "embroidery"
                                                  }
                                                },
                                                "type": {
                                                  "description": "Type of accepted value",
                                                  "type": "string",
                                                  "example": "boolean"
                                                },
                                                "values": {
                                                  "description": "List of available option values.",
                                                  "type": "array",
                                                  "example": [
                                                    true,
                                                    false
                                                  ]
                                                }
                                              }
                                            },
                                            "example": [
                                              {
                                                "name": "3d_puff",
                                                "techniques": [
                                                  "embroidery"
                                                ],
                                                "type": "boolean",
                                                "values": [
                                                  true,
                                                  false
                                                ]
                                              }
                                            ]
                                          }
                                        }
                                      }
                                    ]
                                  }
                                },
                                "placement_options": {
                                  "description": "Possible placement options",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "CatalogOption",
                                    "description": "Catalog option definition",
                                    "required": [
                                      "name",
                                      "techniques",
                                      "type",
                                      "values"
                                    ],
                                    "properties": {
                                      "name": {
                                        "description": "Option identifier",
                                        "type": "string",
                                        "example": "3d_puff"
                                      },
                                      "techniques": {
                                        "description": "Available techniques for option",
                                        "type": "array",
                                        "items": {
                                          "type": "string",
                                          "example": "embroidery"
                                        }
                                      },
                                      "type": {
                                        "description": "Type of accepted value",
                                        "type": "string",
                                        "example": "boolean"
                                      },
                                      "values": {
                                        "description": "List of available option values.",
                                        "type": "array",
                                        "example": [
                                          true,
                                          false
                                        ]
                                      }
                                    }
                                  },
                                  "example": [
                                    {
                                      "name": "unlimited_color",
                                      "techniques": [
                                        "embroidery"
                                      ],
                                      "type": "boolean",
                                      "values": [
                                        true,
                                        false
                                      ]
                                    }
                                  ]
                                }
                              }
                            },
                            {
                              "properties": {
                                "conflicting_placements": {
                                  "description": "List of conflicting placements. Used to determine which placements can be used together.",
                                  "type": "array",
                                  "example": [
                                    "back",
                                    "label_inside"
                                  ],
                                  "items": {
                                    "type": "string"
                                  }
                                }
                              }
                            }
                          ]
                        }
                      },
                      "product_options": {
                        "description": "Possible product options",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "CatalogOption",
                          "description": "Catalog option definition",
                          "required": [
                            "name",
                            "techniques",
                            "type",
                            "values"
                          ],
                          "properties": {
                            "name": {
                              "description": "Option identifier",
                              "type": "string",
                              "example": "3d_puff"
                            },
                            "techniques": {
                              "description": "Available techniques for option",
                              "type": "array",
                              "items": {
                                "type": "string",
                                "example": "embroidery"
                              }
                            },
                            "type": {
                              "description": "Type of accepted value",
                              "type": "string",
                              "example": "boolean"
                            },
                            "values": {
                              "description": "List of available option values.",
                              "type": "array",
                              "example": [
                                true,
                                false
                              ]
                            }
                          }
                        },
                        "example": [
                          {
                            "name": "stitch_color",
                            "techniques": [
                              "cut-sew"
                            ],
                            "type": "string",
                            "values": [
                              "White",
                              "Black"
                            ]
                          }
                        ]
                      }
                    }
                  },
                  {
                    "type": "object",
                    "properties": {
                      "_links": {
                        "type": "object",
                        "title": "Product links",
                        "description": "HATEOAS links",
                        "required": [
                          "self",
                          "variants",
                          "categories",
                          "product_prices",
                          "product_sizes",
                          "product_images",
                          "availability"
                        ],
                        "properties": {
                          "self": {
                            "description": "Link to same resource",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71"
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
                          "variants": {
                            "description": "Link to product variants",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants"
                            },
                            "allOf": [
                              {
                                "$ref": "circular reference to #/components/schemas/HateoasLink"
                              }
                            ]
                          },
                          "categories": {
                            "description": "Link to product categories",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71/categories"
                            },
                            "allOf": [
                              {
                                "$ref": "circular reference to #/components/schemas/HateoasLink"
                              }
                            ]
                          },
                          "product_prices": {
                            "description": "Link to product prices",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71/prices"
                            },
                            "allOf": [
                              {
                                "$ref": "circular reference to #/components/schemas/HateoasLink"
                              }
                            ]
                          },
                          "product_sizes": {
                            "description": "Link product size guides",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71/sizes"
                            },
                            "allOf": [
                              {
                                "$ref": "circular reference to #/components/schemas/HateoasLink"
                              }
                            ]
                          },
                          "product_images": {
                            "description": "Link product images",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71/images"
                            },
                            "allOf": [
                              {
                                "$ref": "circular reference to #/components/schemas/HateoasLink"
                              }
                            ]
                          },
                          "availability": {
                            "description": "Link to product stock availability endpoint",
                            "type": "object",
                            "title": "HateoasLink",
                            "example": {
                              "href": "https://api.printful.com/v2/catalog-products/71/availability"
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

### GET /v2/catalog-variants/{id}
**Summary:** Retrieve information about specific catalog variant

Returns information about single specified catalog variant. [See catalog variant](#tag/Catalog-v2/What-is-a-catalog-variant)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getVariantById",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Variant ID"
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                "type": "object",
                "title": "Variant",
                "required": [
                  "id",
                  "catalog_product_id",
                  "name",
                  "size",
                  "color",
                  "color_code",
                  "color_code2",
                  "image",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "description": "Variant ID, use this to specify the product when creating orders",
                    "type": "integer",
                    "example": 100
                  },
                  "catalog_product_id": {
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
                    "nullable": true,
                    "example": "Black"
                  },
                  "color_code": {
                    "description": "Hexadecimal RGB color code. May not exactly reflect the real-world color",
                    "type": "string",
                    "nullable": true,
                    "example": "#14191e"
                  },
                  "color_code2": {
                    "description": "Secondary hexadecimal RGB color code. May not exactly reflect the real-world color",
                    "type": "string",
                    "nullable": true
                  },
                  "placement_dimensions": {
                    "description": "A list of placement configuration objects, each specifying the layout details for a particular placement.",
                    "items": {
                      "type": "object",
                      "properties": {
                        "placement": {
                          "type": "string",
                          "description": "The name or type of the placement (e.g., \"default\").",
                          "example": "front"
                        },
                        "height": {
                          "type": "number",
                          "description": "The height dimension for the placement (in inches).",
                          "example": 33.11
                        },
                        "width": {
                          "type": "number",
                          "description": "The width dimension for the placement (in inches).",
                          "example": 23.39
                        },
                        "orientation": {
                          "type": "string",
                          "description": "The orientation of the placement, such as \"vertical\", \"horizontal\" or \"any\".",
                          "example": "any"
                        }
                      }
                    }
                  },
                  "image": {
                    "description": "URL of a preview image for this variant",
                    "type": "string",
                    "example": "https://files.cdn.printful.com/products/12/629_1517916489.jpg"
                  },
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "required": [
                      "self",
                      "product_details",
                      "product_variants",
                      "variant_prices",
                      "variant_images",
                      "variant_availability"
                    ],
                    "properties": {
                      "self": {
                        "type": "object",
                        "description": "Link to same resource",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/100"
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
                      "product_details": {
                        "type": "object",
                        "description": "Link to details about the product",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-products/71"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "product_variants": {
                        "type": "object",
                        "description": "Link to a list of all catalog variants that are siblings of this catalog variant, relating to the same catalog product",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "variant_prices": {
                        "type": "object",
                        "description": "Link to pricing information about this catalog variant",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/100/prices"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "variant_images": {
                        "type": "object",
                        "description": "Link to images related to this variant and details about those images",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/100/images"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "variant_availability": {
                        "type": "object",
                        "description": "Link to information about the availability of this variant",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/100/availability"
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
                  "product_variants",
                  "product_details",
                  "variant_prices",
                  "variant_images"
                ],
                "properties": {
                  "self": {
                    "type": "object",
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-variants/100"
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
                  "product_variants": {
                    "description": "Link all variants associated with this variants product",
                    "type": "object",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product_details": {
                    "description": "Link product data associated with this variant",
                    "type": "object",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "variant_prices": {
                    "description": "Link to variant prices",
                    "type": "object",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-variants/4011/prices"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "variant_images": {
                    "description": "Link to variant images",
                    "type": "object",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-variant/4011/images"
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

### GET /v2/catalog-products/{id}/catalog-variants
**Summary:** Retrieve information about catalog product variants

Returns information about all catalog variants associated with the specified
catalog product. [See catalog variant](#tag/Catalog-v2/What-is-a-catalog-variant)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductVariantsById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                "title": "Variant",
                "items": {
                  "type": "object",
                  "title": "Variant",
                  "required": [
                    "id",
                    "catalog_product_id",
                    "name",
                    "size",
                    "color",
                    "color_code",
                    "color_code2",
                    "image",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "description": "Variant ID, use this to specify the product when creating orders",
                      "type": "integer",
                      "example": 100
                    },
                    "catalog_product_id": {
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
                      "nullable": true,
                      "example": "Black"
                    },
                    "color_code": {
                      "description": "Hexadecimal RGB color code. May not exactly reflect the real-world color",
                      "type": "string",
                      "nullable": true,
                      "example": "#14191e"
                    },
                    "color_code2": {
                      "description": "Secondary hexadecimal RGB color code. May not exactly reflect the real-world color",
                      "type": "string",
                      "nullable": true
                    },
                    "placement_dimensions": {
                      "description": "A list of placement configuration objects, each specifying the layout details for a particular placement.",
                      "items": {
                        "type": "object",
                        "properties": {
                          "placement": {
                            "type": "string",
                            "description": "The name or type of the placement (e.g., \"default\").",
                            "example": "front"
                          },
                          "height": {
                            "type": "number",
                            "description": "The height dimension for the placement (in inches).",
                            "example": 33.11
                          },
                          "width": {
                            "type": "number",
                            "description": "The width dimension for the placement (in inches).",
                            "example": 23.39
                          },
                          "orientation": {
                            "type": "string",
                            "description": "The orientation of the placement, such as \"vertical\", \"horizontal\" or \"any\".",
                            "example": "any"
                          }
                        }
                      }
                    },
                    "image": {
                      "description": "URL of a preview image for this variant",
                      "type": "string",
                      "example": "https://files.cdn.printful.com/products/12/629_1517916489.jpg"
                    },
                    "_links": {
                      "type": "object",
                      "description": "HATEOAS links",
                      "required": [
                        "self",
                        "product_details",
                        "product_variants",
                        "variant_prices",
                        "variant_images",
                        "variant_availability"
                      ],
                      "properties": {
                        "self": {
                          "type": "object",
                          "description": "Link to same resource",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-variants/100"
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
                        "product_details": {
                          "type": "object",
                          "description": "Link to details about the product",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-products/71"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        },
                        "product_variants": {
                          "type": "object",
                          "description": "Link to a list of all catalog variants that are siblings of this catalog variant, relating to the same catalog product",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        },
                        "variant_prices": {
                          "type": "object",
                          "description": "Link to pricing information about this catalog variant",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-variants/100/prices"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        },
                        "variant_images": {
                          "type": "object",
                          "description": "Link to images related to this variant and details about those images",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-variants/100/images"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        },
                        "variant_availability": {
                          "type": "object",
                          "description": "Link to information about the availability of this variant",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-variants/100/availability"
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
                  "self",
                  "product_details"
                ],
                "properties": {
                  "self": {
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants?limit=20&offset=40"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants?limit=20&offset=60"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants?limit=20&offset=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants?limit=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-variants?limit=20&offset=100"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product_details": {
                    "description": "Link to the product details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
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

### GET /v2/catalog-categories
**Summary:** Retrieve a list of catalog categories

Returns list of all categories that are present in the catalog. The categories specify the type of the product that is associated with it. For example, the category "Men’s T-shirts" indicates that the product is a subgroup of T-shirts specifically targeted at Men.
Categories can be used to filter the product list by specific tags [See categories_ids](#operation/getProducts)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCategories",
  "security": [
    {
      "OAuth": []
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
                  "type": "object",
                  "title": "Category",
                  "description": "Information about the Category",
                  "required": [
                    "id",
                    "parent_id",
                    "image_url",
                    "title",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "description": "Category ID",
                      "type": "integer",
                      "example": 24
                    },
                    "parent_id": {
                      "description": "ID of the parent Category. If there is no parent Category, null is returned.",
                      "type": "integer",
                      "nullable": true,
                      "example": 6
                    },
                    "image_url": {
                      "description": "The URL of the Category image",
                      "type": "string",
                      "example": "https://s3-printful.stage.printful.dev/upload/catalog_category/b1/b1513c82696405fcc316fc611c57f132_t?v=1646395980"
                    },
                    "title": {
                      "description": "Category title",
                      "type": "string",
                      "example": "T-Shirts"
                    },
                    "_links": {
                      "description": "HATEOAS links",
                      "type": "object",
                      "properties": {
                        "self": {
                          "type": "object",
                          "description": "Link to single category",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-categories/1"
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
                        }
                      }
                    }
                  }
                }
              },
              "_links": {
                "description": "HATEOAS links",
                "type": "object",
                "required": [
                  "self",
                  "first",
                  "last"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-categories?limit=20&offset=0"
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
                      "href": "https://api.printful.com/v2/catalog-categories?limit=20&offset=20"
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
                      "href": "https://api.printful.com/v2/catalog-categories?limit=20&offset=40"
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
                      "href": "https://api.printful.com/v2/catalog-categories?limit=20"
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
                      "href": "https://api.printful.com/v2/catalog-categories?limit=20&offset=90"
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

### GET /v2/catalog-categories/{id}
**Summary:** Retrieve information about specific category

Returns information about a specific catalog category. The categories specify the type of the product that is associated with it. For example, the category "Men’s T-shirts" indicates that the product is a subgroup of T-shirts specifically targeted at Men.
Categories can be used to filter the product list by specific tags [See categories_ids](#operation/getProducts)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCategoryById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "Category",
                "description": "Information about the Category",
                "required": [
                  "id",
                  "parent_id",
                  "image_url",
                  "title",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "description": "Category ID",
                    "type": "integer",
                    "example": 24
                  },
                  "parent_id": {
                    "description": "ID of the parent Category. If there is no parent Category, null is returned.",
                    "type": "integer",
                    "nullable": true,
                    "example": 6
                  },
                  "image_url": {
                    "description": "The URL of the Category image",
                    "type": "string",
                    "example": "https://s3-printful.stage.printful.dev/upload/catalog_category/b1/b1513c82696405fcc316fc611c57f132_t?v=1646395980"
                  },
                  "title": {
                    "description": "Category title",
                    "type": "string",
                    "example": "T-Shirts"
                  },
                  "_links": {
                    "description": "HATEOAS links",
                    "type": "object",
                    "properties": {
                      "self": {
                        "type": "object",
                        "description": "Link to single category",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-categories/1"
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
                      }
                    }
                  }
                }
              },
              "_links": {
                "description": "HATEOAS links",
                "type": "object",
                "required": [
                  "all_categories"
                ],
                "properties": {
                  "all_categories": {
                    "type": "object",
                    "description": "Link to all catalog categories",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-categories"
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

### GET /v2/catalog-products/{id}/catalog-categories
**Summary:** Retrieve a list of catalog product categories

To retrieve information about a particular products categories, use this feature. It returns details about the catalog categories associated with the catalog product. Categories help identify the type of product associated with them. For instance, the category "Men's T-shirts" denotes that the product is a subgroup of T-shirts intended for men.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCategoriesByProductId",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
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
                  "title": "Category",
                  "description": "Information about the Category",
                  "required": [
                    "id",
                    "parent_id",
                    "image_url",
                    "title",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "description": "Category ID",
                      "type": "integer",
                      "example": 24
                    },
                    "parent_id": {
                      "description": "ID of the parent Category. If there is no parent Category, null is returned.",
                      "type": "integer",
                      "nullable": true,
                      "example": 6
                    },
                    "image_url": {
                      "description": "The URL of the Category image",
                      "type": "string",
                      "example": "https://s3-printful.stage.printful.dev/upload/catalog_category/b1/b1513c82696405fcc316fc611c57f132_t?v=1646395980"
                    },
                    "title": {
                      "description": "Category title",
                      "type": "string",
                      "example": "T-Shirts"
                    },
                    "_links": {
                      "description": "HATEOAS links",
                      "type": "object",
                      "properties": {
                        "self": {
                          "type": "object",
                          "description": "Link to single category",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-categories/1"
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
                "description": "HATEOAS links",
                "type": "object",
                "required": [
                  "self",
                  "all_categories"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-categories?limit=20&offset=40"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-categories?limit=20&offset=60"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-categories?limit=20&offset=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-categories?limit=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/catalog-categories?limit=20&offset=100"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "all_categories": {
                    "type": "object",
                    "description": "Link to all catalog categories",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-categories"
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

### GET /v2/catalog-products/{id}/sizes
**Summary:** Retrieve size guide for a catalog product

Returns information about the size guide for a specific product.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductSizeGuideById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                "type": "object",
                "title": "ProductSizeGuide",
                "description": "Size Guide information for the Product",
                "required": [
                  "catalog_product_id",
                  "available_sizes",
                  "size_tables"
                ],
                "properties": {
                  "catalog_product_id": {
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
                        "unit",
                        "description",
                        "image_url",
                        "image_description",
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
                              "type_label",
                              "unit",
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
                        "image_url": "https://s3-printful.stage.printful.dev/upload/measure-yourself/6a/6a4fe322f592f2b91d5a735d7ff8d1c0_t?v=1652962720",
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
                        "image_url": "https://s3-printful.stage.printful.dev/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
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
                        "image_url": "https://s3-printful.stage.printful.dev/upload/measure-yourself/6a/6a4fe322f592f2b91d5a735d7ff8d1c0_t?v=1652962720",
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
                        "image_url": "https://s3-printful.stage.printful.dev/upload/product-measure/85/857e7cc8b802da216e7f1a6114075a72_t?v=1652962720",
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
              },
              "_links": {
                "description": "HATEOAS links",
                "type": "object",
                "required": [
                  "self",
                  "product_details"
                ],
                "properties": {
                  "self": {
                    "type": "object",
                    "description": "Link to same resource",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/sizes"
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
                  "product_details": {
                    "type": "object",
                    "description": "Link to the catalog product",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
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

### GET /v2/catalog-products/{id}/prices
**Summary:** Retrieve catalog product prices


Calculates prices for specific catalog product based on selling region and specified currency. Calculations also include Store discounts. Selling region is used to specify product production currency, that is the price that the product is natively manufactured in. Different selling regions might affect the overall price amount. Currency parameter is used only to define the currency that the prices will be displayed in.

For more information on product pricing please refer to the information provided at https://www.printful.com/pricing
<div class="alert alert-info" style="word-wrap: break-word; padding: 16px; border-radius: 0; cursor: default; color: #31708f; background-color: #d9edf7; border-color: #bce8f1;">
    <p>
        When developing against either API be sure to inform your customers that a placement will be included in the price of the product. If one placement is provided that placement will be included in the price, if multiple are provided the included placement will generally be the placement that comes earliest in the list of placements at `/v2/catalog-products/71` (though the discount will generally be up to the price of the first placement in that list). Certain placements come with additional service fees, such as large embroidery, this additional price will never be included even if the only placement is large embroidery.
    </p>
    <p>
        There is a minor difference in the handling of prices for placements between V1 and V2.
        In V1 the price of the first placement is always null, this is because there
        is always a placement included in the price of each product. In V2 the price of placements
        is always displayed even if it is included in the price of the product because any
        placement can be included.
    </p>
</div>

The product pricing process begins with determining the base product price, which depends on the technique used, such as DTG, Embroidery, or DTF. The first placement is typically included in the base price unless it involves special placements like large embroidery or inside labels, which are priced separately. If additional placements are added, their prices are included in the summary cost. Furthermore, additional options, such as 3D puff embroidery or unlimited thread colors, also increase the total cost. The final price is calculated by summing up the base price, additional placement costs, and any extra option costs.

<pre class="mermaid">
    flowchart TD
    B{Technique Used} -->|DTG| C[Base Price: DTG]
    B -->|Embroidery| D[Base Price: Embroidery]
    B -->|DTF| E[Base Price: DTF]
    C --> A
    D --> A
    E --> A
    A[Base Product Price] --> F[First Placement Included?]


    F -->|Yes| G[Base Price Includes First Placement]
    F -->|No e.g., Large Embroidery, Inside Label| H[Special Placement Priced Separately]

    G --> I[Additional Placements?]
    I -->|Yes| J[Add Placement Price to Summary Cost]
    I -->|No| K[Summary Cost = Base Price]

    H --> J
    J --> L[Additional Options?]
    L -->|Yes, such as 3d puff or unlimited color| M[Add Option Price to Summary Cost]
    L -->|No| K

    M --> K
</pre>


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductPricesById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "example": "worldwide"
      },
      "description": "Specifies the region production currency that the product prices will be calculated in"
    },
    {
      "in": "query",
      "name": "currency",
      "description": "The currency (3-letter code) used to determine currency in which the prices will be displayed. The store currency will be used by default. The format is compliant with ISO 4217 standard.",
      "schema": {
        "type": "string",
        "example": "USD"
      },
      "required": false
    },
    {
      "in": "query",
      "name": "production_currency",
      "description": "The production currency (3-letter code) used to determine currency in which the prices will be calculated. The format is compliant with ISO 4217 standard.\nIf not provided, prices will be calculated for all available production currencies,\nand the highest price will be returned.\n",
      "schema": {
        "type": "string",
        "example": "USD"
      },
      "required": false
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                "type": "object",
                "title": "ProductPrice",
                "description": "Product prices information",
                "required": [
                  "currency",
                  "product",
                  "variants"
                ],
                "properties": {
                  "currency": {
                    "description": "Abbreviation from the store currency or currency specified",
                    "type": "string",
                    "example": "EUR"
                  },
                  "product": {
                    "description": "Product related with the pricing information",
                    "type": "object",
                    "required": [
                      "id",
                      "placements"
                    ],
                    "properties": {
                      "id": {
                        "description": "Product Identifier",
                        "type": "integer",
                        "example": 1
                      },
                      "placements": {
                        "description": "Array containing the pricing information about the placement prices. The product price includes one print, pricing here only applies to each additional placement after the first. Note that while a placement might be included there is sometimes additional service fees that are applied to certain placement, for example large embroidery has an additional fee.",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "AdditionalPlacement",
                          "description": "Info about additional product placements prices.",
                          "required": [
                            "id",
                            "title",
                            "type",
                            "technique_key",
                            "price",
                            "discounted_price",
                            "retail_price",
                            "retail_discounted_price",
                            "placement_options",
                            "layers"
                          ],
                          "properties": {
                            "id": {
                              "description": "ID or key of placement",
                              "type": "string",
                              "example": "default"
                            },
                            "title": {
                              "description": "Title of the placement related",
                              "type": "string",
                              "example": "Print file"
                            },
                            "type": {
                              "description": "Placement type",
                              "type": "string",
                              "example": "Digital printing"
                            },
                            "technique_key": {
                              "description": "Key associated to the available technique",
                              "type": "string",
                              "example": "digital"
                            },
                            "price": {
                              "description": "Price converted to the region currency",
                              "type": "string",
                              "example": "19.75"
                            },
                            "discounted_price": {
                              "description": "Discounted price per region",
                              "type": "string",
                              "example": "18.75"
                            },
                            "placement_options": {
                              "description": "Array containing the pricing information about the file options used",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "FileOptionPrice",
                                "description": "Info about additional product file option prices",
                                "required": [
                                  "name",
                                  "type",
                                  "values",
                                  "description",
                                  "price",
                                  "retail_price"
                                ],
                                "properties": {
                                  "name": {
                                    "description": "Option name",
                                    "type": "string",
                                    "example": "unlimited_color"
                                  },
                                  "type": {
                                    "description": "Option value type",
                                    "type": "string",
                                    "example": "array"
                                  },
                                  "values": {
                                    "description": "Possible option values",
                                    "type": "array",
                                    "example": [
                                      true,
                                      false
                                    ]
                                  },
                                  "description": {
                                    "description": "Option description",
                                    "type": "string",
                                    "example": "Unlimited color"
                                  },
                                  "price": {
                                    "description": "Additional price expressed in the region currency",
                                    "type": "object",
                                    "additionalProperties": {
                                      "type": "string"
                                    },
                                    "example": {
                                      "false": "0.00",
                                      "true": "3.50"
                                    }
                                  }
                                }
                              }
                            },
                            "layers": {
                              "description": "Array containing the pricing information about the layers.",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Layer",
                                "description": "Information about the layer prices",
                                "required": [
                                  "type",
                                  "additional_price",
                                  "retail_additional_price",
                                  "layer_options"
                                ],
                                "properties": {
                                  "type": {
                                    "description": "Type of layer",
                                    "type": "string",
                                    "example": "file"
                                  },
                                  "additional_price": {
                                    "description": "Additional price for layer",
                                    "type": "string",
                                    "example": "0.00"
                                  },
                                  "layer_options": {
                                    "description": "Layer options prices",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "ItemOptionPrice",
                                      "description": "Info about additional product item option prices",
                                      "required": [
                                        "name",
                                        "type",
                                        "values",
                                        "description",
                                        "price",
                                        "retail_price"
                                      ],
                                      "properties": {
                                        "name": {
                                          "description": "Option name",
                                          "type": "string",
                                          "example": "3d_puff"
                                        },
                                        "type": {
                                          "description": "Option value type",
                                          "type": "string",
                                          "example": "array"
                                        },
                                        "values": {
                                          "description": "Possible option values",
                                          "type": "array",
                                          "example": [
                                            true,
                                            false
                                          ]
                                        },
                                        "description": {
                                          "description": "Option description",
                                          "type": "string",
                                          "example": "3D puff"
                                        },
                                        "price": {
                                          "description": "Additional price expressed in the region currency",
                                          "type": "object",
                                          "additionalProperties": {
                                            "type": "string"
                                          },
                                          "example": {
                                            "false": "0.00",
                                            "true": "1.50"
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
                    }
                  },
                  "variants": {
                    "type": "array",
                    "items": {
                      "type": "object",
                      "title": "VariantPriceData",
                      "description": "Variant with the pricing information",
                      "required": [
                        "id",
                        "techniques"
                      ],
                      "properties": {
                        "id": {
                          "description": "Variant id",
                          "type": "integer",
                          "example": 1
                        },
                        "techniques": {
                          "description": "Array containing pricing information about available techniques per variant",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "VariantTechniquePrice",
                            "description": "Product prices information",
                            "required": [
                              "technique_key",
                              "technique_display_name",
                              "price",
                              "discounted_price",
                              "retail_price",
                              "retail_discounted_price"
                            ],
                            "properties": {
                              "technique_key": {
                                "description": "Key associated to the technique",
                                "type": "string",
                                "example": "digital"
                              },
                              "technique_display_name": {
                                "description": "Full technique name",
                                "type": "string",
                                "example": "Digital printing"
                              },
                              "price": {
                                "description": "Price converted to the region currency",
                                "type": "string",
                                "example": "9.50"
                              },
                              "discounted_price": {
                                "description": "Discounted price per region",
                                "type": "string",
                                "example": "8.50"
                              }
                            }
                          }
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
                "description": "HATEOAS links",
                "type": "object",
                "required": [
                  "self",
                  "product_details"
                ],
                "properties": {
                  "self": {
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/prices?limit=10&offset=0"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/prices?limit=10&offset=30"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/prices?limit=10&offset=10"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/prices?limit=10"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/prices?limit=10&offset=90"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product_details": {
                    "description": "Link to product details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
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

### GET /v2/catalog-variants/{id}/prices
**Summary:** Retrieve pricing information for the catalog variant

Return pricing information from a single variant and the parent product

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getVariantPricesById",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Variant ID"
    },
    {
      "in": "query",
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "example": "worldwide"
      },
      "description": "Specifies the region production currency that the product prices will be calculated in"
    },
    {
      "in": "query",
      "name": "currency",
      "description": "The currency (3-letter code) used to determine currency in which the prices will be displayed. The store currency will be used by default. The format is compliant with ISO 4217 standard.",
      "schema": {
        "type": "string",
        "example": "USD"
      },
      "required": false
    },
    {
      "in": "query",
      "name": "production_currency",
      "description": "The production currency (3-letter code) used to determine currency in which the prices will be calculated. The format is compliant with ISO 4217 standard.\nIf not provided, prices will be calculated for all available production currencies,\nand the highest price will be returned.\n",
      "schema": {
        "type": "string",
        "example": "USD"
      },
      "required": false
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                "type": "object",
                "title": "VariantPrice",
                "description": "Variant prices information",
                "required": [
                  "currency",
                  "product",
                  "variant"
                ],
                "properties": {
                  "currency": {
                    "description": "Currency in which prices are returned",
                    "type": "string",
                    "example": "EUR"
                  },
                  "product": {
                    "type": "object",
                    "properties": {
                      "id": {
                        "description": "Product Identifier",
                        "type": "integer",
                        "example": 1
                      },
                      "placements": {
                        "description": "Array containing the pricing information about the placement prices",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "AdditionalPlacement",
                          "description": "Info about additional product placements prices.",
                          "required": [
                            "id",
                            "title",
                            "type",
                            "technique_key",
                            "price",
                            "discounted_price",
                            "retail_price",
                            "retail_discounted_price",
                            "placement_options",
                            "layers"
                          ],
                          "properties": {
                            "id": {
                              "description": "ID or key of placement",
                              "type": "string",
                              "example": "default"
                            },
                            "title": {
                              "description": "Title of the placement related",
                              "type": "string",
                              "example": "Print file"
                            },
                            "type": {
                              "description": "Placement type",
                              "type": "string",
                              "example": "Digital printing"
                            },
                            "technique_key": {
                              "description": "Key associated to the available technique",
                              "type": "string",
                              "example": "digital"
                            },
                            "price": {
                              "description": "Price converted to the region currency",
                              "type": "string",
                              "example": "19.75"
                            },
                            "discounted_price": {
                              "description": "Discounted price per region",
                              "type": "string",
                              "example": "18.75"
                            },
                            "placement_options": {
                              "description": "Array containing the pricing information about the file options used",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "FileOptionPrice",
                                "description": "Info about additional product file option prices",
                                "required": [
                                  "name",
                                  "type",
                                  "values",
                                  "description",
                                  "price",
                                  "retail_price"
                                ],
                                "properties": {
                                  "name": {
                                    "description": "Option name",
                                    "type": "string",
                                    "example": "unlimited_color"
                                  },
                                  "type": {
                                    "description": "Option value type",
                                    "type": "string",
                                    "example": "array"
                                  },
                                  "values": {
                                    "description": "Possible option values",
                                    "type": "array",
                                    "example": [
                                      true,
                                      false
                                    ]
                                  },
                                  "description": {
                                    "description": "Option description",
                                    "type": "string",
                                    "example": "Unlimited color"
                                  },
                                  "price": {
                                    "description": "Additional price expressed in the region currency",
                                    "type": "object",
                                    "additionalProperties": {
                                      "type": "string"
                                    },
                                    "example": {
                                      "false": "0.00",
                                      "true": "3.50"
                                    }
                                  }
                                }
                              }
                            },
                            "layers": {
                              "description": "Array containing the pricing information about the layers.",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "Layer",
                                "description": "Information about the layer prices",
                                "required": [
                                  "type",
                                  "additional_price",
                                  "retail_additional_price",
                                  "layer_options"
                                ],
                                "properties": {
                                  "type": {
                                    "description": "Type of layer",
                                    "type": "string",
                                    "example": "file"
                                  },
                                  "additional_price": {
                                    "description": "Additional price for layer",
                                    "type": "string",
                                    "example": "0.00"
                                  },
                                  "layer_options": {
                                    "description": "Layer options prices",
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "title": "ItemOptionPrice",
                                      "description": "Info about additional product item option prices",
                                      "required": [
                                        "name",
                                        "type",
                                        "values",
                                        "description",
                                        "price",
                                        "retail_price"
                                      ],
                                      "properties": {
                                        "name": {
                                          "description": "Option name",
                                          "type": "string",
                                          "example": "3d_puff"
                                        },
                                        "type": {
                                          "description": "Option value type",
                                          "type": "string",
                                          "example": "array"
                                        },
                                        "values": {
                                          "description": "Possible option values",
                                          "type": "array",
                                          "example": [
                                            true,
                                            false
                                          ]
                                        },
                                        "description": {
                                          "description": "Option description",
                                          "type": "string",
                                          "example": "3D puff"
                                        },
                                        "price": {
                                          "description": "Additional price expressed in the region currency",
                                          "type": "object",
                                          "additionalProperties": {
                                            "type": "string"
                                          },
                                          "example": {
                                            "false": "0.00",
                                            "true": "1.50"
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
                    }
                  },
                  "variant": {
                    "type": "object",
                    "title": "VariantPriceData",
                    "description": "Variant with the pricing information",
                    "required": [
                      "id",
                      "techniques"
                    ],
                    "properties": {
                      "id": {
                        "description": "Variant id",
                        "type": "integer",
                        "example": 1
                      },
                      "techniques": {
                        "description": "Array containing pricing information about available techniques per variant",
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "VariantTechniquePrice",
                          "description": "Product prices information",
                          "required": [
                            "technique_key",
                            "technique_display_name",
                            "price",
                            "discounted_price",
                            "retail_price",
                            "retail_discounted_price"
                          ],
                          "properties": {
                            "technique_key": {
                              "description": "Key associated to the technique",
                              "type": "string",
                              "example": "digital"
                            },
                            "technique_display_name": {
                              "description": "Full technique name",
                              "type": "string",
                              "example": "Digital printing"
                            },
                            "price": {
                              "description": "Price converted to the region currency",
                              "type": "string",
                              "example": "9.50"
                            },
                            "discounted_price": {
                              "description": "Discounted price per region",
                              "type": "string",
                              "example": "8.50"
                            }
                          }
                        }
                      }
                    }
                  }
                }
              },
              "_links": {
                "description": "HATEOAS links",
                "type": "object",
                "properties": {
                  "self": {
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-variants/4011/prices"
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
                  "product_prices": {
                    "description": "Link to product prices",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/prices"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product_details": {
                    "description": "Link to product details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
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

### GET /v2/catalog-products/{id}/images
**Summary:** Retrieve blank images for a catalog product

This feature helps to fetch blank images for a catalog product. These blank images are always white and semi-transparent and can be colored by the user on the client-side as per the specified color in the `data.images.background_color` field. For some mockups the `data.images.background_image` could apply. The endpoint allows filtering of the result based on the type of the mockup, the placement, and the color of the product.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductImagesById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "mockup_style_ids",
      "schema": {
        "type": "integer",
        "example": "1, 2, 3"
      },
      "description": "Used to specify style of images For example:\n  * On the hanger\n  * On the Male/Female model\n  * Flat on the table\n  * etc.\nAvailable mockup styles for catalog product can be found under _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.\n"
    },
    {
      "in": "query",
      "name": "colors",
      "schema": {
        "type": "string",
        "example": "black,red"
      },
      "description": "String values separated by comma. You can specify multiple variant colors filters."
    },
    {
      "in": "query",
      "name": "placement",
      "schema": {
        "type": "string",
        "example": "front"
      },
      "description": "Filters result by specified placement"
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer",
        "minimum": 1,
        "maximum": 20,
        "default": 20
      },
      "required": false,
      "description": "The number of results to return per page."
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
                  "title": "VariantImages",
                  "required": [
                    "catalog_variant_id",
                    "color",
                    "primary_hex_color",
                    "secondary_hex_color",
                    "images"
                  ],
                  "properties": {
                    "catalog_variant_id": {
                      "description": "Variant ID",
                      "type": "integer",
                      "example": 4017
                    },
                    "color": {
                      "description": "Variant color",
                      "type": "string",
                      "nullable": true,
                      "example": "Turquoise"
                    },
                    "primary_hex_color": {
                      "description": "Primary variant hex color used. Use this hex color to fill the mockup.",
                      "type": "string",
                      "nullable": true,
                      "example": "#15d0d2"
                    },
                    "secondary_hex_color": {
                      "description": "Secondary variant hex color used. Use this hex color to fill the mockup.",
                      "nullable": true,
                      "type": "string",
                      "example": null
                    },
                    "images": {
                      "description": "Variant's images",
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "VariantImage",
                        "required": [
                          "placement",
                          "image_url",
                          "background_color",
                          "background_image"
                        ],
                        "properties": {
                          "placement": {
                            "description": "Placement associated with the image",
                            "type": "string",
                            "example": "front"
                          },
                          "image_url": {
                            "description": "image URL",
                            "type": "string",
                            "nullable": true,
                            "example": "https://printful-mockups-dev.s3.amazonaws.com/239-nl4600/medium/flat/front/05_nl4600_flat_front_base_whitebg.png?v=1666248709"
                          },
                          "background_color": {
                            "description": "Background color of an image. Null if background transparent",
                            "type": "string",
                            "example": "#0f0f0f"
                          },
                          "background_image": {
                            "description": "Background image of an image specified in the `image_url`. Null if no background image",
                            "type": "string",
                            "nullable": true,
                            "example": "https://printful-mockups-dev.s3.amazonaws.com/239-nl4600/medium/flat/front/05_nl4600_flat_front_base_whitebg.png?v=1666248709"
                          }
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
                  "self",
                  "product_details"
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
                      }
                    ],
                    "description": "Link to same resource",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog/products/71/images"
                    }
                  },
                  "product_details": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to product details",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-products/71"
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

### GET /v2/catalog-products/{id}/shipping-countries
**Summary:** Retrieve the shipping countries for a Product

Retrieve the list of countries the Catalog Product can be shipped to.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductShippingCountriesById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                  "title": "Country",
                  "required": [
                    "code",
                    "name",
                    "states"
                  ],
                  "properties": {
                    "code": {
                      "type": "string",
                      "description": "Country code",
                      "example": "AU"
                    },
                    "name": {
                      "type": "string",
                      "description": "Country name",
                      "example": "Australia"
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

### GET /v2/catalog-variants/{id}/images
**Summary:** Retrieve blank images for a catalog variant

Returns images for a specified Variant.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getVariantImagesById",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Variant ID"
    },
    {
      "in": "query",
      "name": "mockup_style_ids",
      "schema": {
        "type": "integer",
        "example": "1, 2, 3"
      },
      "description": "Used to specify style of images For example:\n  * On the hanger\n  * On the Male/Female model\n  * Flat on the table\n  * etc.\nAvailable mockup styles for catalog product can be found under _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.\n"
    },
    {
      "in": "query",
      "name": "placement",
      "schema": {
        "type": "string",
        "example": "front"
      },
      "description": "Filters result by specified placement"
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                  "type": "object",
                  "title": "VariantImages",
                  "required": [
                    "catalog_variant_id",
                    "color",
                    "primary_hex_color",
                    "secondary_hex_color",
                    "images"
                  ],
                  "properties": {
                    "catalog_variant_id": {
                      "description": "Variant ID",
                      "type": "integer",
                      "example": 4017
                    },
                    "color": {
                      "description": "Variant color",
                      "type": "string",
                      "nullable": true,
                      "example": "Turquoise"
                    },
                    "primary_hex_color": {
                      "description": "Primary variant hex color used. Use this hex color to fill the mockup.",
                      "type": "string",
                      "nullable": true,
                      "example": "#15d0d2"
                    },
                    "secondary_hex_color": {
                      "description": "Secondary variant hex color used. Use this hex color to fill the mockup.",
                      "nullable": true,
                      "type": "string",
                      "example": null
                    },
                    "images": {
                      "description": "Variant's images",
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "VariantImage",
                        "required": [
                          "placement",
                          "image_url",
                          "background_color",
                          "background_image"
                        ],
                        "properties": {
                          "placement": {
                            "description": "Placement associated with the image",
                            "type": "string",
                            "example": "front"
                          },
                          "image_url": {
                            "description": "image URL",
                            "type": "string",
                            "nullable": true,
                            "example": "https://printful-mockups-dev.s3.amazonaws.com/239-nl4600/medium/flat/front/05_nl4600_flat_front_base_whitebg.png?v=1666248709"
                          },
                          "background_color": {
                            "description": "Background color of an image. Null if background transparent",
                            "type": "string",
                            "example": "#0f0f0f"
                          },
                          "background_image": {
                            "description": "Background image of an image specified in the `image_url`. Null if no background image",
                            "type": "string",
                            "nullable": true,
                            "example": "https://printful-mockups-dev.s3.amazonaws.com/239-nl4600/medium/flat/front/05_nl4600_flat_front_base_whitebg.png?v=1666248709"
                          }
                        }
                      }
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
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
                        "description": "Link to the same resource",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/4017/images"
                        }
                      }
                    ]
                  },
                  "variant_details": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      },
                      {
                        "description": "Link to variant details",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/4011"
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

### GET /v2/catalog-products/{id}/mockup-styles
**Summary:** Retrieve catalog product mockup styles

Returns information about available mockup styles for specified catalog product.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "retrieveMockupStylesByProductId",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "placements",
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "explode": false,
      "examples": {
        "onePlacement": {
          "summary": "Return mockup styles that match a specific placement.",
          "value": [
            "front"
          ]
        },
        "multiplePlacements": {
          "summary": "Return mockup styles that match a list of placements.",
          "value": [
            "front",
            "back"
          ]
        }
      },
      "required": false,
      "description": "One or more placement idenitifiers used to filter in mockup styles that match a given placement. The complete list of placements can be found [here](https://developers.printful.com/docs/#tag/Common/Placements)."
    },
    {
      "in": "query",
      "name": "default_mockup_styles",
      "schema": {
        "type": "boolean",
        "default": false,
        "description": "When set to true only the default mockup styles will be returned.\nFor example mockup styles such as Halloween or Lifestyle won't be returned.\n"
      }
    },
    {
      "in": "query",
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
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
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                  "title": "MockupStyles",
                  "description": "Data containing information about the available mockup styles",
                  "required": [
                    "placement",
                    "display_name",
                    "technique",
                    "print_area_width",
                    "print_area_height",
                    "print_area_type",
                    "dpi",
                    "mockup_styles"
                  ],
                  "properties": {
                    "placement": {
                      "type": "string",
                      "description": "Catalog product placement for which the mockup styles defined in `mockup_style_ids` could be used.",
                      "example": "front"
                    },
                    "display_name": {
                      "type": "string",
                      "description": "Placement display name that can be shown to end-customers.",
                      "example": "Front print"
                    },
                    "technique": {
                      "type": "string",
                      "description": "Technique name",
                      "example": "dtg"
                    },
                    "print_area_width": {
                      "type": "number",
                      "format": "float",
                      "description": "Print area width of a placement defined in inches",
                      "example": 5
                    },
                    "print_area_height": {
                      "type": "number",
                      "format": "float",
                      "description": "Print area height of a placement defined in inches",
                      "example": 5
                    },
                    "print_area_type": {
                      "description": "Type of the print area.",
                      "nullable": true,
                      "type": "string",
                      "enum": [
                        "simple",
                        "advanced"
                      ]
                    },
                    "dpi": {
                      "type": "integer",
                      "description": "Print area DPI",
                      "example": 150
                    },
                    "mockup_styles": {
                      "type": "array",
                      "description": "A list of available mockup styles for example:\n  * On the hanger\n  * On the Male/Female model\n  * Flat on the table\n  * etc.",
                      "items": {
                        "type": "object",
                        "required": [
                          "id",
                          "category_name",
                          "view_name",
                          "restricted_to_variants"
                        ],
                        "properties": {
                          "id": {
                            "type": "integer",
                            "description": "Identifier of mockup style. This value is intended to be used when specifying the mockup styles during mockup generator task creation _[Create mockup generator tasks](#operation/createMockupGeneratorTasks)_.",
                            "example": 1
                          },
                          "category_name": {
                            "type": "string",
                            "description": "The category name of the mockup style",
                            "example": "Couple's"
                          },
                          "view_name": {
                            "type": "string",
                            "description": "Display name of mockup view. View determines a point of view of the camera. E.g to the right or left to the mockup.",
                            "example": "Front"
                          },
                          "restricted_to_variants": {
                            "type": "array",
                            "nullable": true,
                            "description": "A list of variants that this style is restricted to. If `null`, this means that there are no restrictions and the style can be used with all variants",
                            "example": [
                              4,
                              15032,
                              10750
                            ]
                          }
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
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-styles?limit=20&offset=40"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-styles?limit=20&offset=60"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-styles?limit=20&offset=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-styles?limit=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-styles?limit=20&offset=100"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product": {
                    "description": "Link to the Catalog Product",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
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
              "data": {
                "type": "string",
                "description": "Actual error message",
                "example": "Malformed Authorization header"
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
                    "example": "Malformed Authorization header"
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
              "data": {
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

### GET /v2/catalog-products/{id}/mockup-templates
**Summary:** Retrieve catalog product mockup templates

Returns positional data for specified catalog product mockups. The data from this endpoint could be used
to generate your own mockups without the need to use Printful's mockup generator.
![Mockup template](images/mockups/mockup_template.png)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getMockupTemplatesByProductId",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "placements",
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "explode": false,
      "examples": {
        "onePlacement": {
          "summary": "Return products with variants that have front placement.",
          "value": [
            "front"
          ]
        },
        "multiplePlacements": {
          "summary": "Return products with variants that have front or back placement.",
          "value": [
            "front",
            "back"
          ]
        }
      },
      "required": false,
      "description": "One or more identifiers of a placement to return only products with variants that have that placement. The complete list of placements can be found [here](https://developers.printful.com/docs/#tag/Common/Placements)."
    },
    {
      "in": "query",
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
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
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
                  "type": "object",
                  "title": "MockupTemplates",
                  "description": "Data containing information about the available mockup templates which can be used for user-side positioning. For example for intention of generating mockups without the use of Printful's mockup generator.",
                  "required": [
                    "catalog_variant_ids",
                    "placement",
                    "technique",
                    "image_url",
                    "background_url",
                    "background_color",
                    "template_width",
                    "template_height",
                    "print_area_width",
                    "print_area_height",
                    "print_area_top",
                    "print_area_left",
                    "template_positioning",
                    "orientation"
                  ],
                  "properties": {
                    "catalog_variant_ids": {
                      "type": "array",
                      "description": "A list of variant IDs for which the positions apply.",
                      "items": {
                        "type": "integer",
                        "example": 4011
                      }
                    },
                    "placement": {
                      "type": "string",
                      "description": "Catalog product placement that is used for the design.",
                      "example": "front"
                    },
                    "technique": {
                      "type": "string",
                      "description": "Catalog product technique that is used for the design.",
                      "example": "dtg"
                    },
                    "image_url": {
                      "type": "string",
                      "description": "Semi-transparent main template image URL.",
                      "example": "https://www.printful.com/files/generator/40/11oz_template.png"
                    },
                    "background_url": {
                      "type": "string",
                      "description": "Background image URL (optional). Used for certain mockups e.g. a wall behind hanged poster. If it's defined it is intended to be layered under the image defined in `image_url`.",
                      "nullable": true,
                      "example": null
                    },
                    "background_color": {
                      "type": "integer",
                      "description": "HEX color code that should be used as a background color of `image_url`.",
                      "nullable": true,
                      "example": null
                    },
                    "template_width": {
                      "type": "integer",
                      "description": "Width of the whole template in pixels.",
                      "example": 560
                    },
                    "template_height": {
                      "type": "integer",
                      "description": "Height of the whole template in pixels.",
                      "example": 295
                    },
                    "print_area_width": {
                      "type": "integer",
                      "description": "Print area width (image is positioned in this area).",
                      "example": 520
                    },
                    "print_area_height": {
                      "type": "integer",
                      "description": "Print area height (image is positioned in this area).",
                      "example": 202
                    },
                    "print_area_top": {
                      "type": "integer",
                      "description": "Print area top offset (offset in template).",
                      "example": 18
                    },
                    "print_area_left": {
                      "type": "integer",
                      "description": "Print area left offset (offset in template).",
                      "example": 20
                    },
                    "template_positioning": {
                      "type": "string",
                      "enum": [
                        "overlay",
                        "background"
                      ],
                      "description": "Should the main template image (image_url) be used as an overlay or as a background.",
                      "example": "overlay"
                    },
                    "orientation": {
                      "type": "string",
                      "description": "Wall art product orientation. Possible values: horizontal, vertical, any",
                      "enum": [
                        "horizontal",
                        "vertical",
                        "any"
                      ],
                      "example": "any"
                    },
                    "template_type": {
                      "description": "Type of inside label used, \"native\" refers to labels that have preset information, \"custom\" are fully customizable and require the user to supply country of manufacturing origin, original garment size, and material information. \"advanced\" is for products like for products like AOP Tote. \"color_group\" for the new inside labels where there are multiple designs for the overlay.",
                      "type": "string",
                      "nullable": true,
                      "enum": [
                        "custom",
                        "native",
                        "color_group",
                        "advanced"
                      ]
                    },
                    "role": {
                      "type": "string",
                      "enum": [
                        "primary",
                        "template",
                        "extra",
                        "unknown",
                        "advanced_template"
                      ],
                      "description": "Mockup template role.",
                      "example": "template"
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
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-templates?limit=20&offset=40"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-templates?limit=20&offset=60"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-templates?limit=20&offset=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-templates?limit=20"
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
                      "href": "https://api.printful.com/v2/catalog-products/71/mockup-templates?limit=20&offset=100"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product": {
                    "description": "Link to the Catalog Product",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-products/71"
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

### GET /v2/catalog-products/{id}/availability
**Summary:** Retrieve catalog product stock availability

Provides information about the catalog product stock status. Stock availability is grouped by variants &rarr; techniques &rarr; selling regions.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductStockAvailabilityById",
  "security": [
    {
      "OAuth": []
    }
  ],
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
      "name": "techniques",
      "schema": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "dtg",
            "digital",
            "cut-sew",
            "uv",
            "embroidery",
            "sublimation",
            "dtfilm"
          ]
        }
      },
      "explode": false,
      "examples": {
        "oneTechnique": {
          "summary": "Products with variants that can be printed using the technique digital.",
          "value": [
            "digital"
          ]
        },
        "multipleTechniques": {
          "summary": "Products with variants that can be printed using the techniques digital or embroidery.",
          "value": [
            "digital",
            "embroidery"
          ]
        }
      },
      "required": false,
      "description": "One or more techniques to return only products with variants that can be printed using one of the techniques."
    },
    {
      "in": "query",
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
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
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
              "filter_settings",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "VariantStockAvailability",
                  "description": "Stock availability data for a specific catalog variant",
                  "required": [
                    "catalog_variant_id",
                    "techniques",
                    "_links"
                  ],
                  "properties": {
                    "catalog_variant_id": {
                      "description": "Catalog variant ID for which the the stock availability data apply",
                      "type": "integer",
                      "example": 4011
                    },
                    "techniques": {
                      "description": "Stock availability data for specific techniques of a catalog variant",
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "TechniqueStockAvailability",
                        "required": [
                          "technique",
                          "selling_regions"
                        ],
                        "properties": {
                          "technique": {
                            "allOf": [
                              {
                                "type": "string",
                                "enum": [
                                  "dtg",
                                  "digital",
                                  "cut-sew",
                                  "uv",
                                  "embroidery",
                                  "sublimation",
                                  "dtfilm"
                                ]
                              },
                              {
                                "description": "Name of the technique for which the stock availability apply"
                              }
                            ]
                          },
                          "selling_regions": {
                            "description": "List of selling regions with stock availability",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "SellingRegionStockAvailability",
                              "required": [
                                "name",
                                "availability",
                                "placement_option_availability"
                              ],
                              "properties": {
                                "name": {
                                  "description": "Name of the selling region for which the stock availability apply",
                                  "type": "string",
                                  "enum": [
                                    "worldwide",
                                    "north_america",
                                    "canada",
                                    "europe",
                                    "spain",
                                    "latvia",
                                    "uk",
                                    "france",
                                    "germany",
                                    "australia",
                                    "japan",
                                    "new_zealand",
                                    "italy",
                                    "brazil",
                                    "southeast_asia",
                                    "republic_of_korea"
                                  ]
                                },
                                "availability": {
                                  "type": "string",
                                  "description": "Availability status:\n  * in stock: The product is stocked in this region and fulfillable with the specified technique\n  * out of stock: Product went out of stock at the supplier in this region but is fulfillable with the specified technique\n  * not fulfillable: (a) Printful does not stock this product in this region; or (b) the product is not fulfillable with the specified technique in this region\n  * unknown: The exact stock status could not be determined\n",
                                  "enum": [
                                    "in stock",
                                    "out of stock",
                                    "not fulfillable",
                                    "unknown"
                                  ]
                                },
                                "placement_option_availability": {
                                  "description": "Availability of a placement options for a catalog variant in a specified selling region. If a placement option is present in this array and availability is set to true it means it is available for this product. If it is set to false it means that the placement option is available for the variant, but not currently fulfillable for the given selling region settings. If an option is not present in the array but is present as an option on the product (see: [Retrieve a single catalog product](#tag/Catalog-v2/operation/getProducts)) it means the option is always available for that product.\n",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "properties": {
                                      "name": {
                                        "type": "string",
                                        "example": "unlimited_color"
                                      },
                                      "availability": {
                                        "type": "boolean",
                                        "example": true
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
                    "_links": {
                      "type": "object",
                      "description": "HATEOAS links",
                      "properties": {
                        "variant": {
                          "description": "Link to the catalog variant details",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/catalog-variants/4011"
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
              "filter_settings": {
                "type": "array",
                "title": "FilterSettings",
                "readOnly": true,
                "description": "The list of the filters that were used in the request",
                "items": {
                  "type": "object",
                  "required": [
                    "name",
                    "value"
                  ],
                  "properties": {
                    "name": {
                      "type": "string",
                      "description": "Name of the filter",
                      "example": "selling_region_name"
                    },
                    "value": {
                      "type": "string",
                      "description": "Value of the filter",
                      "example": "worldwide"
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self",
                  "product"
                ],
                "properties": {
                  "self": {
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-product/71/availability?limit=20&offset=40"
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
                      "href": "https://api.printful.com/v2/catalog-product/71/availability?limit=20&offset=60"
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
                      "href": "https://api.printful.com/v2/catalog-product/71/availability?limit=20&offset=20"
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
                      "href": "https://api.printful.com/v2/catalog-product/71/availability?limit=20"
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
                      "href": "https://api.printful.com/v2/catalog-product/71/availability?limit=20&offset=100"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "product": {
                    "description": "Link to the catalog product details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-product/71"
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

### GET /v2/catalog-variants/{id}/availability
**Summary:** Retrieve catalog variant stock availability

Provides information about the catalog variant stock status. Stock availability is grouped by variants &rarr; techniques &rarr; selling regions.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getVariantStockAvailabilityById",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Variant ID"
    },
    {
      "in": "query",
      "name": "techniques",
      "schema": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": [
            "dtg",
            "digital",
            "cut-sew",
            "uv",
            "embroidery",
            "sublimation",
            "dtfilm"
          ]
        }
      },
      "explode": false,
      "examples": {
        "oneTechnique": {
          "summary": "Products with variants that can be printed using the technique digital.",
          "value": [
            "digital"
          ]
        },
        "multipleTechniques": {
          "summary": "Products with variants that can be printed using the techniques digital or embroidery.",
          "value": [
            "digital",
            "embroidery"
          ]
        }
      },
      "required": false,
      "description": "One or more techniques to return only products with variants that can be printed using one of the techniques."
    },
    {
      "in": "query",
      "name": "selling_region_name",
      "schema": {
        "type": "string",
        "default": "worldwide",
        "enum": [
          "worldwide",
          "north_america",
          "canada",
          "europe",
          "spain",
          "latvia",
          "uk",
          "france",
          "germany",
          "australia",
          "japan",
          "new_zealand",
          "italy",
          "brazil",
          "southeast_asia",
          "republic_of_korea",
          "all"
        ]
      },
      "description": "Only returns the products that can be sold in the specified region. If is set to 'all' returns each region availability for specified product."
    },
    {
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
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
              "filter_settings",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "VariantStockAvailability",
                "description": "Stock availability data for a specific catalog variant",
                "required": [
                  "catalog_variant_id",
                  "techniques",
                  "_links"
                ],
                "properties": {
                  "catalog_variant_id": {
                    "description": "Catalog variant ID for which the the stock availability data apply",
                    "type": "integer",
                    "example": 4011
                  },
                  "techniques": {
                    "description": "Stock availability data for specific techniques of a catalog variant",
                    "type": "array",
                    "items": {
                      "type": "object",
                      "title": "TechniqueStockAvailability",
                      "required": [
                        "technique",
                        "selling_regions"
                      ],
                      "properties": {
                        "technique": {
                          "allOf": [
                            {
                              "type": "string",
                              "enum": [
                                "dtg",
                                "digital",
                                "cut-sew",
                                "uv",
                                "embroidery",
                                "sublimation",
                                "dtfilm"
                              ]
                            },
                            {
                              "description": "Name of the technique for which the stock availability apply"
                            }
                          ]
                        },
                        "selling_regions": {
                          "description": "List of selling regions with stock availability",
                          "type": "array",
                          "items": {
                            "type": "object",
                            "title": "SellingRegionStockAvailability",
                            "required": [
                              "name",
                              "availability",
                              "placement_option_availability"
                            ],
                            "properties": {
                              "name": {
                                "description": "Name of the selling region for which the stock availability apply",
                                "type": "string",
                                "enum": [
                                  "worldwide",
                                  "north_america",
                                  "canada",
                                  "europe",
                                  "spain",
                                  "latvia",
                                  "uk",
                                  "france",
                                  "germany",
                                  "australia",
                                  "japan",
                                  "new_zealand",
                                  "italy",
                                  "brazil",
                                  "southeast_asia",
                                  "republic_of_korea"
                                ]
                              },
                              "availability": {
                                "type": "string",
                                "description": "Availability status:\n  * in stock: The product is stocked in this region and fulfillable with the specified technique\n  * out of stock: Product went out of stock at the supplier in this region but is fulfillable with the specified technique\n  * not fulfillable: (a) Printful does not stock this product in this region; or (b) the product is not fulfillable with the specified technique in this region\n  * unknown: The exact stock status could not be determined\n",
                                "enum": [
                                  "in stock",
                                  "out of stock",
                                  "not fulfillable",
                                  "unknown"
                                ]
                              },
                              "placement_option_availability": {
                                "description": "Availability of a placement options for a catalog variant in a specified selling region. If a placement option is present in this array and availability is set to true it means it is available for this product. If it is set to false it means that the placement option is available for the variant, but not currently fulfillable for the given selling region settings. If an option is not present in the array but is present as an option on the product (see: [Retrieve a single catalog product](#tag/Catalog-v2/operation/getProducts)) it means the option is always available for that product.\n",
                                "type": "array",
                                "items": {
                                  "type": "object",
                                  "properties": {
                                    "name": {
                                      "type": "string",
                                      "example": "unlimited_color"
                                    },
                                    "availability": {
                                      "type": "boolean",
                                      "example": true
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
                  "_links": {
                    "type": "object",
                    "description": "HATEOAS links",
                    "properties": {
                      "variant": {
                        "description": "Link to the catalog variant details",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/catalog-variants/4011"
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
                      }
                    }
                  }
                }
              },
              "filter_settings": {
                "type": "array",
                "title": "FilterSettings",
                "readOnly": true,
                "description": "The list of the filters that were used in the request",
                "items": {
                  "type": "object",
                  "required": [
                    "name",
                    "value"
                  ],
                  "properties": {
                    "name": {
                      "type": "string",
                      "description": "Name of the filter",
                      "example": "selling_region_name"
                    },
                    "value": {
                      "type": "string",
                      "description": "Value of the filter",
                      "example": "worldwide"
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "properties": {
                  "self": {
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-variants/4011/availability"
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
                  "variant": {
                    "description": "Link to the catalog variant details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/catalog-variants/4011"
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


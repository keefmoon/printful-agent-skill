# Warehouse Products v2

Warehouse Products v2

## Endpoints

### GET /v2/warehouse-products
**Summary:** Retrieve a list of warehouse products

This endpoint returns paginated results containing detailed information about warehouse products, including their variants, stock levels, and dimensions.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getWarehouseProducts",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "filter[name*]",
      "schema": {
        "type": "string"
      },
      "required": false,
      "description": "Wildcard match of the name value. Note that the value will be matched if the name property contains the value anywhere in the string."
    },
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
            "type": "object",
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
                  "required": [
                    "id",
                    "name",
                    "status",
                    "currency",
                    "image_url",
                    "retail_price",
                    "warehouse_variants",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "type": "integer",
                      "description": "Unique identifier of the warehouse product",
                      "example": 356
                    },
                    "name": {
                      "type": "string",
                      "description": "Name of the warehouse product",
                      "example": "Black Heather"
                    },
                    "status": {
                      "type": "string",
                      "enum": [
                        "draft",
                        "awaiting_approval",
                        "approved",
                        "declined",
                        "suspended"
                      ],
                      "description": "Current status of the warehouse product"
                    },
                    "currency": {
                      "type": "string",
                      "description": "Currency code for the product's pricing (e.g., \"USD\")",
                      "example": "USD"
                    },
                    "image_url": {
                      "type": "string",
                      "description": "URL of the product's image",
                      "example": "https://example.com/upload/product-templates/d1/d1341a6efb49f59cc58172ce1c15eb20_l"
                    },
                    "retail_price": {
                      "type": "string",
                      "description": "Retail price of the product (base variant)",
                      "example": 21.54
                    },
                    "warehouse_variants": {
                      "type": "array",
                      "description": "Array of variant details for the product",
                      "items": {
                        "type": "object",
                        "required": [
                          "id",
                          "name",
                          "sku",
                          "image_url",
                          "retail_price",
                          "quantity",
                          "dimensions",
                          "_links"
                        ],
                        "properties": {
                          "id": {
                            "type": "integer",
                            "description": "Unique identifier of the variant",
                            "example": 32453
                          },
                          "name": {
                            "type": "string",
                            "description": "Name of the variant",
                            "example": "Black Heather"
                          },
                          "sku": {
                            "type": "string",
                            "description": "Stock Keeping Unit (SKU) of the variant",
                            "example": "some-sku-12"
                          },
                          "image_url": {
                            "type": "string",
                            "description": "URL of the variant's image",
                            "example": "url.to/your/image/location.png"
                          },
                          "retail_price": {
                            "type": "string",
                            "description": "Retail price of the variant",
                            "example": 32.56
                          },
                          "quantity": {
                            "type": "integer",
                            "description": "Available quantity of the variant",
                            "example": 23
                          },
                          "dimensions": {
                            "type": "object",
                            "description": "Dimensions of the variant",
                            "required": [
                              "width",
                              "height",
                              "weight",
                              "measurement_system",
                              "length"
                            ],
                            "properties": {
                              "measurement_system": {
                                "type": "string",
                                "description": "The system of measurement used for the dimensions.",
                                "enum": [
                                  "imperial",
                                  "metric"
                                ]
                              },
                              "width": {
                                "type": "number",
                                "description": "Width of the variant.",
                                "example": 3.21
                              },
                              "height": {
                                "type": "number",
                                "description": "Height of the variant.",
                                "example": 4.56
                              },
                              "length": {
                                "type": "number",
                                "description": "Length of the variant. (NEW)",
                                "example": 6.53
                              },
                              "weight": {
                                "type": "number",
                                "description": "Weight of the variant.",
                                "example": 3
                              }
                            }
                          },
                          "_links": {
                            "type": "object",
                            "properties": {
                              "self": {
                                "type": "object",
                                "description": "Link to same resource",
                                "title": "HateoasLink",
                                "example": {
                                  "href": "https://api.printful.com/v2/warehouse-variants/5678"
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
                      "type": "object",
                      "description": "Links to related resources",
                      "properties": {
                        "self": {
                          "type": "object",
                          "description": "Link to same resource",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/warehouse-products/1234"
                          },
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            }
                          ]
                        },
                        "warehouse_variants": {
                          "type": "object",
                          "description": "Link to warehouse variants of the warehouse product",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/warehouse-products/1234/warehouse-variants"
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
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/warehouse-products?limit=10&offset=20"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "next": {
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/warehouse-products?limit=10&offset=30"
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
                      "href": "https://api.printful.com/v2/warehouse-products?limit=10&limit=10"
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
                      "href": "https://api.printful.com/v2/warehouse-products?limit=10"
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
                      "href": "https://api.printful.com/v2/warehouse-products?limit=10&offset=90"
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

### GET /v2/warehouse-products/{warehouse_product_id}
**Summary:** Retrieve a single warehouse product

Get information about a single warehouse product with it's stock location.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getWarehouseProductById",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "warehouse_product_id",
      "schema": {
        "type": "integer"
      },
      "required": true,
      "description": "Warehouse Product ID."
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
            "type": "object",
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "object",
                "required": [
                  "id",
                  "name",
                  "status",
                  "currency",
                  "image_url",
                  "retail_price",
                  "warehouse_variants",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "type": "integer",
                    "description": "Unique identifier of the warehouse product",
                    "example": 356
                  },
                  "name": {
                    "type": "string",
                    "description": "Name of the warehouse product",
                    "example": "Black Heather"
                  },
                  "status": {
                    "type": "string",
                    "enum": [
                      "draft",
                      "awaiting_approval",
                      "approved",
                      "declined",
                      "suspended"
                    ],
                    "description": "Current status of the warehouse product"
                  },
                  "currency": {
                    "type": "string",
                    "description": "Currency code for the product's pricing (e.g., \"USD\")",
                    "example": "USD"
                  },
                  "image_url": {
                    "type": "string",
                    "description": "URL of the product's image",
                    "example": "https://example.com/upload/product-templates/d1/d1341a6efb49f59cc58172ce1c15eb20_l"
                  },
                  "retail_price": {
                    "type": "string",
                    "description": "Retail price of the product (base variant)",
                    "example": 21.54
                  },
                  "warehouse_variants": {
                    "type": "array",
                    "description": "Array of variant details for the product",
                    "items": {
                      "type": "object",
                      "required": [
                        "id",
                        "name",
                        "sku",
                        "image_url",
                        "retail_price",
                        "quantity",
                        "stock_location",
                        "dimensions",
                        "_links"
                      ],
                      "properties": {
                        "id": {
                          "type": "integer",
                          "description": "Unique identifier of the warehouse variant",
                          "example": 32453
                        },
                        "name": {
                          "type": "string",
                          "description": "Name of the variant",
                          "example": "Black Heather"
                        },
                        "sku": {
                          "type": "string",
                          "description": "Stock Keeping Unit (SKU) of the warehouse variant",
                          "example": "some-sku-12"
                        },
                        "image_url": {
                          "type": "string",
                          "description": "URL of the variant's image",
                          "example": "url.to/your/image/location.png"
                        },
                        "retail_price": {
                          "type": "string",
                          "description": "Retail price of the variant",
                          "example": 32.56
                        },
                        "quantity": {
                          "type": "integer",
                          "description": "Available quantity of the variant",
                          "example": 23
                        },
                        "stock_location": {
                          "type": "array",
                          "description": "Stock location of the variant",
                          "items": {
                            "type": "object",
                            "required": [
                              "facility",
                              "stocked",
                              "available"
                            ],
                            "properties": {
                              "facility": {
                                "description": "Name of the warehouse facility",
                                "type": "string",
                                "example": "Charlotte Steele Point, United States"
                              },
                              "stocked": {
                                "description": "Total quantity of product variant in our stock",
                                "type": "integer",
                                "example": 10
                              },
                              "available": {
                                "description": "Available quantity of product variant in our stock",
                                "type": "integer",
                                "example": 10
                              }
                            }
                          }
                        },
                        "dimensions": {
                          "type": "object",
                          "description": "Dimensions of the variant",
                          "required": [
                            "width",
                            "height",
                            "weight",
                            "measurement_system",
                            "length"
                          ],
                          "properties": {
                            "measurement_system": {
                              "type": "string",
                              "description": "The system of measurement used for the dimensions.",
                              "enum": [
                                "imperial",
                                "metric"
                              ]
                            },
                            "width": {
                              "type": "number",
                              "description": "Width of the variant.",
                              "example": 3.21
                            },
                            "height": {
                              "type": "number",
                              "description": "Height of the variant.",
                              "example": 4.56
                            },
                            "length": {
                              "type": "number",
                              "description": "Length of the variant. (NEW)",
                              "example": 6.53
                            },
                            "weight": {
                              "type": "number",
                              "description": "Weight of the variant.",
                              "example": 3
                            }
                          }
                        },
                        "_links": {
                          "type": "object",
                          "properties": {
                            "self": {
                              "type": "object",
                              "description": "Link to same resource",
                              "title": "HateoasLink",
                              "example": {
                                "href": "https://api.printful.com/v2/warehouse-variants/5678"
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
                    "type": "object",
                    "description": "Links to related resources",
                    "properties": {
                      "self": {
                        "type": "object",
                        "description": "Link to same resource",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/warehouse-products/1234"
                        },
                        "allOf": [
                          {
                            "$ref": "circular reference to #/components/schemas/HateoasLink"
                          }
                        ]
                      },
                      "warehouse_variants": {
                        "type": "object",
                        "description": "Link to warehouse variants of the warehouse product",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/warehouse-products/1234/warehouse-variants"
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


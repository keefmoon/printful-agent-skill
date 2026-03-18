# Warehouse Products API

Warehouse Products API

## Endpoints

### GET /warehouse/products
**Summary:** Get a list of your warehouse products

Returns a list of warehouse products from your store

<div class="alert alert-info">
  The response for this endpoint was documented as paginated, although it was not paginated. The behavior will be 
  fixed and the paginated result will be set as the default. Currently to get paginated results please send
  <code>true</code>or <code>1</code> in <code>X-PF-Force-Pagination</code> header. 
</div>


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
      "name": "query",
      "schema": {
        "type": "string"
      },
      "required": false,
      "description": "Filter by partial or full product name"
    },
    {
      "in": "header",
      "name": "X-PF-Force-Pagination",
      "schema": {
        "oneOf": [
          {
            "type": "boolean"
          },
          {
            "type": "integer"
          }
        ]
      },
      "required": false,
      "description": "Whether the pagination behavior should be forced. The response will be paginated if the value is `true` or `1`."
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
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer"
      },
      "required": false,
      "description": "Result set offset"
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
                "type": "object",
                "properties": {
                  "result": {
                    "type": "array",
                    "description": "Array of WarehouseProducts",
                    "items": {
                      "type": "object",
                      "title": "WarehouseProduct",
                      "description": "Warehouse product data",
                      "properties": {
                        "id": {
                          "type": "integer",
                          "description": "Product ID",
                          "example": 12
                        },
                        "name": {
                          "type": "string",
                          "description": "Product name",
                          "example": "Some product name"
                        },
                        "status": {
                          "type": "string",
                          "description": "Product status:\n\n **created** - product request created,\n\n **active** - product request approved\n\n **suspended** - product suspended\n\n **declined** - product request declined\n\n **draft** - product created as a draft",
                          "enum": [
                            "created",
                            "active",
                            "suspended",
                            "declined",
                            "draft"
                          ],
                          "example": "draft"
                        },
                        "currency": {
                          "type": "string",
                          "description": "Currency",
                          "example": "USD"
                        },
                        "image_url": {
                          "type": "string",
                          "description": "Image URL of product",
                          "example": "url.to/your/image/location.png"
                        },
                        "retail_price": {
                          "type": "number",
                          "description": "Retail price of product",
                          "example": 12.99
                        },
                        "variants": {
                          "type": "array",
                          "description": "Array of product variants",
                          "items": {
                            "type": "object",
                            "title": "WarehouseProductVariant",
                            "description": "Warehouse product variant data",
                            "properties": {
                              "id": {
                                "type": "integer",
                                "description": "Product variant ID",
                                "example": 12
                              },
                              "name": {
                                "type": "string",
                                "description": "Name of product variant",
                                "example": "SomeName"
                              },
                              "sku": {
                                "type": "string",
                                "description": "SKU of product variant",
                                "example": "some-sku-12"
                              },
                              "image_url": {
                                "type": "string",
                                "description": "Image URL of product variant",
                                "example": "url.to/image/variant.png"
                              },
                              "retail_price": {
                                "type": "number",
                                "description": "Retail price of product variant",
                                "format": "float",
                                "example": 1.1
                              },
                              "quantity": {
                                "type": "integer",
                                "description": "Total available quantity of product variant in our stock",
                                "example": 10
                              },
                              "length": {
                                "type": "number",
                                "description": "Length of product variant",
                                "format": "float",
                                "example": 3.21
                              },
                              "width": {
                                "type": "number",
                                "description": "Width of product variant",
                                "format": "float",
                                "example": 2.13
                              },
                              "height": {
                                "type": "number",
                                "description": "Height of product variant",
                                "format": "float",
                                "example": 3.11
                              },
                              "weight": {
                                "type": "number",
                                "description": "Weight of product variant",
                                "format": "float",
                                "example": 1.11
                              },
                              "stock_locations": {
                                "description": "Product variant quantities in the warehouse stock",
                                "type": "array",
                                "items": {
                                  "type": "object",
                                  "title": "WarehouseStockLocation",
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
      "source": "curl --location --request GET 'https://api.printful.com/warehouse/products?query=some?offset=0&limit=100' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### GET /warehouse/products/{id}
**Summary:** Get warehouse product data

Returns warehouse product data by ID

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getWarehouseProduct",
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
      "description": "Product ID"
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
                "type": "object",
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "WarehouseProduct",
                    "description": "Warehouse product data",
                    "properties": {
                      "id": {
                        "type": "integer",
                        "description": "Product ID",
                        "example": 12
                      },
                      "name": {
                        "type": "string",
                        "description": "Product name",
                        "example": "Some product name"
                      },
                      "status": {
                        "type": "string",
                        "description": "Product status:\n\n **created** - product request created,\n\n **active** - product request approved\n\n **suspended** - product suspended\n\n **declined** - product request declined\n\n **draft** - product created as a draft",
                        "enum": [
                          "created",
                          "active",
                          "suspended",
                          "declined",
                          "draft"
                        ],
                        "example": "draft"
                      },
                      "currency": {
                        "type": "string",
                        "description": "Currency",
                        "example": "USD"
                      },
                      "image_url": {
                        "type": "string",
                        "description": "Image URL of product",
                        "example": "url.to/your/image/location.png"
                      },
                      "retail_price": {
                        "type": "number",
                        "description": "Retail price of product",
                        "example": 12.99
                      },
                      "variants": {
                        "type": "array",
                        "description": "Array of product variants",
                        "items": {
                          "type": "object",
                          "title": "WarehouseProductVariant",
                          "description": "Warehouse product variant data",
                          "properties": {
                            "id": {
                              "type": "integer",
                              "description": "Product variant ID",
                              "example": 12
                            },
                            "name": {
                              "type": "string",
                              "description": "Name of product variant",
                              "example": "SomeName"
                            },
                            "sku": {
                              "type": "string",
                              "description": "SKU of product variant",
                              "example": "some-sku-12"
                            },
                            "image_url": {
                              "type": "string",
                              "description": "Image URL of product variant",
                              "example": "url.to/image/variant.png"
                            },
                            "retail_price": {
                              "type": "number",
                              "description": "Retail price of product variant",
                              "format": "float",
                              "example": 1.1
                            },
                            "quantity": {
                              "type": "integer",
                              "description": "Total available quantity of product variant in our stock",
                              "example": 10
                            },
                            "length": {
                              "type": "number",
                              "description": "Length of product variant",
                              "format": "float",
                              "example": 3.21
                            },
                            "width": {
                              "type": "number",
                              "description": "Width of product variant",
                              "format": "float",
                              "example": 2.13
                            },
                            "height": {
                              "type": "number",
                              "description": "Height of product variant",
                              "format": "float",
                              "example": 3.11
                            },
                            "weight": {
                              "type": "number",
                              "description": "Weight of product variant",
                              "format": "float",
                              "example": 1.11
                            },
                            "stock_locations": {
                              "description": "Product variant quantities in the warehouse stock",
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "WarehouseStockLocation",
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
      "source": "curl --location --request GET 'https://api.printful.com/warehouse/products/12' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---


# Stores v2

Stores v2

## Endpoints

### GET /v2/stores/{store_id}
**Summary:** Retrieve a single store

Get information about a single store.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStoreById",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "store_id",
      "schema": {
        "type": "integer",
        "description": "ID of the resource, assigned by Printful",
        "example": 1234
      },
      "required": true,
      "description": "Store ID"
    },
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use.\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
                "description": "Information about the Store",
                "required": [
                  "id",
                  "type",
                  "name"
                ],
                "properties": {
                  "id": {
                    "description": "Store ID",
                    "type": "integer",
                    "example": 10
                  },
                  "type": {
                    "description": "The type of the store is a reference to the type of integration used, Shopify, Etsy, etc. If no first party integration is used, the type will be `native`.",
                    "type": "string",
                    "example": "native"
                  },
                  "name": {
                    "description": "The name given to the store, chosen by the user.",
                    "type": "string",
                    "example": "My Store"
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

### GET /v2/stores
**Summary:** Retrieves a list of stores

Retrieves a list of all stores available to the token. If the token is a store level token it will return only the one store, if it is an account level token it will return all stores available to the account.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStores",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "header",
      "name": "X-PF-Store-Id",
      "description": "Use this to specify which store you want to use.\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
              "_links",
              "paging"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "description": "Information about the Store",
                  "required": [
                    "id",
                    "type",
                    "name"
                  ],
                  "properties": {
                    "id": {
                      "description": "Store ID",
                      "type": "integer",
                      "example": 10
                    },
                    "type": {
                      "description": "The type of the store is a reference to the type of integration used, Shopify, Etsy, etc. If no first party integration is used, the type will be `native`.",
                      "type": "string",
                      "example": "native"
                    },
                    "name": {
                      "description": "The name given to the store, chosen by the user.",
                      "type": "string",
                      "example": "My Store"
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
                        "example": "\u200bhttps://api.printful.com/v2/stores?limit=20&offset=0"
                      }
                    }
                  },
                  "next": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/stores?limit=20&offset=20"
                      }
                    }
                  },
                  "previous": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/stores?limit=20&offset=0"
                      }
                    }
                  },
                  "first": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/stores?limit=20&offset=0"
                      }
                    }
                  },
                  "last": {
                    "type": "object",
                    "properties": {
                      "href": {
                        "type": "string",
                        "example": "\u200bhttps://api.printful.com/v2/stores?limit=20&offset=20"
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

### GET /v2/stores/{store_id}/statistics
**Summary:** Retrieve statistics for a single store

Returns statistics for specified report types.

You need to specify the report types you want to retrieve in the `report_types` query parameter as a comma-separated list,
e.g. `report_types=sales_and_costs,profit`.

**Note**: You cannot get statistics for a period longer than 6 months.

#### Example

To get statistics in the default currency of a store for `sales_and_costs` and `profit` reports for August 2022, you can use the
following
URL: https://api.printful.com/v2/stores/{id}/statistics?report_types=sales_and_costs,profit&date_from=2022-08-01&date_to=2022-08-31.

### Report types

Currently, the following report types are available:

| Report type                | Description                                              |
|----------------------------|----------------------------------------------------------|
| `sales_and_costs`          | Detailed information on sales and costs grouped by date. |
| `sales_and_costs_summary`  | Short information on sales and costs grouped by date.    |
| `printful_costs`           | Amount paid to Printful for fulfillment and shipping.    |
| `profit`                   | Profit in the specified period.                          |
| `total_paid_orders`        | The number of paid orders in the specified period.       |
| `costs_by_amount`          | Information on costs by amount grouped by date.          |
| `costs_by_product`         | Information on costs grouped by product.                 |
| `costs_by_variant`         | Information on costs grouped by variant.                 |
| `average_fulfillment_time` | Average time it took Printful to fulfill your orders.    |

The response structure for the specific reports is documented in the response schema (`result.store_statistics.[reportName]`).


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getReports",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "store_id",
      "description": "Use this to specify which store you want to use.\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "integer"
      },
      "required": true
    },
    {
      "in": "query",
      "name": "date_from",
      "description": "The beginning of the period to get the statistics from (date in `Y-m-d` format).",
      "schema": {
        "type": "string",
        "format": "date",
        "example": "2022-08-01"
      },
      "required": true
    },
    {
      "in": "query",
      "name": "date_to",
      "description": "The end of the period to get the statistics from (date in `Y-m-d` format).",
      "schema": {
        "type": "string",
        "format": "date",
        "example": "2022-08-31"
      },
      "required": true
    },
    {
      "in": "query",
      "name": "currency",
      "description": "The currency (3-letter code) to return the statistics in.\nThe store currency will be used by default.\n",
      "schema": {
        "type": "string",
        "example": "USD"
      },
      "required": false
    },
    {
      "in": "query",
      "name": "report_types",
      "description": "A comma-separated list of report types to be retrieved.",
      "schema": {
        "type": "string",
        "enum": [
          "sales_and_costs",
          "profit",
          "average_fulfillment_type",
          "costs_by_amount",
          "costs_by_product",
          "costs_by_variant",
          "printful_costs",
          "sales_and_costs_summary",
          "total_paid_orders"
        ]
      },
      "required": true
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
              "data"
            ],
            "properties": {
              "data": {
                "type": "object",
                "title": "StoreStatistics",
                "description": "Statistics for a single store",
                "required": [
                  "store_id",
                  "currency"
                ],
                "properties": {
                  "store_id": {
                    "type": "integer",
                    "description": "The ID of the store for which the statistics are returned"
                  },
                  "currency": {
                    "type": "string",
                    "description": "The code of the currency in which the statistics are returned",
                    "example": "USD"
                  },
                  "sales_and_costs": {
                    "type": "array",
                    "title": "SalesAndCostsValue",
                    "description": "Sales and costs report",
                    "items": {
                      "type": "object",
                      "required": [
                        "date",
                        "sales",
                        "fulfillment",
                        "profit",
                        "sales_discount",
                        "fulfillment_discount",
                        "sales_shipping",
                        "fulfillment_shipping"
                      ],
                      "properties": {
                        "date": {
                          "type": "string",
                          "description": "The date of the value: day in `Y-m-d` format, month in `Y-m` format or \"Total\" for the first element of the list which shows the total values for the whole requested period"
                        },
                        "sales": {
                          "type": "string",
                          "description": "Order retail price data. Available only if retail price fields are properly set up on the integration's side"
                        },
                        "fulfillment": {
                          "type": "string",
                          "description": "Product fulfillment, digitization, branding, shipping costs and taxes that are charged by Printful"
                        },
                        "profit": {
                          "type": "string",
                          "description": "The difference between Sales and Fulfillment. If retail price data is not available, profit might be negative"
                        },
                        "sales_discount": {
                          "type": "string",
                          "description": "Any retail price discounts set up on the integration's side"
                        },
                        "fulfillment_discount": {
                          "type": "string",
                          "description": "Any fulfillment discounts (such as the monthly discount) set up on Printful's side"
                        },
                        "sales_shipping": {
                          "type": "string",
                          "description": "The retail shipping price that was paid by the buyer"
                        },
                        "fulfillment_shipping": {
                          "type": "string",
                          "description": "Shipping costs that were charged by Printful"
                        }
                      }
                    }
                  },
                  "sales_and_costs_summary": {
                    "type": "array",
                    "title": "SalesAndCostsSummaryValue",
                    "description": "Sales and costs summary report",
                    "required": [
                      "date",
                      "order_count",
                      "costs",
                      "profit"
                    ],
                    "items": {
                      "type": "object",
                      "properties": {
                        "date": {
                          "type": "string",
                          "description": "The date of the value: day in `Y-m-d` format, month in `Y-m` format or \"Total\" for the first element of the list which shows the total values for the whole requested period"
                        },
                        "order_count": {
                          "type": "number",
                          "description": "The order count in the aggregation period"
                        },
                        "costs": {
                          "type": "string",
                          "description": "Product fulfillment, digitization, branding, shipping costs and taxes that are charged by Printful"
                        },
                        "profit": {
                          "type": "string",
                          "description": "The difference between Sales and Fulfillment. If retail price data is not available, profit might be negative"
                        }
                      }
                    }
                  },
                  "printful_costs": {
                    "type": "object",
                    "title": "PrintfulCosts",
                    "description": "Printful costs report",
                    "required": [
                      "value",
                      "relative_difference"
                    ],
                    "properties": {
                      "value": {
                        "type": "string",
                        "description": "Amount paid to Printful for fulfillment and shipping."
                      },
                      "relative_difference": {
                        "type": "string",
                        "nullable": true,
                        "description": "Relative difference from the value from the previous period. -1 means 100% decrease, 1 means 100% increase. 0 is returned if there is no change or the previous value was 0."
                      }
                    }
                  },
                  "profit": {
                    "type": "object",
                    "title": "Profit",
                    "description": "Profit report",
                    "required": [
                      "value",
                      "relative_difference"
                    ],
                    "properties": {
                      "value": {
                        "type": "string",
                        "description": "The difference between Sales and Fulfillment. If retail price data is not available, profit might be negative"
                      },
                      "relative_difference": {
                        "type": "string",
                        "nullable": true,
                        "description": "Relative difference from the value from the previous period. -1 means 100% decrease, 1 means 100% increase. 0 is returned if there is no change or the previous value was 0."
                      }
                    }
                  },
                  "total_paid_orders": {
                    "type": "object",
                    "title": "TotalPaidOrders",
                    "description": "Total paid orders report",
                    "required": [
                      "value",
                      "relative_difference"
                    ],
                    "properties": {
                      "value": {
                        "type": "integer",
                        "description": "Number of unique orders for period"
                      },
                      "relative_difference": {
                        "type": "string",
                        "nullable": true,
                        "description": "Relative difference from the value from the previous period. -1 means 100% decrease, 1 means 100% increase. 0 is returned if there is no change or the previous value was 0."
                      }
                    }
                  },
                  "costs_by_amount": {
                    "type": "array",
                    "title": "CostsByAmountValue",
                    "description": "Costs by amount report",
                    "items": {
                      "type": "object",
                      "required": [
                        "date",
                        "product_amount",
                        "digitization",
                        "branding",
                        "vat",
                        "sales_tax",
                        "shipping",
                        "discount",
                        "total"
                      ],
                      "properties": {
                        "date": {
                          "type": "string",
                          "description": "The date of the value: day in `Y-m-d` format, month in `Y-m` format or \"Total\" for the first element of the list which shows the total values for the whole requested period"
                        },
                        "product_amount": {
                          "type": "string",
                          "description": "Product & fulfillment costs"
                        },
                        "digitization": {
                          "type": "string",
                          "description": "Embroidery digitization costs"
                        },
                        "branding": {
                          "type": "string",
                          "description": "Pack-in costs"
                        },
                        "vat": {
                          "type": "string",
                          "description": "Tax amounts. If not applicable, it will be 0."
                        },
                        "sales_tax": {
                          "type": "string",
                          "description": "Tax amounts. If not applicable, it will be 0."
                        },
                        "shipping": {
                          "type": "string",
                          "description": "Shipping costs that were charged by Printful"
                        },
                        "discount": {
                          "type": "string",
                          "description": "Any fulfillment discounts (such as the monthly discount) set up on Printful's side"
                        },
                        "total": {
                          "type": "string",
                          "description": "Summary of all costs"
                        }
                      }
                    }
                  },
                  "costs_by_product": {
                    "type": "array",
                    "title": "CostsByProductValue",
                    "description": "Costs by product report",
                    "items": {
                      "type": "object",
                      "required": [
                        "product_id",
                        "product_name",
                        "fulfillment",
                        "sales",
                        "quantity"
                      ],
                      "properties": {
                        "product_id": {
                          "type": "integer",
                          "description": "Product ID. See [Catalog API](#tag/Catalog-API)."
                        },
                        "product_name": {
                          "type": "string",
                          "description": "Product name."
                        },
                        "fulfillment": {
                          "type": "string",
                          "description": "All fulfillment costs that are charged by Printful, excluding shipping."
                        },
                        "sales": {
                          "type": "string",
                          "description": "Order retail price data. Available only if retail price fields are properly set up on the integration's side."
                        },
                        "quantity": {
                          "type": "integer",
                          "description": "Total quantity of items ordered from this product in the selected period."
                        }
                      }
                    }
                  },
                  "costs_by_variant": {
                    "type": "array",
                    "title": "CostsByVariantValue",
                    "description": "Costs by variant report",
                    "items": {
                      "type": "object",
                      "required": [
                        "variant_id",
                        "variant_name",
                        "product_id",
                        "fulfillment",
                        "sales",
                        "quantity"
                      ],
                      "properties": {
                        "variant_id": {
                          "type": "integer",
                          "description": "Variant ID. See [Catalog API](#tag/Catalog-API)."
                        },
                        "variant_name": {
                          "type": "string",
                          "description": "Variant name."
                        },
                        "product_id": {
                          "type": "integer",
                          "description": "Product ID. See [Catalog API](#tag/Catalog-API)."
                        },
                        "fulfillment": {
                          "type": "string",
                          "description": "All fulfillment costs that are charged by Printful, excluding shipping."
                        },
                        "sales": {
                          "type": "string",
                          "description": "Order retail price data. Available only if retail price fields are properly set up on the integration's side."
                        },
                        "quantity": {
                          "type": "integer",
                          "description": "Total quantity of items ordered from this product in the selected period."
                        }
                      }
                    }
                  },
                  "average_fulfillment_time": {
                    "type": "object",
                    "title": "AverageFulfillmentTime",
                    "description": "Average fulfillment time report",
                    "required": [
                      "value",
                      "relative_difference"
                    ],
                    "properties": {
                      "value": {
                        "type": "string",
                        "description": "Average time it took Printful to fulfill your orders."
                      },
                      "relative_difference": {
                        "type": "string",
                        "nullable": true,
                        "description": "Relative difference from the value from the previous period. -1 means 100% decrease, 1 means 100% increase. 0 is returned if there is no change or the previous value was 0."
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


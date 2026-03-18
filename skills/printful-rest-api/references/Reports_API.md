# Reports API

The Reports API lets you retrieve reports like the statistics related to the orders fulfilled for your stores.


## Endpoints

### GET /reports/statistics
**Summary:** Get statistics

Returns statistics for specified report types.

You need to specify the report types you want to retrieve in the `report_types` query parameter as a comma-separated list,
e.g. `report_types=sales_and_costs,profit`.

**Note**: You cannot get statistics for a period longer than 6 months.

#### Example

To get statistics in the default currency of a store for `sales_and_costs` and `profit` reports for August 2022, you can use the
following
URL: https://api.printful.com/reports/statistics?report_types=sales_and_costs,profit&date_from=2022-08-01&date_to=2022-08-31.

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
  "operationId": "getStatistics",
  "security": [
    {
      "OAuth": []
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
      "description": "The currency (3-letter code) to return the statistics in. You can also specify `display_currency` as the value to get the statistics in the account's display currency. The store currency will be used by default.",
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
        "example": "sales_and_costs,profit"
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
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
                "type": "object",
                "title": "Statistics",
                "description": "Statistics",
                "properties": {
                  "store_statistics": {
                    "type": "array",
                    "description": "The statistics for each store (currently only a single store is supported).",
                    "items": {
                      "type": "object",
                      "title": "StoreStatistics",
                      "description": "Statistics for a single store",
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
                            "properties": {
                              "date": {
                                "type": "string",
                                "description": "The date of the value: day in `Y-m-d` format, month in `Y-m` format or \"Total\" for the first element of the list which shows the total values for the whole requested period"
                              },
                              "sales": {
                                "type": "number",
                                "description": "Order retail price data. Available only if retail price fields are properly set up on the integration's side"
                              },
                              "fulfillment": {
                                "type": "number",
                                "description": "Product fulfillment, digitization, branding, shipping costs and taxes that are charged by Printful"
                              },
                              "profit": {
                                "type": "number",
                                "description": "The difference between Sales and Fulfillment. If retail price data is not available, profit might be negative"
                              },
                              "sales_discount": {
                                "type": "number",
                                "description": "Any retail price discounts set up on the integration's side"
                              },
                              "fulfillment_discount": {
                                "type": "number",
                                "description": "Any fulfillment discounts (such as the monthly discount) set up on Printful's side"
                              },
                              "sales_shipping": {
                                "type": "number",
                                "description": "The retail shipping price that was paid by the buyer"
                              },
                              "fulfillment_shipping": {
                                "type": "number",
                                "description": "Shipping costs that were charged by Printful"
                              }
                            }
                          }
                        },
                        "sales_and_costs_summary": {
                          "type": "array",
                          "title": "SalesAndCostsSummaryValue",
                          "description": "Sales and costs summary report",
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
                                "type": "number",
                                "description": "Product fulfillment, digitization, branding, shipping costs and taxes that are charged by Printful"
                              },
                              "profit": {
                                "type": "number",
                                "description": "The difference between Sales and Fulfillment. If retail price data is not available, profit might be negative"
                              }
                            }
                          }
                        },
                        "printful_costs": {
                          "type": "object",
                          "title": "PrintfulCosts",
                          "description": "Printful costs report",
                          "properties": {
                            "value": {
                              "type": "number",
                              "description": "Amount paid to Printful for fulfillment and shipping."
                            },
                            "relative_difference": {
                              "type": "number",
                              "description": "Relative difference from the value from the previous period. -1 means 100% decrease, 1 means 100% increase. 0 is returned if there is no change or the previous value was 0."
                            }
                          }
                        },
                        "profit": {
                          "type": "object",
                          "title": "Profit",
                          "description": "Profit report",
                          "properties": {
                            "value": {
                              "type": "number",
                              "description": "The difference between Sales and Fulfillment. If retail price data is not available, profit might be negative"
                            },
                            "relative_difference": {
                              "type": "number",
                              "description": "Relative difference from the value from the previous period. -1 means 100% decrease, 1 means 100% increase. 0 is returned if there is no change or the previous value was 0."
                            }
                          }
                        },
                        "total_paid_orders": {
                          "type": "object",
                          "title": "TotalPaidOrders",
                          "description": "Total paid orders report",
                          "properties": {
                            "value": {
                              "type": "number",
                              "description": "Number of unique orders for period"
                            },
                            "relative_difference": {
                              "type": "number",
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
                            "properties": {
                              "date": {
                                "type": "string",
                                "description": "The date of the value: day in `Y-m-d` format, month in `Y-m` format or \"Total\" for the first element of the list which shows the total values for the whole requested period"
                              },
                              "product_amount": {
                                "type": "number",
                                "description": "Product & fulfillment costs"
                              },
                              "digitization": {
                                "type": "number",
                                "description": "Embroidery digitization costs"
                              },
                              "branding": {
                                "type": "number",
                                "description": "Pack-in costs"
                              },
                              "vat": {
                                "type": "number",
                                "description": "Tax amounts. If not applicable, it will be 0."
                              },
                              "sales_tax": {
                                "type": "number",
                                "description": "Tax amounts. If not applicable, it will be 0."
                              },
                              "shipping": {
                                "type": "number",
                                "description": "Shipping costs that were charged by Printful"
                              },
                              "discount": {
                                "type": "number",
                                "description": "Any fulfillment discounts (such as the monthly discount) set up on Printful's side"
                              },
                              "total": {
                                "type": "number",
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
                                "type": "number",
                                "description": "All fulfillment costs that are charged by Printful, excluding shipping."
                              },
                              "sales": {
                                "type": "number",
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
                                "type": "number",
                                "description": "All fulfillment costs that are charged by Printful, excluding shipping."
                              },
                              "sales": {
                                "type": "number",
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
                          "properties": {
                            "value": {
                              "type": "number",
                              "description": "Average time it took Printful to fulfill your orders."
                            },
                            "relative_difference": {
                              "type": "number",
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
      "source": "curl --location --request GET 'https://api.printful.com/reports/statistics?report_types=sales_and_costs,profit&date_from=2022-08-01&date_to=2022-08-31&currency=PLN' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---


# Shipping Rate API

The Shipping rate API calculates the shipping rates for an order based on the recipient's location and the contents of
the order.

The returned shipping rate ID can be used to specify the shipping method when creating an order.

See [Country/State Code API](#tag/CountryState-Code-API) for information about the Country codes.

See [Catalog API](#tag/Catalog-API) for information about the Variant IDs.

<div class="alert alert-info">
<p><strong>Note:</strong> The shipping rates endpoints are meant to be called only right before placing an order to display
available shipping rates and methods.</p>

Dynamic shipping rates can change even in the span of one hour because it takes live facility and carrier information
into account. Different rates may be calculated for different products, quantities and recipient data.

Even daily downloads of this data, reused only for identical orders, can result in mismatches between the displayed
rates and the charged rates, potentially resulting in customer dissatisfaction.

A CSV file containing flat rates data for different categories of products may be downloaded
from https://www.printful.com/shipping-rates-report/shipping-rates-report/download and hardcoded to reduce the shipping
rate volume massively if you use flat rate shipping.
</div>

**Rate limiting:** The default rate limit is 120 requests per 60 seconds.

<div class="alert alert-danger">
  <strong>Warning:</strong> If the summary item quantity count exceeds 100 then the rate limit
is changed to 5 requests per 60 seconds.
</div>

A 60 seconds lockout is applied if request count is exceeded.

## Endpoints

### POST /shipping/rates
**Summary:** Calculate shipping rates

Returns available shipping options and rates for the given list of products.

**Recipient Address Requirements:**
- Only `country_code` is required in the recipient object
- `state_code` is only required for United States (US), Australia (AU), and Canada (CA)
- All other recipient fields are optional

**Note:** Providing more address information may produce more precise results and more shipping options. While only the country code is required, including additional details like city, postal code, and state/province can help return more accurate shipping rates and additional delivery options.

**Important:** Shipping rates returned by this endpoint may differ from those returned by `/orders/estimate-costs` if your store has [store shipping settings](https://www.printful.com/dashboard/settings/store-shipping) configured. 

The `/orders/estimate-costs` endpoint automatically applies your store's shipping settings (including shipping markup), while this endpoint applies them **only if the store shipping settings are enabled**. To ensure consistent results between both endpoints, make sure your store shipping settings are enabled and properly configured in your Printful Dashboard.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "calculateShippingRates",
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
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "CalculateShippingRates",
          "description": "Order information",
          "required": [
            "recipient",
            "items"
          ],
          "properties": {
            "recipient": {
              "type": "object",
              "title": "ShippingRatesAddress",
              "description": "Recipient address information for shipping rate calculation.\n\n**Required fields:**\n- `country_code`: Always required\n\n**Conditionally required fields:**\n- `state_code`: Required for United States (US), Australia (AU), and Canada (CA)\n\n**Optional fields:**\n- All other fields are optional but providing more information may produce more precise results and more shipping options.\n",
              "required": [
                "country_code"
              ],
              "properties": {
                "address1": {
                  "description": "Address line 1",
                  "type": "string",
                  "example": "19749 Dearborn St"
                },
                "address2": {
                  "description": "Address line 2",
                  "type": "string",
                  "nullable": true,
                  "example": "Apt 2B"
                },
                "city": {
                  "description": "City",
                  "type": "string",
                  "example": "Chatsworth"
                },
                "state_code": {
                  "description": "State/province code. Required for United States (US), Australia (AU), and Canada (CA).\nFor other countries, this field is optional.\n",
                  "type": "string",
                  "example": "CA",
                  "nullable": true
                },
                "country_code": {
                  "description": "Two-letter country code (ISO 3166-1 alpha-2)",
                  "type": "string",
                  "example": "US"
                },
                "zip": {
                  "description": "ZIP or postal code",
                  "type": "string",
                  "example": "91311",
                  "nullable": true
                },
                "phone": {
                  "description": "Phone number",
                  "type": "string",
                  "example": "+1234567890",
                  "nullable": true
                }
              }
            },
            "items": {
              "type": "array",
              "description": "Array of order items",
              "items": {
                "type": "object",
                "title": "ItemInfo",
                "description": "Order item information",
                "required": [
                  "quantity"
                ],
                "properties": {
                  "variant_id": {
                    "type": "string",
                    "description": "Catalog Variant ID of the item ordered. See [Catalog API](#tag/Catalog-API) <span style=\"color:red\">*Required if no other IDs given</span>",
                    "example": "202"
                  },
                  "external_variant_id": {
                    "type": "string",
                    "description": "External Variant ID of the item ordered. See [Ecommerce Platform Sync API](#tag/Ecommerce-Platform-Sync-API). <span style=\"color:red\">*Required if no other IDs given</span>",
                    "example": "1001"
                  },
                  "warehouse_product_variant_id": {
                    "type": "string",
                    "description": "Warehouse product variant ID of the item ordered. See [Warehouse Products API](#tag/Warehouse-Products-API). <span style=\"color:red\">*Required if no other IDs given</span>",
                    "example": "2"
                  },
                  "quantity": {
                    "type": "integer",
                    "description": "Number of items ordered",
                    "example": 10
                  },
                  "value": {
                    "type": "string",
                    "description": "Item retail value - optional but can help to properly calculate",
                    "example": "2.99"
                  }
                }
              }
            },
            "currency": {
              "type": "string",
              "description": "3 letter currency code (optional), required if the rates need to be converted to another currency instead of store default currency",
              "example": "USD"
            },
            "locale": {
              "type": "string",
              "description": "Locale in which shipping rate names will be returned. Available options: `en_US` (default), `es_ES`",
              "example": "en_US"
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
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "ShippingInfo",
                  "description": "Shipping information",
                  "properties": {
                    "id": {
                      "type": "string",
                      "description": "Shipping method ID",
                      "example": "STANDARD"
                    },
                    "name": {
                      "type": "string",
                      "description": "Shipping method name",
                      "example": "Flat Rate (Estimated delivery: May 19\u201324) "
                    },
                    "rate": {
                      "type": "string",
                      "description": "Shipping rate",
                      "example": "13.60"
                    },
                    "currency": {
                      "type": "string",
                      "description": "Currency code in which the rate is returned",
                      "example": "EUR"
                    },
                    "minDeliveryDays": {
                      "type": "integer",
                      "description": "Estimated minimum delivery days. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": 4
                    },
                    "maxDeliveryDays": {
                      "type": "integer",
                      "description": "Estimated maximum delivery days. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": 7
                    },
                    "minDeliveryDate": {
                      "type": "integer",
                      "description": "Estimated minimum delivery date. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": "2022-10-17"
                    },
                    "maxDeliveryDate": {
                      "type": "integer",
                      "description": "Estimated maximum delivery date. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": "2022-10-20"
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
      "source": "curl --location --request POST 'https://api.printful.com/shipping/rates' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n                \"recipient\": {\n                    \"address1\": \"19749 Dearborn St\",\n                    \"city\": \"Chatsworth\",\n                    \"country_code\": \"US\",\n                    \"state_code\": \"CA\",\n                    \"zip\": 91311\n                },\n                \"items\": [\n                    {\n                        \"quantity\": 1,\n                        \"variant_id\": 2\n                    },\n                    {\n                        \"quantity\": 5,\n                        \"variant_id\": 202\n                    }\n                ]\n            }'"
    }
  ]
}
```
</details>

---


# Tax Rate API

 

## Endpoints

### GET /tax/countries
**Summary:** Get a list of countries for tax calculation

Retrieve state list that requires sales tax calculation

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getTaxCountries",
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
                  "allOf": [
                    {
                      "type": "object",
                      "title": "Country",
                      "properties": {
                        "code": {
                          "type": "string",
                          "description": "Country code",
                          "example": "Australia"
                        },
                        "name": {
                          "type": "string",
                          "description": "Country name",
                          "example": "AU"
                        },
                        "states": {
                          "type": "array",
                          "nullable": true,
                          "items": {
                            "type": "object",
                            "title": "State",
                            "properties": {
                              "code": {
                                "type": "string",
                                "description": "State code",
                                "example": "ACT"
                              },
                              "name": {
                                "type": "string",
                                "description": "State name",
                                "example": "Australian Capital Territory"
                              }
                            }
                          }
                        },
                        "region": {
                          "type": "string",
                          "example": "oceania"
                        }
                      }
                    },
                    {
                      "type": "object",
                      "properties": {
                        "states": {
                          "type": "array",
                          "items": {
                            "type": "object",
                            "properties": {
                              "shipping_taxable": {
                                "description": "Whether shipping is taxable in this state",
                                "type": "boolean",
                                "example": true
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
      "source": "curl --location --request GET 'https://api.printful.com/tax/countries'"
    }
  ]
}
```
</details>

---

### POST /tax/rates
**Summary:** Calculate tax rate

Calculates sales tax rate for given address if required.

<div class="alert alert-info">
  ⚠️ <strong>Important – <tt>/tax/rates</tt> endpoint sunset</strong>
  <p>Since May 2023, the POST <tt>/tax/rates</tt> endpoint is no longer maintained and may return inaccurate results.</p>
  <p>On <strong>July 29, 2025</strong>, we started the sunset process. The rate limit is being reduced by 10 RPM each week (starting with 60) until it reaches 0 on <strong>September 8, 2025</strong>, at which point the endpoint will be removed entirely.</p>
  <p>There is no replacement endpoint in the Printful API for retrieving standalone tax rates. For accurate tax information, please use official government sources or external tax calculation providers.</p>
  <p>If you require the total order cost including taxes, use the order creation or estimation endpoints.</p>
</div>


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "calculateTaxRates",
  "deprecated": true,
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "TaxRequest",
          "description": "Tax address information",
          "required": [
            "recipient"
          ],
          "properties": {
            "recipient": {
              "type": "object",
              "title": "TaxAddressInfo",
              "description": "Recipient address information",
              "required": [
                "country_code",
                "state_code",
                "city",
                "zip"
              ],
              "properties": {
                "country_code": {
                  "type": "string",
                  "description": "Country code",
                  "example": "US"
                },
                "state_code": {
                  "type": "string",
                  "description": "State code",
                  "example": "CA"
                },
                "city": {
                  "type": "string",
                  "description": "City",
                  "example": "Chatsworth"
                },
                "zip": {
                  "type": "string",
                  "description": "ZIP or postal code",
                  "example": "91311"
                }
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
            "type": "object",
            "properties": {
              "code": {
                "type": "integer",
                "description": "Response status code `200`",
                "example": 200
              },
              "result": {
                "type": "object",
                "title": "TaxInfo",
                "description": "Tax address information",
                "properties": {
                  "required": {
                    "type": "boolean",
                    "example": true,
                    "description": "Whether sales tax is required for the given address"
                  },
                  "rate": {
                    "type": "number",
                    "example": 0.095,
                    "description": "Tax rate"
                  },
                  "shipping_taxable": {
                    "description": "Whether shipping is taxable",
                    "type": "boolean",
                    "example": false
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
      "source": "curl --location --request POST 'https://api.printful.com/tax/rates' \\\n--header 'Content-Type: application/json' \\\n--data-raw '{\n                \"recipient\": {\n                    \"country_code\": \"US\",\n                    \"state_code\": \"CA\",\n                    \"city\": \"Chatsworth\",\n                    \"zip\": \"91311\"\n                }\n            }'"
    }
  ]
}
```
</details>

---


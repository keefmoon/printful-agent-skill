# Country/State Code API

To create an order, you have to use country and state codes to specify the recipient address. Both country code and state code are mandatory for orders to the USA, Canada and Australia. For other countries only the country code is needed to create an order.

Country codes are based on the ISO 3166-1 alpha-2 standard and are two letters long.

State codes are based on the ISO 3166-2 standard by omitting the country code part of the code and are used only for the USA, Canada, Japan and Australia.

All state/country codes that Printful accepts can be listed by this API.


## Endpoints

### GET /countries
**Summary:** Retrieve country list

Returns list of countries and states that are accepted by the Printful.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCountries",
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
      "source": "curl --location --request GET 'https://api.printful.com/countries'"
    }
  ]
}
```
</details>

---


# Webhook API

Webhooks are an API feature that allows your system to receive notifications about certain events.

When an event occurs, the Printful server ([Webhook Simulator](https://www.printful.com/api/webhook-simulator))
will make a POST request to your defined
URL that will contain a JSON object in the request body.
Your server has to respond with HTTP status `2xx OK`,
otherwise, the request will be retried in increasing intervals
(after 1, 4, 16, 64, 256 and 1024 minutes).

The JSON object will always contain these attributes:
<table>
    <tr>
        <td><strong>type</strong></td>
        <td>string</td>
        <td>Event type</td>
    </tr>
    <tr>
        <td><strong>created</strong></td>
        <td>timestamp</td>
        <td>Event time</td>
    </tr>
    <tr>
        <td><strong>retries</strong></td>
        <td>integer</td>
        <td>Number of previous attempts to deliver this webhook event</td>
    </tr>
    <tr>
        <td><strong>store</strong></td>
        <td>integer</td>
        <td>ID of the store that the event occured to</td>
    </tr>
    <tr>
        <td><strong>data</strong></td>
        <td>Object</td>
        <td>Additional data, depending on the event type</td>
    </tr>
</table>

Please use [Webhook Simulator](https://www.printful.com/api/webhook-simulator) to test your webhook event receiver.

To set up webhooks, use API requests described below:

## Endpoints

### GET /webhooks
**Summary:** Get webhook configuration

Returns configured webhook URL and list of webhook event types enabled for the store

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getWebhooks",
  "security": [
    {
      "OAuth": [
        "webhooks"
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
                    "title": "WebhookInfo",
                    "properties": {
                      "url": {
                        "type": "string",
                        "description": "Webhook URL that will receive store's event notifications",
                        "example": "\u200bhttps://www.example.com/printful/webhook"
                      },
                      "types": {
                        "type": "array",
                        "description": "Array of enabled webhook event types",
                        "items": {
                          "type": "string"
                        },
                        "example": [
                          "package_shipped",
                          "stock_updated"
                        ]
                      },
                      "params": {
                        "type": "object",
                        "additionalProperties": true,
                        "example": {
                          "stock_updated": {
                            "product_ids": [
                              5,
                              12
                            ]
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
      "source": "curl --location --request GET 'https://api.printful.com/webhooks' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### POST /webhooks
**Summary:** Set up webhook configuration

Use this endpoint to enable a webhook URL for a store and select webhook event types that will be sent to this URL.

 Note that only one webhook URL can be active for a store, so calling this method disables all existing webhook configuration.

 Setting up the [Stock updated](#operation/stockUpdated) webhook requires passing IDs for products that need to be monitored for changes. Stock update webhook will only include information for specified products. These product IDs need to be set up using the params property.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createWebhook",
  "security": [
    {
      "OAuth": [
        "webhooks"
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
          "allOf": [
            {
              "title": "WebhookInfo",
              "properties": {
                "url": {
                  "type": "string",
                  "description": "Webhook URL that will receive store's event notifications",
                  "example": "\u200bhttps://www.example.com/printful/webhook"
                },
                "types": {
                  "type": "array",
                  "description": "Array of enabled webhook event types",
                  "items": {
                    "type": "string"
                  },
                  "example": [
                    "package_shipped",
                    "stock_updated"
                  ]
                },
                "params": {
                  "type": "object",
                  "additionalProperties": true,
                  "example": {
                    "stock_updated": {
                      "product_ids": [
                        5,
                        12
                      ]
                    }
                  }
                }
              }
            },
            {
              "required": [
                "url",
                "types"
              ]
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
                    "title": "WebhookInfo",
                    "properties": {
                      "url": {
                        "type": "string",
                        "description": "Webhook URL that will receive store's event notifications",
                        "example": "\u200bhttps://www.example.com/printful/webhook"
                      },
                      "types": {
                        "type": "array",
                        "description": "Array of enabled webhook event types",
                        "items": {
                          "type": "string"
                        },
                        "example": [
                          "package_shipped",
                          "stock_updated"
                        ]
                      },
                      "params": {
                        "type": "object",
                        "additionalProperties": true,
                        "example": {
                          "stock_updated": {
                            "product_ids": [
                              5,
                              12
                            ]
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
      "source": "curl --location --request POST 'https://api.printful.com/webhooks' \\\n--header 'Content-Type: application/json' \\\n--data-raw '{\n                \"url\": \"http://www.example.com/printful/webhook\",\n                \"types\": [\n                    \"package_shipped\",\n                    \"stock_updated\"\n                ],\n                \"params\": {\n                    \"stock_updated\": {\n                        \"product_ids\": [5, 12]\n                    }\n                }\n            }'"
    }
  ]
}
```
</details>

---

### DELETE /webhooks
**Summary:** Disable webhook support

Removes the webhook URL and all event types from the store.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "disableWebhook",
  "security": [
    {
      "OAuth": [
        "webhooks"
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
                    "title": "WebhookInfo",
                    "properties": {
                      "url": {
                        "type": "string",
                        "description": "Webhook URL that will receive store's event notifications",
                        "example": "\u200bhttps://www.example.com/printful/webhook"
                      },
                      "types": {
                        "type": "array",
                        "description": "Array of enabled webhook event types",
                        "items": {
                          "type": "string"
                        },
                        "example": [
                          "package_shipped",
                          "stock_updated"
                        ]
                      },
                      "params": {
                        "type": "object",
                        "additionalProperties": true,
                        "example": {
                          "stock_updated": {
                            "product_ids": [
                              5,
                              12
                            ]
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
      "source": "curl --location --request DELETE 'https://api.printful.com/webhooks' \\\n--header 'Content-Type: application/json'"
    }
  ]
}
```
</details>

---


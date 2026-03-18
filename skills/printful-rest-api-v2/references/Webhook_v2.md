# Webhook v2

Webhooks are an API feature that allows your system to receive notifications about certain events.

When an event occurs, for example one of the products you use comes back in stock, the Printful server
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
        <td><strong>occurred_at</strong></td>
        <td>timestamp</td>
        <td>Event time</td>
    </tr>
    <tr>
        <td><strong>retries</strong></td>
        <td>integer</td>
        <td>Number of previous attempts to deliver this webhook event</td>
    </tr>
    <tr>
        <td><strong>store_id</strong></td>
        <td>integer</td>
        <td>ID of the store that the event occurred to</td>
    </tr>
    <tr>
        <td><strong>data</strong></td>
        <td>Object</td>
        <td>Additional data, depending on the event type</td>
    </tr>
</table>

Currently, our [Webhook Simulator](https://www.printful.com/api/webhook-simulator) doesn't support the v2 webhooks.

### Event signing

Since the webhook URL used to accept the events needs to be publicly availably, it's important to be able to
verify if the event originates from our servers or was sent by someone else who knows the URL.

That's why we introduced event signing in Webhook v2.

Webhook settings are associated with a public and a secret key. The public key is used to identify the specific
settings (you may use the same URL for different configurations) and the secret key is used to generate
and verify the event signature.

We're using HMAC-SHA256 hash to create the signature. For tests, you may find an online or offline tool
and check if you get the same hash for the event data you receive and the configuration's secret key.

Please note that in the `secret_key` field we return the hexadecimal representation of the secret key. It means that you
need to decode it before passing it to the tool or method that calculates the signature, unless it supports keys in
hexadecimal format.

For actual usage, we strongly recommend to incorporate the signature verification in the event handling process.

If you find that you receive an event with invalid or missing signature, you should ignore the event.

You may investigate the situation, starting with verifying if your configuration is correct.
It's possible you used wrong secret key to generate the signature on your side or generated it for the formatted
event content instead of the raw content etc.

If you didn't find an explanation, you may contact us with the event details (content, your URL, event time)
so we can check if we sent such an event and find out why the signature is invalid.

### Expiration time

It's possible to specify the expiration time of the configuration. If you decide to use this option,
you will be forced to resubscribe which will generate a new key pair. You may simply send the GET request
when the configuration is still valid and use the returned data to create the new configuration.

If the configuration is already expired and you don't have the data saved somewhere, a regular GET request
will result in a 404 Not Found response with a message explaining what needs to be done:
"The settings are expired. To view them, please use query param show_expired with value true."

You may then use `show_expired` param with value `true` to retrieve the configuration and use it to resubscribe.

Please note that once the new configuration is saved, the new secret key will be used to sign the events.
It needs to be updated in your code as soon as possible to avoid invalid signature warnings.


## Endpoints

### GET /v2/webhooks
**Summary:** Get webhook configuration

Returns a configured webhook URL and a list of webhook event types enabled for the store

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getWebhooks",
  "security": [
    {
      "OAuth": [
        "webhooks/read"
      ]
    }
  ],
  "parameters": [
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
      "name": "show_expired",
      "description": "If this parameter is passed with value `true`, expired settings will be returned instead of a 404 warning.",
      "schema": {
        "type": "boolean"
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
                    "required": [
                      "default_url",
                      "expires_at",
                      "events"
                    ],
                    "properties": {
                      "default_url": {
                        "type": "string",
                        "nullable": true,
                        "description": "Webhook URL (HTTPS-only) that will receive store's event notifications if no URL is set for the event.",
                        "example": "\u200bhttps://www.example.com/printful/webhook"
                      },
                      "expires_at": {
                        "type": "string",
                        "format": "date-time",
                        "example": "2023-04-05T06:07:08Z",
                        "nullable": true,
                        "description": "Date-time indicating when the configuration will expire. The default value is `null` meaning it won't expire."
                      },
                      "events": {
                        "type": "array",
                        "description": "Array of enabled webhook event configurations",
                        "items": {
                          "title": "EventConfiguration",
                          "required": [
                            "type"
                          ],
                          "properties": {
                            "type": {
                              "type": "string",
                              "description": "Event type.",
                              "example": "catalog_stock_updated"
                            },
                            "url": {
                              "type": "string",
                              "nullable": true,
                              "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                              "example": "\u200bhttps://www.example.com/printful/webhook"
                            },
                            "params": {
                              "type": "array",
                              "items": {
                                "type": "object",
                                "title": "EventParam",
                                "properties": {
                                  "name": {
                                    "type": "string",
                                    "description": "Param name."
                                  },
                                  "value": {
                                    "description": "Param value."
                                  }
                                }
                              },
                              "example": [
                                {
                                  "name": "products",
                                  "value": [
                                    {
                                      "id": 1
                                    },
                                    {
                                      "id": 71
                                    }
                                  ]
                                }
                              ]
                            }
                          }
                        },
                        "example": [
                          {
                            "type": "shipment_sent",
                            "url": "\u200bhttps://www.example.com/printful/webhook/shipment_sent"
                          },
                          {
                            "type": "catalog_stock_updated",
                            "params": [
                              {
                                "name": "products",
                                "value": [
                                  {
                                    "id": 1
                                  },
                                  {
                                    "id": 71
                                  }
                                ]
                              }
                            ]
                          }
                        ]
                      },
                      "public_key": {
                        "type": "string",
                        "description": "Public key used to identify the specific settings.",
                        "example": "SbF/9d/uWguI"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/v2/webhooks' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### POST /v2/webhooks
**Summary:** Set up webhook configuration

Use this endpoint to enable a webhook URL for a store and select webhook event types that will be sent to this URL.

Note that only one webhook configuration can be active for each private OAuth token or app, calling this method will disable the previous webhook configuration.

Setting up the [Catalog stock updated](#operation/catalogStockUpdated) webhook requires passing products (currently only IDs are taken into account).

Stock update webhook (`catalog_stock_updated`) will only include information for the products specified in the `products` param.

Configuring all other events require the same set of fields with no parameters.


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
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
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
                "default_url": {
                  "type": "string",
                  "nullable": true,
                  "description": "Webhook URL (HTTPS-only) that will receive store's event notifications if no URL is set for the event.",
                  "example": "\u200bhttps://www.example.com/printful/webhook"
                },
                "expires_at": {
                  "type": "string",
                  "format": "date-time",
                  "example": "2023-04-05T06:07:08Z",
                  "nullable": true,
                  "description": "Date-time indicating when the configuration should expire. The default value is `null` meaning it won't expire."
                },
                "events": {
                  "type": "array",
                  "description": "Array of enabled webhook event configurations",
                  "items": {
                    "title": "EventConfiguration",
                    "type": "object",
                    "oneOf": [
                      {
                        "title": "DefaultEventConfiguration",
                        "required": [
                          "type"
                        ],
                        "properties": {
                          "type": {
                            "type": "string",
                            "description": "Event type. The dropdown shows values required for webhook event configuration. During the setup of a new webhook\nevent please check this dropdown if the event does not require additional parameters to be set.\n",
                            "example": "shipment_sent"
                          },
                          "url": {
                            "type": "string",
                            "nullable": true,
                            "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                            "example": "\u200bhttps://www.example.com/printful/webhook"
                          }
                        }
                      },
                      {
                        "title": "CatalogStockUpdatedEventConfiguration",
                        "allOf": [
                          {
                            "required": [
                              "params"
                            ]
                          },
                          {
                            "title": "DefaultEventConfiguration",
                            "required": [
                              "type"
                            ],
                            "properties": {
                              "type": {
                                "type": "string",
                                "description": "Event type. The dropdown shows values required for webhook event configuration. During the setup of a new webhook\nevent please check this dropdown if the event does not require additional parameters to be set.\n",
                                "example": "shipment_sent"
                              },
                              "url": {
                                "type": "string",
                                "nullable": true,
                                "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                                "example": "\u200bhttps://www.example.com/printful/webhook"
                              }
                            }
                          },
                          {
                            "properties": {
                              "params": {
                                "type": "array",
                                "items": {
                                  "oneOf": [
                                    {
                                      "title": "ProductsParam",
                                      "properties": {
                                        "name": {
                                          "type": "string",
                                          "description": "Param name.",
                                          "enum": [
                                            "products"
                                          ],
                                          "example": "products"
                                        },
                                        "value": {
                                          "description": "Param value - list of product data.",
                                          "type": "array",
                                          "items": {
                                            "title": "ProductData",
                                            "properties": {
                                              "id": {
                                                "type": "integer",
                                                "description": "Product ID.",
                                                "example": 1
                                              }
                                            }
                                          },
                                          "example": [
                                            {
                                              "id": 1
                                            },
                                            {
                                              "id": 71
                                            }
                                          ]
                                        }
                                      }
                                    }
                                  ],
                                  "discriminator": {
                                    "propertyName": "name",
                                    "mapping": {
                                      "products": "#/components/schemas/ProductsParam"
                                    }
                                  }
                                },
                                "example": [
                                  {
                                    "name": "products",
                                    "value": [
                                      {
                                        "id": 1
                                      },
                                      {
                                        "id": 71
                                      }
                                    ]
                                  }
                                ]
                              }
                            }
                          }
                        ]
                      }
                    ],
                    "discriminator": {
                      "propertyName": "type",
                      "mapping": {
                        "catalog_stock_updated": "#/components/schemas/CatalogStockUpdatedEventConfigurationRequest",
                        "shipment_sent": "#/components/schemas/DefaultEventConfigurationRequest",
                        "shipment_returned": "#/components/schemas/DefaultEventConfigurationRequest",
                        "shipment_out_of_stock": "#/components/schemas/DefaultEventConfigurationRequest",
                        "shipment_canceled": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_created": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_updated": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_failed": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_canceled": "#/components/schemas/DefaultEventConfigurationRequest",
                        "product_synced": "#/components/schemas/DefaultEventConfigurationRequest",
                        "product_updated": "#/components/schemas/DefaultEventConfigurationRequest",
                        "product_deleted": "#/components/schemas/DefaultEventConfigurationRequest",
                        "catalog_price_changed": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_put_hold": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_put_hold_approval": "#/components/schemas/DefaultEventConfigurationRequest",
                        "order_remove_hold": "#/components/schemas/DefaultEventConfigurationRequest",
                        "mockup_task_finished": "#/components/schemas/DefaultEventConfigurationRequest",
                        "shipment_put_hold": "#/components/schemas/DefaultEventConfigurationRequest",
                        "shipment_put_hold_approval": "#/components/schemas/DefaultEventConfigurationRequest",
                        "shipment_remove_hold": "#/components/schemas/DefaultEventConfigurationRequest"
                      }
                    }
                  },
                  "example": [
                    {
                      "type": "shipment_sent",
                      "url": "\u200bhttps://www.example.com/printful/webhook/shipment_sent"
                    },
                    {
                      "type": "catalog_stock_updated",
                      "params": [
                        {
                          "name": "products",
                          "value": [
                            {
                              "id": 1
                            },
                            {
                              "id": 71
                            }
                          ]
                        }
                      ]
                    }
                  ]
                }
              }
            },
            {
              "required": [
                "events"
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
                    "allOf": [
                      {
                        "required": [
                          "secret_key"
                        ]
                      },
                      {
                        "title": "WebhookInfo",
                        "required": [
                          "default_url",
                          "expires_at",
                          "events"
                        ],
                        "properties": {
                          "default_url": {
                            "type": "string",
                            "nullable": true,
                            "description": "Webhook URL (HTTPS-only) that will receive store's event notifications if no URL is set for the event.",
                            "example": "\u200bhttps://www.example.com/printful/webhook"
                          },
                          "expires_at": {
                            "type": "string",
                            "format": "date-time",
                            "example": "2023-04-05T06:07:08Z",
                            "nullable": true,
                            "description": "Date-time indicating when the configuration will expire. The default value is `null` meaning it won't expire."
                          },
                          "events": {
                            "type": "array",
                            "description": "Array of enabled webhook event configurations",
                            "items": {
                              "title": "EventConfiguration",
                              "required": [
                                "type"
                              ],
                              "properties": {
                                "type": {
                                  "type": "string",
                                  "description": "Event type.",
                                  "example": "catalog_stock_updated"
                                },
                                "url": {
                                  "type": "string",
                                  "nullable": true,
                                  "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                                  "example": "\u200bhttps://www.example.com/printful/webhook"
                                },
                                "params": {
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "EventParam",
                                    "properties": {
                                      "name": {
                                        "type": "string",
                                        "description": "Param name."
                                      },
                                      "value": {
                                        "description": "Param value."
                                      }
                                    }
                                  },
                                  "example": [
                                    {
                                      "name": "products",
                                      "value": [
                                        {
                                          "id": 1
                                        },
                                        {
                                          "id": 71
                                        }
                                      ]
                                    }
                                  ]
                                }
                              }
                            },
                            "example": [
                              {
                                "type": "shipment_sent",
                                "url": "\u200bhttps://www.example.com/printful/webhook/shipment_sent"
                              },
                              {
                                "type": "catalog_stock_updated",
                                "params": [
                                  {
                                    "name": "products",
                                    "value": [
                                      {
                                        "id": 1
                                      },
                                      {
                                        "id": 71
                                      }
                                    ]
                                  }
                                ]
                              }
                            ]
                          },
                          "public_key": {
                            "type": "string",
                            "description": "Public key used to identify the specific settings.",
                            "example": "SbF/9d/uWguI"
                          }
                        }
                      },
                      {
                        "properties": {
                          "secret_key": {
                            "type": "string",
                            "description": "A hexadecimal representation of the secret key used to generate and verify the event signature. Visible only once upon configuration set-up.",
                            "example": "a01376f30766acf7f706cc84beb57b79f9779e4a3e79d5519b3a9069c2156807098aw1d2abb43d356e03a46e71eeb9ab4519b3774d993abde8a3fabcebeb30fe"
                          }
                        }
                      }
                    ]
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/v2/webhooks' \\\n--header 'Content-Type: application/json' \\\n--data-raw '{\n                \"default_url\": \"http://www.example.com/printful/webhook\",\n                \"expires_at\": \"1700000000\",\n                \"events\": [\n                    {\n                        \"type\": \"shipment_sent\"\n                        \"url\": \"http://www.example.com/printful/webhook/shipment_sent\"\n                    },\n                    {\n                        \"type\": \"catalog_stock_updated\"\n                        \"params\": [\n                            {\n                                \"name\": \"products\",\n                                \"value\": [\n                                    {\n                                        \"id\": 5\n                                    },\n                                    {\n                                        \"id\": 12\n                                    }\n                                ]\n                            }\n                        ]\n                    }\n                ]\n            }'"
    }
  ]
}
```
</details>

---

### DELETE /v2/webhooks
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
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "responses": {
    "204": {
      "description": "No Content"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request DELETE 'https://api.printful.com/v2/webhooks' \\\n--header 'Content-Type: application/json'"
    }
  ]
}
```
</details>

---

### GET /v2/webhooks/{eventType}
**Summary:** Get event configuration

Returns event configuration for store

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getWebhookEventConfiguration",
  "security": [
    {
      "OAuth": [
        "webhooks/read"
      ]
    }
  ],
  "parameters": [
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
      "in": "path",
      "name": "eventType",
      "description": "Event type",
      "schema": {
        "type": "string"
      },
      "required": true
    },
    {
      "in": "query",
      "name": "show_expired",
      "description": "If this parameter is passed with value `true`, expired settings will be returned instead of a 404 warning.",
      "schema": {
        "type": "boolean"
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
                    "title": "EventConfiguration",
                    "required": [
                      "type"
                    ],
                    "properties": {
                      "type": {
                        "type": "string",
                        "description": "Event type.",
                        "example": "catalog_stock_updated"
                      },
                      "url": {
                        "type": "string",
                        "nullable": true,
                        "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                        "example": "\u200bhttps://www.example.com/printful/webhook"
                      },
                      "params": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "EventParam",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Param name."
                            },
                            "value": {
                              "description": "Param value."
                            }
                          }
                        },
                        "example": [
                          {
                            "name": "products",
                            "value": [
                              {
                                "id": 1
                              },
                              {
                                "id": 71
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/v2/webhooks/shipment_sent' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### POST /v2/webhooks/{eventType}
**Summary:** Set up event configuration

Use this endpoint to create or replace specific event configuration for a store.

Setting up the [Catalog stock updated](#operation/catalogStockUpdated) webhook requires passing products (currently only IDs are taken into account).

Stock update webhook will only include information for the products specified in the `products` param.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createWebhookEventConfiguration",
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
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "eventType",
      "description": "Event type",
      "schema": {
        "type": "string"
      },
      "required": true
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "title": "EventConfiguration",
          "type": "object",
          "oneOf": [
            {
              "title": "DefaultEventConfiguration",
              "required": [
                "type"
              ],
              "properties": {
                "type": {
                  "type": "string",
                  "description": "Event type. The dropdown shows values required for webhook event configuration. During the setup of a new webhook\nevent please check this dropdown if the event does not require additional parameters to be set.\n",
                  "example": "shipment_sent"
                },
                "url": {
                  "type": "string",
                  "nullable": true,
                  "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                  "example": "\u200bhttps://www.example.com/printful/webhook"
                }
              }
            },
            {
              "title": "CatalogStockUpdatedEventConfiguration",
              "allOf": [
                {
                  "required": [
                    "params"
                  ]
                },
                {
                  "title": "DefaultEventConfiguration",
                  "required": [
                    "type"
                  ],
                  "properties": {
                    "type": {
                      "type": "string",
                      "description": "Event type. The dropdown shows values required for webhook event configuration. During the setup of a new webhook\nevent please check this dropdown if the event does not require additional parameters to be set.\n",
                      "example": "shipment_sent"
                    },
                    "url": {
                      "type": "string",
                      "nullable": true,
                      "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                      "example": "\u200bhttps://www.example.com/printful/webhook"
                    }
                  }
                },
                {
                  "properties": {
                    "params": {
                      "type": "array",
                      "items": {
                        "oneOf": [
                          {
                            "title": "ProductsParam",
                            "properties": {
                              "name": {
                                "type": "string",
                                "description": "Param name.",
                                "enum": [
                                  "products"
                                ],
                                "example": "products"
                              },
                              "value": {
                                "description": "Param value - list of product data.",
                                "type": "array",
                                "items": {
                                  "title": "ProductData",
                                  "properties": {
                                    "id": {
                                      "type": "integer",
                                      "description": "Product ID.",
                                      "example": 1
                                    }
                                  }
                                },
                                "example": [
                                  {
                                    "id": 1
                                  },
                                  {
                                    "id": 71
                                  }
                                ]
                              }
                            }
                          }
                        ],
                        "discriminator": {
                          "propertyName": "name",
                          "mapping": {
                            "products": "#/components/schemas/ProductsParam"
                          }
                        }
                      },
                      "example": [
                        {
                          "name": "products",
                          "value": [
                            {
                              "id": 1
                            },
                            {
                              "id": 71
                            }
                          ]
                        }
                      ]
                    }
                  }
                }
              ]
            }
          ],
          "discriminator": {
            "propertyName": "type",
            "mapping": {
              "catalog_stock_updated": "#/components/schemas/CatalogStockUpdatedEventConfigurationRequest",
              "shipment_sent": "#/components/schemas/DefaultEventConfigurationRequest",
              "shipment_returned": "#/components/schemas/DefaultEventConfigurationRequest",
              "shipment_out_of_stock": "#/components/schemas/DefaultEventConfigurationRequest",
              "shipment_canceled": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_created": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_updated": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_failed": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_canceled": "#/components/schemas/DefaultEventConfigurationRequest",
              "product_synced": "#/components/schemas/DefaultEventConfigurationRequest",
              "product_updated": "#/components/schemas/DefaultEventConfigurationRequest",
              "product_deleted": "#/components/schemas/DefaultEventConfigurationRequest",
              "catalog_price_changed": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_put_hold": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_put_hold_approval": "#/components/schemas/DefaultEventConfigurationRequest",
              "order_remove_hold": "#/components/schemas/DefaultEventConfigurationRequest",
              "mockup_task_finished": "#/components/schemas/DefaultEventConfigurationRequest",
              "shipment_put_hold": "#/components/schemas/DefaultEventConfigurationRequest",
              "shipment_put_hold_approval": "#/components/schemas/DefaultEventConfigurationRequest",
              "shipment_remove_hold": "#/components/schemas/DefaultEventConfigurationRequest"
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
                    "title": "EventConfiguration",
                    "required": [
                      "type"
                    ],
                    "properties": {
                      "type": {
                        "type": "string",
                        "description": "Event type.",
                        "example": "catalog_stock_updated"
                      },
                      "url": {
                        "type": "string",
                        "nullable": true,
                        "description": "Webhook URL (HTTPS-only) that will receive the event notifications.",
                        "example": "\u200bhttps://www.example.com/printful/webhook"
                      },
                      "params": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "EventParam",
                          "properties": {
                            "name": {
                              "type": "string",
                              "description": "Param name."
                            },
                            "value": {
                              "description": "Param value."
                            }
                          }
                        },
                        "example": [
                          {
                            "name": "products",
                            "value": [
                              {
                                "id": 1
                              },
                              {
                                "id": 71
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/v2/webhooks/shipment_sent' \\\n--header 'Content-Type: application/json' \\\n--data-raw '{\n                \"type\": \"shipment_sent\"\n                \"url\": \"http://www.example.com/printful/webhook/shipment_sent\"\n            }'"
    }
  ]
}
```
</details>

---

### DELETE /v2/webhooks/{eventType}
**Summary:** Disable support for event

Disables the event for a store and clears its configuration, leaving other webhooks intact.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "disableWebhookEvent",
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
      "description": "Use this to specify which store you want to use (required only for account level token).\n\nThe store IDs can be retrieved with the [Get basic information about stores](#tag/Stores-v2/operation/getStores) endpoint.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    },
    {
      "in": "path",
      "name": "eventType",
      "description": "Event type",
      "schema": {
        "type": "string"
      },
      "required": true
    }
  ],
  "responses": {
    "204": {
      "description": "No Content"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request DELETE 'https://api.printful.com/v2/webhooks/shipment_sent' \\\n--header 'Content-Type: application/json'"
    }
  ]
}
```
</details>

---


# Store Information API

 

## Endpoints

### POST /store/packing-slip
**Summary:** Change packing slip

Modifies packing slip information of the currently authorized Printful store.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "changePackingSlip",
  "security": [
    {
      "OAuth": [
        "stores_list"
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
          "type": "object",
          "title": "OrderPackingSlip",
          "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
          "anyOf": [
            {
              "required": [
                "email"
              ]
            },
            {
              "required": [
                "phone"
              ]
            },
            {
              "required": [
                "message"
              ]
            },
            {
              "required": [
                "custom_order_id"
              ]
            }
          ],
          "properties": {
            "email": {
              "type": "string",
              "description": "Customer service email",
              "example": "your-name@your-domain.com"
            },
            "phone": {
              "type": "string",
              "description": "Customer service phone",
              "example": "+371 28888888"
            },
            "message": {
              "type": "string",
              "description": "Custom packing slip message",
              "example": "Message on packing slip"
            },
            "logo_url": {
              "type": "string",
              "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
              "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
            },
            "store_name": {
              "type": "string",
              "description": "Store name override for the return address",
              "example": "Your store name"
            },
            "custom_order_id": {
              "type": "string",
              "description": "Your own Order ID that will be printed instead of Printful's Order ID",
              "example": "kkk2344lm"
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
                "properties": {
                  "packing_slip": {
                    "type": "object",
                    "title": "OrderPackingSlip",
                    "description": "Custom packing slip for this order. Example of a packing slip with explained fields can be found [here](#packing-slip).",
                    "anyOf": [
                      {
                        "required": [
                          "email"
                        ]
                      },
                      {
                        "required": [
                          "phone"
                        ]
                      },
                      {
                        "required": [
                          "message"
                        ]
                      },
                      {
                        "required": [
                          "custom_order_id"
                        ]
                      }
                    ],
                    "properties": {
                      "email": {
                        "type": "string",
                        "description": "Customer service email",
                        "example": "your-name@your-domain.com"
                      },
                      "phone": {
                        "type": "string",
                        "description": "Customer service phone",
                        "example": "+371 28888888"
                      },
                      "message": {
                        "type": "string",
                        "description": "Custom packing slip message",
                        "example": "Message on packing slip"
                      },
                      "logo_url": {
                        "type": "string",
                        "description": "URL address to a sticker we will put on a package. The provided image is converted to grayscale/1-bit monochrome image.",
                        "example": "\u200bhttp://www.your-domain.com/packing-logo.png"
                      },
                      "store_name": {
                        "type": "string",
                        "description": "Store name override for the return address",
                        "example": "Your store name"
                      },
                      "custom_order_id": {
                        "type": "string",
                        "description": "Your own Order ID that will be printed instead of Printful's Order ID",
                        "example": "kkk2344lm"
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
    }
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/store/packing-slip' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"email\": \"email@example.com\",\n    \"phone\": \"+1 1 11111111\",\n    \"message\": \"Custom message\",\n}'"
    }
  ]
}
```
</details>

---

### GET /stores
**Summary:** Get basic information about stores

Get basic information about stores depending on the token access level

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStores",
  "security": [
    {
      "OAuth": [
        "stores_list"
      ]
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
              },
              {
                "properties": {
                  "result": {
                    "type": "array",
                    "items": {
                      "allOf": [
                        {
                          "type": "object",
                          "description": "Information about the Store",
                          "properties": {
                            "id": {
                              "description": "Store ID",
                              "type": "integer",
                              "example": 10
                            },
                            "type": {
                              "description": "Store type",
                              "type": "string",
                              "example": "native"
                            },
                            "name": {
                              "description": "Store name",
                              "type": "string",
                              "example": "My Store"
                            }
                          }
                        },
                        {
                          "type": "object"
                        }
                      ]
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
    }
  }
}
```
</details>

---

### GET /stores/{id}
**Summary:** Get basic information about a store

Get basic information about a store based on provided ID

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getStore",
  "security": [
    {
      "OAuth": [
        "stores_list"
      ]
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
      "description": "Store ID"
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
                    "allOf": [
                      {
                        "type": "object",
                        "description": "Information about the Store",
                        "properties": {
                          "id": {
                            "description": "Store ID",
                            "type": "integer",
                            "example": 10
                          },
                          "type": {
                            "description": "Store type",
                            "type": "string",
                            "example": "native"
                          },
                          "name": {
                            "description": "Store name",
                            "type": "string",
                            "example": "My Store"
                          }
                        }
                      },
                      {
                        "type": "object"
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
    }
  }
}
```
</details>

---


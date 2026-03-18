# OAuth API

OAuth API allows receiving data for token

## Endpoints

### GET /oauth/scopes
**Summary:** Get scopes for token

Returns a list of scopes associated with the token

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getScopes",
  "security": [
    {
      "OAuth": []
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
                    "type": "object",
                    "properties": {
                      "scopes": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "properties": {
                            "scope": {
                              "type": "string",
                              "example": "orders/read"
                            },
                            "display_name": {
                              "type": "string",
                              "example": "View all orders"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/oauth/scopes' --header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---


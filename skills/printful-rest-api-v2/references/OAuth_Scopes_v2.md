# OAuth Scopes v2

OAuth API allows receiving data for token

## Endpoints

### GET /v2/oauth-scopes
**Summary:** Retrieve OAuth scopes

This endpoint will retrieve all OAuth scopes associated with the used token

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getOAuthScopes",
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
            "required": [
              "data",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "OAuthScope",
                  "description": "Information about the OAuth scope",
                  "required": [
                    "name",
                    "value"
                  ],
                  "properties": {
                    "name": {
                      "description": "Display name of the scope",
                      "type": "string",
                      "example": "View all orders",
                      "enum": [
                        "View orders of the authorized store",
                        "View and manage orders of the authorized store",
                        "View store products",
                        "View and manage store products",
                        "View store files",
                        "View and manage store files",
                        "View store webhooks",
                        "View and manage store webhooks"
                      ]
                    },
                    "value": {
                      "description": "The scope value",
                      "type": "string",
                      "example": "orders/read",
                      "enum": [
                        "orders/read",
                        "orders",
                        "sync_products/read",
                        "sync_products",
                        "file_library/read",
                        "file_library",
                        "webhooks/read",
                        "webhooks/read"
                      ]
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
                    "description": "Link to the order details",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/oauth-scopes"
                    },
                    "allOf": [
                      {
                        "type": "object",
                        "title": "HateoasLink",
                        "readOnly": true,
                        "required": [
                          "href"
                        ],
                        "properties": {
                          "href": {
                            "description": "The HREF of the linked resource.",
                            "type": "string"
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
  }
}
```
</details>

---


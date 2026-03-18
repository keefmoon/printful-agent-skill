# Approval Sheets v2

Get information about approval sheets.

## Endpoints

### GET /v2/approval-sheets
**Summary:** Retrieve a list of approval sheets

Retrieve a list of approval sheets confirming suggested changes to files of on hold orders.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getApprovalSheets",
  "security": [
    {
      "OAuth": [
        "files"
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
      "name": "order_id",
      "schema": {
        "type": "integer"
      },
      "description": "Order ID."
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
                  "title": "ApprovalSheet",
                  "description": "Approval sheet",
                  "required": [
                    "confirm_hash",
                    "status",
                    "submitted_design",
                    "recommended_design",
                    "approval_sheet",
                    "order_id",
                    "order_item_id",
                    "_links"
                  ],
                  "properties": {
                    "confirm_hash": {
                      "type": "string",
                      "description": "Confirmation hash value.",
                      "example": "a14e51714be01f98487fcf5131727d31"
                    },
                    "status": {
                      "type": "string",
                      "description": "Status of Approval Sheet.",
                      "enum": [
                        "waiting_for_action",
                        "approval_pending",
                        "approved",
                        "changes_requested",
                        "files_changed"
                      ],
                      "example": "waiting_for_action"
                    },
                    "submitted_design": {
                      "type": "string",
                      "description": "URL to submitted design.",
                      "example": "https://s3.staging.printful.com/upload/approval-design/ae/ae7b3d3e965c238b3e5c1a4e15696f07_l"
                    },
                    "recommended_design": {
                      "type": "string",
                      "description": "URL to recommended design.",
                      "example": "https://s3.staging.printful.com/upload/approval-design/aa/aaf9e1c6b32cb7a2c04d2746108d4124_l"
                    },
                    "approval_sheet": {
                      "type": "string",
                      "description": "URL to Approval sheet.",
                      "example": "\u200bhttps://example.com/approval-sheet.pdf"
                    },
                    "order_id": {
                      "type": "integer",
                      "description": "Order ID.",
                      "example": 123
                    },
                    "order_item_id": {
                      "type": "integer",
                      "description": "Item ID.",
                      "example": 123
                    },
                    "_links": {
                      "type": "object",
                      "title": "Approval sheet links",
                      "description": "HATEOAS links",
                      "required": [
                        "order",
                        "order_item"
                      ],
                      "properties": {
                        "order": {
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
                            },
                            {
                              "description": "Link to a specific order."
                            },
                            {
                              "properties": {
                                "href": {
                                  "example": "/v2/orders/123"
                                }
                              }
                            }
                          ]
                        },
                        "order_item": {
                          "allOf": [
                            {
                              "$ref": "circular reference to #/components/schemas/HateoasLink"
                            },
                            {
                              "description": "Link to a specific order item."
                            },
                            {
                              "properties": {
                                "href": {
                                  "example": "/v2/orders/123/order-items/123"
                                }
                              }
                            }
                          ]
                        }
                      }
                    }
                  }
                }
              },
              "_links": {
                "type": "object",
                "title": "Approval sheet links",
                "description": "HATEOAS links",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
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
                    ],
                    "description": "Link to the same resource",
                    "example": {
                      "href": "https://api.printful.com/v2/approval-sheets?limit=10&offset=20"
                    }
                  },
                  "next": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ],
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page).",
                    "example": {
                      "href": "https://api.printful.com/v2/approval-sheets?limit=10&offset=30"
                    }
                  },
                  "previous": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ],
                    "description": "Link to the previous page (absent if the current page is the first one or if there is only one page).",
                    "example": {
                      "href": "https://api.printful.com/v2/approval-sheets?limit=10&offset=10"
                    }
                  },
                  "first": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ],
                    "description": "Link to the first page (absent if there is only one page).",
                    "example": {
                      "href": "https://api.printful.com/v2/approval-sheets?limit=10"
                    }
                  },
                  "last": {
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ],
                    "description": "Link to the last page (absent if there is only one page).",
                    "example": {
                      "href": "https://api.printful.com/v2/approval-sheets?limit=10&offset=90"
                    }
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


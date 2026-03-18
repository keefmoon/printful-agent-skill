# Approval Sheets API

Approval Sheets API

## Endpoints

### GET /approval-sheets
**Summary:** Retrieve a list of approval sheets

Retrieve a list of approval sheets confirming suggested changes to files of on hold orders.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getApprovalSheets",
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
                    "type": "array",
                    "items": {
                      "type": "object",
                      "title": "ApprovalSheet",
                      "description": "Approval sheet",
                      "properties": {
                        "id": {
                          "type": "integer",
                          "example": 2
                        },
                        "status": {
                          "type": "string",
                          "enum": [
                            "waiting_for_action",
                            "approved"
                          ],
                          "example": "waiting_for_action"
                        },
                        "confirm_hash": {
                          "type": "string",
                          "example": "a14e51714be01f98487fcf5131727d31"
                        },
                        "submitted_design": {
                          "type": "string",
                          "example": "https://s3.staging.printful.com/upload/approval-design/ae/ae7b3d3e965c238b3e5c1a4e15696f07_l"
                        },
                        "recommended_design": {
                          "type": "string",
                          "example": "https://s3.staging.printful.com/upload/approval-design/aa/aaf9e1c6b32cb7a2c04d2746108d4124_l"
                        },
                        "approval_sheet": {
                          "type": "string",
                          "example": "https://www.printful.test/dashboard/order/download-approval-sheet-pdf?confirmationHash=13aa35854bfc67a85b7ce231aef2ae8"
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
  }
}
```
</details>

---

### POST /approval-sheets
**Summary:** Approve a design

Uses the confirm hash of an approval sheet to approve a design and remove the hold on an order

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "approveDesign",
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
      "name": "confirm_hash",
      "schema": {
        "type": "string"
      },
      "required": true,
      "description": "The confirm hash for the approval sheet you would like to approve.",
      "example": "a14e51714be01f98487fcf5131727d31"
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "Approval Sheet",
          "description": "Change the status of the approval sheet",
          "required": [
            "status"
          ],
          "properties": {
            "status": {
              "type": "string",
              "example": "approved",
              "enum": [
                "approved"
              ]
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
                    "type": "object",
                    "properties": {
                      "message": {
                        "type": "string",
                        "example": "Design approval submitted successfully"
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
  }
}
```
</details>

---

### POST /approval-sheets/changes
**Summary:** Submit changes to an approval sheet

Use this to submit alternative changes to a design that has an approval sheet

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "submitApprovalSheetChanges",
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
      "name": "confirm_hash",
      "schema": {
        "type": "string"
      },
      "required": true,
      "description": "The confirm hash for the approval sheet you would like to approve.",
      "example": "a14e51714be01f98487fcf5131727d31"
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "title": "Approval Sheet Changes",
          "description": "Changes to a design in an Approval sheet",
          "required": [
            "message",
            "files"
          ],
          "properties": {
            "message": {
              "type": "string",
              "description": "A message to send to Printful designers. Customers should use this to describe the changes they want.",
              "example": "The design needs to be aligned to the left"
            },
            "files": {
              "type": "array",
              "description": "An array of images to help describe the requested changes. Consider using the mockup generator to generate these images. The array is required but can be empty if you do not want to email any images.",
              "items": {
                "type": "object",
                "required": [
                  "url"
                ],
                "properties": {
                  "url": {
                    "type": "string",
                    "description": "A url to an image, consider using the mockup generator to generate this image.",
                    "example": "example.com"
                  }
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
                    "title": "ApprovalSheet",
                    "description": "Approval sheet",
                    "properties": {
                      "id": {
                        "type": "integer",
                        "example": 2
                      },
                      "status": {
                        "type": "string",
                        "enum": [
                          "waiting_for_action",
                          "approved"
                        ],
                        "example": "waiting_for_action"
                      },
                      "confirm_hash": {
                        "type": "string",
                        "example": "a14e51714be01f98487fcf5131727d31"
                      },
                      "submitted_design": {
                        "type": "string",
                        "example": "https://s3.staging.printful.com/upload/approval-design/ae/ae7b3d3e965c238b3e5c1a4e15696f07_l"
                      },
                      "recommended_design": {
                        "type": "string",
                        "example": "https://s3.staging.printful.com/upload/approval-design/aa/aaf9e1c6b32cb7a2c04d2746108d4124_l"
                      },
                      "approval_sheet": {
                        "type": "string",
                        "example": "https://www.printful.test/dashboard/order/download-approval-sheet-pdf?confirmationHash=13aa35854bfc67a85b7ce231aef2ae8"
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
  }
}
```
</details>

---


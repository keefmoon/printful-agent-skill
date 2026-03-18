# Product Templates API

The Product Templates API resource lets you retrieve the product templates information.

### External Product ID

In case of a single template retrieval it is possible to get it by the External Product ID. In order to do this, the ID needs to be prepended with the '@' character. Here are the examples of how to get the template data by the Template ID and by the External Product ID.

```
GET /product-templates/11001  - reference by Printful Template ID
GET /product-templates/@988123  - reference by External ID
```

[See examples](#tag/Product-Templates-API)


## Endpoints

### GET /product-templates
**Summary:** Get product template list

Returns a list of templates.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductTemplates",
  "security": [
    {
      "OAuth": [
        "product_templates/read"
      ]
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer"
      },
      "required": false,
      "description": "Result set offset"
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer"
      },
      "required": false,
      "description": "Number of items per page (max 100)"
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
                    "title": "Product template",
                    "description": "Information about the template",
                    "properties": {
                      "items": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "Product template",
                          "description": "Information about the template",
                          "properties": {
                            "id": {
                              "type": "integer"
                            },
                            "product_id": {
                              "type": "integer"
                            },
                            "external_product_id": {
                              "type": "string"
                            },
                            "title": {
                              "type": "string"
                            },
                            "available_variant_ids": {
                              "type": "array",
                              "items": {
                                "type": "integer"
                              }
                            },
                            "option_data": {
                              "type": "array",
                              "items": {
                                "type": "object",
                                "properties": {
                                  "id": {
                                    "type": "string"
                                  },
                                  "value": {
                                    "type": "array",
                                    "items": {
                                      "type": "string"
                                    }
                                  }
                                }
                              }
                            },
                            "colors": {
                              "type": "array",
                              "items": {
                                "type": "object",
                                "properties": {
                                  "color_name": {
                                    "type": "string"
                                  },
                                  "color_codes": {
                                    "type": "array"
                                  }
                                }
                              }
                            },
                            "sizes": {
                              "type": "array",
                              "items": {
                                "type": "string"
                              }
                            },
                            "mockup_file_url": {
                              "type": "string"
                            },
                            "placements": {
                              "type": "array",
                              "items": {
                                "type": "object",
                                "properties": {
                                  "placement": {
                                    "type": "string"
                                  },
                                  "display_name": {
                                    "type": "string"
                                  },
                                  "technique_key": {
                                    "type": "string",
                                    "example": "SUBLIMATION"
                                  },
                                  "technique_display_name": {
                                    "type": "string",
                                    "example": "Sublimation"
                                  }
                                }
                              }
                            },
                            "created_at": {
                              "type": "integer"
                            },
                            "updated_at": {
                              "type": "integer"
                            },
                            "placement_option_data": {
                              "type": "array",
                              "items": {
                                "type": "object",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "example": "embroidery_chest_left"
                                  },
                                  "options": {
                                    "type": "array",
                                    "items": {
                                      "type": "object",
                                      "properties": {
                                        "id": {
                                          "type": "string",
                                          "example": "full_color"
                                        },
                                        "value": {
                                          "example": true
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
                    }
                  },
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
      "source": "curl --location --request GET 'https://api.printful.com/product-templates' \\\n--header 'Authorization: Bearer {oauth_token}'\n"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Get 10 templates\n$orders = $pf->get('product-templates', ['limit' => 10]);"
    }
  ]
}
```
</details>

---

### GET /product-templates/{id}
**Summary:** Get product template

Get information about a single product template

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getProductTemplateById",
  "security": [
    {
      "OAuth": [
        "product_templates/read"
      ]
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Template ID (integer) or External Product ID (if prefixed with @)"
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
                    "title": "Product template",
                    "description": "Information about the template",
                    "properties": {
                      "id": {
                        "type": "integer"
                      },
                      "product_id": {
                        "type": "integer"
                      },
                      "external_product_id": {
                        "type": "string"
                      },
                      "title": {
                        "type": "string"
                      },
                      "available_variant_ids": {
                        "type": "array",
                        "items": {
                          "type": "integer"
                        }
                      },
                      "option_data": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "properties": {
                            "id": {
                              "type": "string"
                            },
                            "value": {
                              "type": "array",
                              "items": {
                                "type": "string"
                              }
                            }
                          }
                        }
                      },
                      "colors": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "properties": {
                            "color_name": {
                              "type": "string"
                            },
                            "color_codes": {
                              "type": "array"
                            }
                          }
                        }
                      },
                      "sizes": {
                        "type": "array",
                        "items": {
                          "type": "string"
                        }
                      },
                      "mockup_file_url": {
                        "type": "string"
                      },
                      "placements": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "properties": {
                            "placement": {
                              "type": "string"
                            },
                            "display_name": {
                              "type": "string"
                            },
                            "technique_key": {
                              "type": "string",
                              "example": "SUBLIMATION"
                            },
                            "technique_display_name": {
                              "type": "string",
                              "example": "Sublimation"
                            }
                          }
                        }
                      },
                      "created_at": {
                        "type": "integer"
                      },
                      "updated_at": {
                        "type": "integer"
                      },
                      "placement_option_data": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "properties": {
                            "id": {
                              "type": "string",
                              "example": "embroidery_chest_left"
                            },
                            "options": {
                              "type": "array",
                              "items": {
                                "type": "object",
                                "properties": {
                                  "id": {
                                    "type": "string",
                                    "example": "full_color"
                                  },
                                  "value": {
                                    "example": true
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
      "source": "curl --location --request GET 'https://api.printful.com/product-templates/{template_id}' \\\n--header 'Authorization: Bearer {oauth_token}'\n"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Get template with id 12345\n$order = $pf->get('product-templates/12345');"
    }
  ]
}
```
</details>

---

### DELETE /product-templates/{id}
**Summary:** Delete product template

Delete product template by ID or External Product ID

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "deleteProductTemplate",
  "security": [
    {
      "OAuth": [
        "product_templates"
      ]
    }
  ],
  "parameters": [
    {
      "in": "path",
      "name": "id",
      "schema": {
        "oneOf": [
          {
            "type": "integer"
          },
          {
            "type": "string"
          }
        ]
      },
      "required": true,
      "description": "Template ID (integer) or External Product ID (if prefixed with @)"
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
                      "success": {
                        "type": "boolean",
                        "description": "Whether the deletion was successful"
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
      "source": "curl --location --request DELETE 'https://api.printful.com/product-templates/{template_id}' \\\n--header 'Authorization: Bearer {oauth_token}'"
    },
    {
      "lang": "PrintfulSdk",
      "source": "<?php\n\n// Replace this with your API key\n$apiKey = '';\n\n$pf = new PrintfulApiClient($apiKey);\n\n// Get template with id 12345\n$order = $pf->delete('product-templates/12345');"
    }
  ]
}
```
</details>

---


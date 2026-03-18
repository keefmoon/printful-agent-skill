# Shipping Rates v2

<div class="alert alert-info">
<p><strong>Note:</strong> The shipping rates endpoints are meant to be called only right before placing an order to display
available shipping rates and methods.</p>

Dynamic shipping rates can change even in the span of one hour because it takes live facility and carrier information
into account. Different rates may be calculated for different products, quantities and recipient data.

Even daily downloads of this data, reused only for identical orders, can result in mismatches between the displayed
rates and the charged rates, potentially resulting in customer dissatisfaction.

A CSV file containing flat rates data for different categories of products may be downloaded
from https://www.printful.com/shipping-rates-report/shipping-rates-report/download and hardcoded to reduce the shipping
rate volume massively if you use flat rate shipping.
</div>

**Rate limiting:** The default rate limit is 120 requests per 60 seconds.

<div class="alert alert-danger">
  <strong>Warning:</strong> If the summary item quantity count exceeds 100 then the rate limit
is changed to 5 requests per 60 seconds.
</div>

A 60 seconds lockout is applied if request count is exceeded.

### Recipient in shipping rates estimates

When making a shipping rates estimate the only data that is necessary is the country code, and state code in the case
of the case of the United States, Canada, and Australia. This allows for an estimate to be retrieved for a
customer before their address is known, however the more information provided the more accurate the estimate will be.


## Endpoints

### POST /v2/shipping-rates
**Summary:** Calculate Shipping Rates

Returns available shipping options and rates for the given list of products.

**Recipient Address Requirements:**
- Only `country_code` is required in the recipient object
- `state_code` is only required for United States (US), Australia (AU), and Canada (CA)
- All other recipient fields are optional

**Note:** Providing more address information may produce more precise results and more shipping options. While only the country code is required, including additional details like city, postal code, and state/province can help return more accurate shipping rates and additional delivery options.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "calculateShppingRates",
  "security": [
    {
      "OAuth": [
        "orders"
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
      "in": "header",
      "name": "X-PF-Language",
      "description": "Use this to specify which locale you would like to use in the responses, for some endpoints this can affect translations.\n",
      "schema": {
        "type": "string"
      },
      "required": false
    }
  ],
  "requestBody": {
    "description": "POST request body",
    "content": {
      "application/json": {
        "schema": {
          "required": [
            "recipient",
            "order_items"
          ],
          "type": "object",
          "properties": {
            "recipient": {
              "description": "The recipient data.",
              "allOf": [
                {
                  "type": "object",
                  "title": "Address",
                  "description": "Information about the address.\n\n**Required fields:**\n- `country_code`: Always required\n\n**Conditionally required fields:**\n- `state_code`: Required for United States (US), Australia (AU), and Canada (CA)\n\n**Optional fields:**\n- All other fields are optional but providing more information may produce more precise results and more shipping options.\n",
                  "required": [
                    "country_code"
                  ],
                  "properties": {
                    "address1": {
                      "description": "Address line 1",
                      "type": "string",
                      "example": "19749 Dearborn St"
                    },
                    "address2": {
                      "description": "Address line 2",
                      "type": "string",
                      "nullable": true
                    },
                    "city": {
                      "description": "City",
                      "type": "string",
                      "example": "Chatsworth"
                    },
                    "state_code": {
                      "description": "State/province code. Required for United States (US), Australia (AU), and Canada (CA).\nFor other countries, this field is optional.\n",
                      "type": "string",
                      "example": "CA",
                      "nullable": true
                    },
                    "country_code": {
                      "description": "Country code",
                      "type": "string",
                      "example": "US"
                    },
                    "zip": {
                      "description": "ZIP/Postal code",
                      "type": "string",
                      "example": "91311",
                      "nullable": true
                    }
                  }
                }
              ]
            },
            "order_items": {
              "type": "array",
              "description": "Array of order items",
              "items": {
                "oneOf": [
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Item",
                        "required": [
                          "catalog_variant_id",
                          "quantity",
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Catalog source",
                            "type": "string",
                            "enum": [
                              "catalog"
                            ]
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "minimum": 1,
                            "maximum": 1000,
                            "example": 1
                          },
                          "catalog_variant_id": {
                            "description": "ID of catalog variant",
                            "type": "integer",
                            "example": 4011
                          },
                          "placements": {
                            "description": "Each entry in this list represents a placement on the physical product.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Placement",
                              "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used.",
                              "required": [
                                "placement",
                                "technique"
                              ],
                              "properties": {
                                "placement": {
                                  "description": "Name of the placement",
                                  "type": "string",
                                  "example": "front"
                                },
                                "technique": {
                                  "description": "Placement's technique",
                                  "type": "string",
                                  "example": "dtg"
                                },
                                "print_area_type": {
                                  "description": "Type of the print area. Defaults to simple. Advanced type might be specified only for some specific products\nfor example All-over Tote bag. In that case both sides of the product will be designed.\nAdvanced designs are often more complicated so it's advised to refer to the [mockup templates endpoint](#tag/Catalog-v2/operation/getMockupTemplatesByProductId) when using advanced designs.\nAvailable print area types for a specific product can be found in [Mockup Styles endpoint](/docs/#operation/createMockupGeneratorTasks).\n",
                                  "type": "string",
                                  "writeOnly": true,
                                  "default": "simple",
                                  "enum": [
                                    "simple",
                                    "advanced"
                                  ]
                                },
                                "placement_options": {
                                  "description": "List of placement options",
                                  "type": "array",
                                  "items": {
                                    "oneOf": [
                                      {
                                        "type": "object",
                                        "title": "UnlimitedColorOption",
                                        "description": "Specify if the design should use unlimited color technique",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "unlimited_color",
                                            "enum": [
                                              "unlimited_color"
                                            ]
                                          },
                                          "value": {
                                            "description": "Whether the design should use unlimited color technique.",
                                            "type": "boolean",
                                            "example": true
                                          }
                                        }
                                      },
                                      {
                                        "type": "object",
                                        "title": "InsideLabelTypeOption",
                                        "description": "Specify the type of inside label",
                                        "required": [
                                          "name",
                                          "value"
                                        ],
                                        "properties": {
                                          "name": {
                                            "description": "Option identifier",
                                            "type": "string",
                                            "example": "inside_label_type",
                                            "enum": [
                                              "inside_label_type"
                                            ]
                                          },
                                          "value": {
                                            "description": "Specifies type of inside label design that should be used",
                                            "type": "string",
                                            "enum": [
                                              "native",
                                              "custom"
                                            ]
                                          }
                                        }
                                      }
                                    ]
                                  }
                                }
                              }
                            }
                          },
                          "product_options": {
                            "description": "List of product options",
                            "type": "array",
                            "items": {
                              "oneOf": [
                                {
                                  "type": "object",
                                  "title": "InsidePocketOption",
                                  "description": "Specify if inside pocket should be added to the product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "inside_pocket",
                                      "enum": [
                                        "inside_pocket"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether inside pocket should be added to the product.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "StitchColorOption",
                                  "description": "Specified what color should be used for stitches",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "stitch_color",
                                      "enum": [
                                        "stitch_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "white"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "NotesOption",
                                  "description": "Include additional notes for fulfillment for embroidery prints",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "notes",
                                      "enum": [
                                        "notes"
                                      ]
                                    },
                                    "value": {
                                      "description": "Additional notes for fulfillment for embroidery prints.",
                                      "type": "string",
                                      "example": "Please make sure that top side of the print is within print area"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "LifelikeOption",
                                  "description": "Specifies if generated mockup should use lifelike effect",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "lifelike",
                                      "enum": [
                                        "lifelike"
                                      ]
                                    },
                                    "value": {
                                      "description": "Whether generated mockup should use lifelike effect.",
                                      "type": "boolean",
                                      "example": true
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "CustomBorderColorOption",
                                  "description": "Used to specify the border color of a sticker",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "custom_border_color",
                                      "enum": [
                                        "custom_border_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Color defined in hexadecimal format",
                                      "type": "string",
                                      "example": "#FF0000"
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearBaseColorOption",
                                  "description": "Used to specify the base color on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "base_color",
                                      "enum": [
                                        "base_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearTrimColorOption",
                                  "description": "Used to specify the color of the trim on a knitwear product.",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "trim_color",
                                      "enum": [
                                        "trim_color"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "example": "#fdfafa",
                                      "enum": [
                                        "#090909",
                                        "#404040",
                                        "#563c33",
                                        "#d52213",
                                        "#6e5242",
                                        "#7f6a53",
                                        "#cd5e38",
                                        "#b57648",
                                        "#d1773b",
                                        "#d68785",
                                        "#c6b5a7",
                                        "#d6c6b4",
                                        "#dcd3cc",
                                        "#edd9d9",
                                        "#e2dfdc",
                                        "#fdfafa",
                                        "#999996",
                                        "#dda032",
                                        "#d1c6ae",
                                        "#eddea4",
                                        "#48542e",
                                        "#6e8c4b",
                                        "#c0c1bd",
                                        "#243f33",
                                        "#c5d1d0",
                                        "#175387",
                                        "#237d96",
                                        "#787979",
                                        "#343d55",
                                        "#4e59be",
                                        "#566e99",
                                        "#504372",
                                        "#4c1c29",
                                        "#f66274",
                                        "#eda6b4",
                                        "#ddabc8"
                                      ]
                                    }
                                  }
                                },
                                {
                                  "type": "object",
                                  "title": "KnitwearColorReductionMode",
                                  "description": "Used to set the color reduction mode for a knitwear product",
                                  "required": [
                                    "name",
                                    "value"
                                  ],
                                  "properties": {
                                    "name": {
                                      "description": "Option identifier",
                                      "type": "string",
                                      "example": "color_reduction_mode",
                                      "enum": [
                                        "color_reduction_mode"
                                      ]
                                    },
                                    "value": {
                                      "description": "Option value",
                                      "type": "string",
                                      "enum": [
                                        "solid",
                                        "pixelated"
                                      ],
                                      "example": "pixelated"
                                    }
                                  }
                                }
                              ],
                              "discriminator": {
                                "propertyName": "name",
                                "mapping": {
                                  "inside_pocket": "#/components/schemas/InsidePocketOption",
                                  "stitch_color": "#/components/schemas/StitchColorOption",
                                  "notes": "#/components/schemas/NotesOption",
                                  "lifelike": "#/components/schemas/LifelikeOption",
                                  "custom_border_color": "#/components/schemas/CustomBorderColorOption",
                                  "base_color": "#/components/schemas/KnitwearBaseColor",
                                  "trim_color": "#/components/schemas/KnitwearTrimColor",
                                  "color_reduction_mode": "#/components/schemas/KnitwearColorReductionMode"
                                }
                              }
                            }
                          }
                        }
                      }
                    ]
                  },
                  {
                    "allOf": [
                      {
                        "type": "object",
                        "title": "Item",
                        "required": [
                          "warehouse_variant_id",
                          "quantity",
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Warehouse source",
                            "type": "string",
                            "enum": [
                              "catalog"
                            ]
                          },
                          "quantity": {
                            "description": "Item quantity",
                            "type": "integer",
                            "minimum": 1,
                            "maximum": 1000,
                            "example": 1
                          },
                          "warehouse_variant_id": {
                            "description": "ID of catalog variant",
                            "type": "integer",
                            "example": 4011
                          }
                        }
                      }
                    ]
                  }
                ],
                "discriminator": {
                  "propertyName": "source",
                  "mapping": {
                    "catalog": "#/components/schemas/CatalogShippingRateItem",
                    "warehouse": "#/components/schemas/WarehouseShippingRateItem"
                  }
                }
              }
            },
            "currency": {
              "type": "string",
              "description": "Currency code in which the rate is returned. If not provided will use the default currency for the store."
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "required": [
                    "shipping",
                    "shipping_method_name",
                    "rate",
                    "currency",
                    "shipments"
                  ],
                  "type": "object",
                  "properties": {
                    "shipping": {
                      "type": "string",
                      "description": "Shipping method ID",
                      "example": "STANDARD"
                    },
                    "shipping_method_name": {
                      "type": "string",
                      "description": "A descriptive name of the shipping method including an estimate that can be displayed to customers",
                      "example": "Flat Rate (Estimated delivery: May 19\u201324) "
                    },
                    "rate": {
                      "type": "string",
                      "description": "Shipping rate",
                      "example": "13.60"
                    },
                    "currency": {
                      "type": "string",
                      "description": "Currency code in which the rate is returned",
                      "example": "EUR"
                    },
                    "min_delivery_days": {
                      "type": "integer",
                      "description": "Estimated minimum delivery days. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": 4
                    },
                    "max_delivery_days": {
                      "type": "integer",
                      "description": "Estimated maximum delivery days. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": 7
                    },
                    "min_delivery_date": {
                      "type": "string",
                      "format": "date",
                      "description": "Estimated minimum delivery date. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": "2022-10-17"
                    },
                    "max_delivery_date": {
                      "type": "string",
                      "format": "date",
                      "description": "Estimated maximum delivery date. <span style=\"color:orange\">Warning! This value may not be present in response.</span>",
                      "example": "2022-10-20"
                    },
                    "shipments": {
                      "type": "array",
                      "description": "List of shipments.",
                      "items": {
                        "type": "object",
                        "title": "Shipment",
                        "description": "Basic info about the shipment.",
                        "required": [
                          "departure_country",
                          "shipment_items",
                          "customs_fees_possible"
                        ],
                        "properties": {
                          "departure_country": {
                            "description": "Two-letter code (ISO 3166-1 alpha-2) associated with the departure country.",
                            "type": "string",
                            "example": "US",
                            "nullable": true
                          },
                          "shipment_items": {
                            "description": "List of items included in the shipment.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "ShipmentItem",
                              "description": "Basic info about the shipment item.",
                              "required": [
                                "catalog_variant_id",
                                "quantity"
                              ],
                              "properties": {
                                "catalog_variant_id": {
                                  "type": "number",
                                  "format": "integer",
                                  "description": "Catalog Variant ID of the shipment item.",
                                  "example": 4011
                                },
                                "quantity": {
                                  "description": "List of items included in the shipment.",
                                  "type": "number",
                                  "format": "integer",
                                  "example": 3
                                }
                              }
                            }
                          },
                          "customs_fees_possible": {
                            "description": "Whether customs fees may be required for this shipment.",
                            "type": "boolean"
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
    "400": {
      "description": "Bad Request",
      "content": {
        "application/json": {
          "schema": {
            "type": "object",
            "properties": {
              "data": {
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


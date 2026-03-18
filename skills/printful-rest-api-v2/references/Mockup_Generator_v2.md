# Mockup Generator v2

This resource is used to visualize how the designed products will look like. Since this is a resource-heavy operation we process the requests asynchronously. That means that you won't get the results
immediately, and instead you will get task ID which you could use to retrieve the mockups once they are done
using _[Retrieve mockup generator tasks](#operation/getMockupGeneratorTasks)_.
You can also subscribe to a webhook _[Mockup generator task finished event](#operation/mockupTaskFinished)_ to be
notified once the tasks finish processing.

To create a mockup you need to specify:
* catalog product ID – _[Retrieve a list of catalog products](#operation/getProducts)_
* catalog variant IDs – _[Retrieve information about catalog product variants](#operation/getProductVariantsById)_
* design data – Data used to specify how the design should look like on the catalog product
* mockup styles – _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_

While `placements` specifies how the catalog product should look like, the `mockup_style_ids` define
how the mockups themselves should look like. You can find available mockup styles for a product by requesting
_[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.

To give a little bit more context on how to use product mockup styles see the table below:

The example product is Bella Canvas 3001 with ID 71.


<table>
    <tr>
        <td> Mockup styles </td> 
        <td> Mockups </td>
    </tr>
    <tr>
<tr>
<td>

```
    {
        "style_id": 1115,
        "style_name": "Men's",
        "view_name": "Front"
    }
```

</td>
<td> <img alt="Men's front mockup" src="images/mockup-styles/mens_front.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 1103,
        "style_name": "Men's",
        "view_name": "Left sleeve"
    }
```

</td>
<td> <img alt="Men's left sleeve mockup" src="images/mockup-styles/mens_left.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 1117,
        "style_name": "Women's",
        "view_name": "Front"
    }
```

</td>
<td> <img alt="Women's front mockup" src="images/mockup-styles/womens_front.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 1107,
        "style_name": "Women's",
        "view_name": "Left sleeve"
    }
```

</td>
<td> <img alt="Women's left sleeve mockup" src="images/mockup-styles/womens_left.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 768,
        "style_name": "Women's Lifestyle",
        "view_name": "Front"
    }
```

</td>
<td> <img alt="Women's lifestyle front mockup" src="images/mockup-styles/womens_lifestyle_front.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 770,
        "style_name": "Women's Lifestyle",
        "view_name": "Left"
    }
```

</td>
<td> <img alt="Women's lifestyle left mockup" src="images/mockup-styles/womens_lifestyle_left.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 849,
        "style_name": "Flat",
        "view_name": "Front"
    }
```

</td>
<td> <img alt="Flat front mockup" src="images/mockup-styles/flat_front.png" width="300"/> </td>
</tr>

<tr>
<td>

```
    {
        "style_id": 931,
        "style_name": "On hanger",
        "view_name": "Front"
    }
```

</td>
<td> <img alt="On hanger front mockup" src="images/mockup-styles/on_hanger_front.png" width="300"/> </td>
</tr>
</table>

## Mockups from the product templates

Instead of creating a mockups from catalog variants data you can generate them from a product template.
In that case all you need is:
* product template ID
* catalog variant IDs – _[Retrieve information about catalog product variants](#operation/getProductVariantsById)_
* mockup styles – _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_

**Important**: All the catalog variants specified in the request must also be defined in the product template,
otherwise an error will be returned.

## Throttling and daily file limit

An account is allowed to make a number of mockup generation requests per minute. The limit is:

* 2 requests per minute for new stores;
* 10 requests per minute for stores with at least $10.00 worth of fulfilled orders.

Additionally, there is a limit of 20,000 files that can be generated daily by each environment.

## Number of generated files

There will be one mockup file returned for each valid combination of Catalog Variant, mockup style and placement within
one product object.

If the same file would be generated for two or more combinations (for example, for different sizes of a T-Shirt in the
same color),
it will only be generated once and the same URL will be returned for each of these combinations. If only one file is generated it will count
as one file toward the daily limit.

## Endpoints

### POST /v2/mockup-tasks
**Summary:** Create Mockup Generator tasks

Create Mockup Generator tasks

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createMockupGeneratorTasks",
  "security": [
    {
      "OAuth": []
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
    "description": "This action schedules asynchronous mockup generation tasks. In the response of this request `id` will be returned which can be used to retrieve the result of the tasks _[Retrieve mockup generator tasks](#operation/getMockupGeneratorTasks)_. In addition, the webhook event will be sent informing about the task completion, if the correct webhook has been subscribed to _[Mockup generator task finished event](#operation/mockupTaskFinished)_. You can subscribe to webhook events by using _[Set up event configuration](#operation/createWebhookEventConfiguration)_.",
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "required": [
            "products"
          ],
          "properties": {
            "format": {
              "type": "string",
              "description": "Generated file format. PNG will have a transparent background, JPG will have a smaller file size.",
              "enum": [
                "jpg",
                "png"
              ]
            },
            "mockup_width_px": {
              "type": "integer",
              "description": "Width of the mockup image in pixels. If not specified, the default value will be used. The default value is 1000px.\nIf the value is specified as 2000 that means that resulting mockups will be 2000x2000px.\n",
              "example": 1000
            },
            "products": {
              "type": "array",
              "items": {
                "type": "object",
                "title": "MockupProduct",
                "oneOf": [
                  {
                    "type": "object",
                    "required": [
                      "catalog_product_id",
                      "placements"
                    ],
                    "allOf": [
                      {
                        "type": "object",
                        "required": [
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Mockup product source",
                            "type": "string",
                            "example": "catalog"
                          },
                          "mockup_style_ids": {
                            "type": "array",
                            "description": "Used to specify style of mockups that should be generated.\nFor example:\n  * On the hanger\n  * On the Male/Female model\n  * Flat on the table\n  * etc.\n\nAvailable mockup styles for catalog product can be found under\n_[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.\n\n**Default:** Automatically set to the first available style ID.\nIf `orientation` is specified, only styles matching that orientation will be considered.\n",
                            "items": {
                              "type": "integer",
                              "example": 123
                            }
                          }
                        }
                      },
                      {
                        "properties": {
                          "catalog_product_id": {
                            "description": "Catalog product identifier",
                            "type": "integer",
                            "example": 71
                          },
                          "catalog_variant_ids": {
                            "type": "array",
                            "description": "List of catalog variant identifiers for which mockups will be generated.\nVariants must belong to the catalog product specified with `catalog_product_id`.\nMultiple variants of the same color do not count towards the daily number of files limit.\nMeaning that for a Red T-Shirt of (S,M) sizes we still generate only one mockup.\n\n**Default:** Automatically set to the first active variant ID associated with the given `catalog_product_id`.\n",
                            "items": {
                              "type": "integer",
                              "example": 4011
                            }
                          },
                          "orientation": {
                            "type": "string",
                            "enum": [
                              "vertical",
                              "horizontal"
                            ],
                            "nullable": true,
                            "description": "Define orientation of the product (used in case of e.g. canvas). This parameter:\n* is ignored for products without orientation\n* helps select default `mockup_style_ids` with the given orientation\n* must be compatible with any explicitly defined `mockup_style_ids`\n* should be set to enforce the same orientation for all layer coordinates, disabling automatic orientation detection based on image size\n"
                          },
                          "placements": {
                            "description": "Each entry in this list represents a placement on the physical product and the design that will be printed in that location.",
                            "type": "array",
                            "items": {
                              "type": "object",
                              "title": "Placement",
                              "description": "A placement is used to represent the physical area in which a design will be printed, and the technique used. It includes the layers that will be printed on the placement.",
                              "required": [
                                "placement",
                                "technique",
                                "layers",
                                "status",
                                "status_explanation"
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
                                "layers": {
                                  "description": "Information about placement's layers",
                                  "type": "array",
                                  "items": {
                                    "type": "object",
                                    "title": "Layer",
                                    "description": "Information about the Layer",
                                    "required": [
                                      "type",
                                      "url"
                                    ],
                                    "properties": {
                                      "type": {
                                        "description": "Type of layer (e.g. file, text)",
                                        "type": "string",
                                        "example": "file"
                                      },
                                      "url": {
                                        "description": "File image URL if layer type is file",
                                        "type": "string",
                                        "example": "\u200bhttps://www.printful.com/static/images/layout/printful-logo.png"
                                      },
                                      "layer_options": {
                                        "description": "List of layer options",
                                        "type": "array",
                                        "items": {
                                          "oneOf": [
                                            {
                                              "type": "object",
                                              "title": "3dPuffOption",
                                              "description": "Should thread use 3d puff technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "3d_puff",
                                                  "enum": [
                                                    "3d_puff"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Whether the 3d puff technique should be used for the layer.",
                                                  "type": "boolean",
                                                  "example": true
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "ThreadColorsOption",
                                              "description": "Specify thread colors for embroidery technique",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "thread_colors",
                                                  "enum": [
                                                    "thread_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Thread colors for embroidery technique",
                                                  "type": "array",
                                                  "items": {
                                                    "type": "string",
                                                    "example": "#FFFFFF"
                                                  }
                                                }
                                              }
                                            },
                                            {
                                              "type": "object",
                                              "title": "KnitwearYarnColorsOption",
                                              "description": "Used to specify the yarn colors for the whole product. These are the colors that will be used across the whole product.",
                                              "required": [
                                                "name",
                                                "value"
                                              ],
                                              "properties": {
                                                "name": {
                                                  "description": "Option identifier",
                                                  "type": "string",
                                                  "example": "yarn_colors",
                                                  "enum": [
                                                    "yarn_colors"
                                                  ]
                                                },
                                                "value": {
                                                  "description": "Option value",
                                                  "type": "array",
                                                  "items": {
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
                                              }
                                            }
                                          ]
                                        }
                                      },
                                      "position": {
                                        "type": "object",
                                        "title": "LayerPosition",
                                        "description": "Information about the Layer position. If the positions are not provided then the design will be automatically centered.",
                                        "required": [
                                          "top",
                                          "left",
                                          "width",
                                          "height"
                                        ],
                                        "properties": {
                                          "width": {
                                            "description": "Layer width in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "height": {
                                            "description": "Layer height in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "minimum": 0.3,
                                            "example": 10
                                          },
                                          "top": {
                                            "description": "Layer top position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          },
                                          "left": {
                                            "description": "Layer left position in inches",
                                            "type": "number",
                                            "format": "float",
                                            "multipleOf": 0.001,
                                            "example": 0
                                          }
                                        }
                                      }
                                    }
                                  }
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
                                },
                                "status": {
                                  "type": "string",
                                  "readOnly": true,
                                  "enum": [
                                    "ok",
                                    "failed"
                                  ],
                                  "description": "Status of the placement design"
                                },
                                "status_explanation": {
                                  "type": "string",
                                  "readOnly": true,
                                  "example": "Product with ID: 656 cannot have disjointed design elements.",
                                  "description": "Reason behind failed status"
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
                    "type": "object",
                    "required": [
                      "source",
                      "product_template_id"
                    ],
                    "allOf": [
                      {
                        "type": "object",
                        "required": [
                          "source"
                        ],
                        "properties": {
                          "source": {
                            "description": "Mockup product source",
                            "type": "string",
                            "example": "catalog"
                          },
                          "mockup_style_ids": {
                            "type": "array",
                            "description": "Used to specify style of mockups that should be generated.\nFor example:\n  * On the hanger\n  * On the Male/Female model\n  * Flat on the table\n  * etc.\n\nAvailable mockup styles for catalog product can be found under\n_[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.\n\n**Default:** Automatically set to the first available style ID.\nIf `orientation` is specified, only styles matching that orientation will be considered.\n",
                            "items": {
                              "type": "integer",
                              "example": 123
                            }
                          }
                        }
                      },
                      {
                        "properties": {
                          "source": {
                            "description": "Mockup product source",
                            "type": "string",
                            "example": "template"
                          },
                          "product_template_id": {
                            "description": "Product Template identifier",
                            "type": "integer",
                            "example": 712152512
                          },
                          "catalog_variant_ids": {
                            "type": "array",
                            "description": "List of catalog variant identifiers for which mockups will be generated. Variants must be defined in the product template specified with `product_template_id`. Multiple variants of the same color do not count towards the daily number of files limit. Meaning that for a Red T-Shirt of (S,M) sizes we still generate only one mockup.",
                            "items": {
                              "type": "integer",
                              "example": 12457
                            }
                          },
                          "mockup_style_ids": {
                            "type": "array",
                            "description": "Used to specify style of mockups that should be generated. For example:\n  * On the hanger\n  * On the Male/Female model\n  * Flat on the table\n  * etc.\nAvailable mockup styles for catalog product can be found under _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.\n",
                            "items": {
                              "type": "integer",
                              "example": 123
                            }
                          }
                        }
                      }
                    ]
                  }
                ],
                "discriminator": {
                  "propertyName": "source",
                  "mapping": {
                    "catalog": "#/components/schemas/CatalogMockupProduct",
                    "product_template": "#/components/schemas/TemplateMockupProduct"
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
            "required": [
              "data"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "MockupGeneratorTask",
                  "description": "Result of mockup generator task",
                  "required": [
                    "id",
                    "status",
                    "catalog_variant_mockups",
                    "items",
                    "failure_reasons",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "type": "integer",
                      "description": "Unique task identifier used to check status of the task and retrieve the results once the task is ready.",
                      "example": 597350033
                    },
                    "status": {
                      "type": "string",
                      "enum": [
                        "completed",
                        "pending",
                        "failed"
                      ],
                      "description": "Task status:\n * `completed` \u2013 Mockup Generator task was successfully processed\n * `pending` \u2013 Mockup Generator task is still being processed\n * `failed` \u2013 Mockup Generator task failed\n"
                    },
                    "catalog_variant_mockups": {
                      "type": "array",
                      "description": "A list of mockups grouped by variant. Note that the same list of mockups can appear under multiple variants, this happens in cases where the variants have the same mockups, for example if the only difference is the size of the variant.",
                      "items": {
                        "type": "object",
                        "required": [
                          "catalog_variant_id",
                          "mockups"
                        ],
                        "properties": {
                          "catalog_variant_id": {
                            "type": "integer",
                            "description": "ID of a catalog variant for which the mockup was generated.",
                            "example": 4011
                          },
                          "mockups": {
                            "type": "array",
                            "description": "List of generated mockups",
                            "items": {
                              "type": "object",
                              "title": "Mockup",
                              "description": "Result of mockup generator tasks.",
                              "required": [
                                "placement",
                                "display_name",
                                "technique",
                                "style_id",
                                "mockup_url"
                              ],
                              "properties": {
                                "placement": {
                                  "type": "string",
                                  "description": "Placement name for which the mockup was generated",
                                  "example": "front"
                                },
                                "display_name": {
                                  "type": "string",
                                  "description": "This is a name that can be displayed to end customers.",
                                  "example": "Front Print"
                                },
                                "technique": {
                                  "type": "string",
                                  "description": "Technique name for which the mockup was generated",
                                  "example": "dtg"
                                },
                                "style_id": {
                                  "type": "integer",
                                  "description": "Mockup style identifier. Available mockup styles can be found under _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.",
                                  "example": 1
                                },
                                "mockup_url": {
                                  "type": "string",
                                  "description": "Temporary URL to generated mockup image. Image will be removed from the hosting after a day so make sure to persist a copy if needed.",
                                  "example": "https://printful-upload.s3-accelerate.amazonaws.com/tmp/9c711aabb422cd386da3cb41735069f3/unisex-staple-t-shirt-white-front-659e8b94763bc.png"
                                }
                              }
                            }
                          }
                        }
                      }
                    },
                    "failure_reasons": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "Error",
                        "readOnly": true,
                        "properties": {
                          "type": {
                            "type": "string",
                            "description": "a URI that uniquely identifies the validation rule that failed. If it\u2019s a URL, it should point to an explanation of the constraint in the documentation.",
                            "example": "https://developers.printful.com/docs/v2/errors#specific-validation-error"
                          },
                          "detail": {
                            "type": "string",
                            "description": "A human-readable explanation of the error",
                            "example": "Parameter `xyz` was incorrect"
                          },
                          "source": {
                            "description": "Source of the value that caused the issue",
                            "oneOf": [
                              {
                                "type": "object",
                                "properties": {
                                  "header": {
                                    "type": "string",
                                    "description": "Name of the header which is incorrect",
                                    "example": "X-PF-Language"
                                  }
                                }
                              },
                              {
                                "type": "object",
                                "properties": {
                                  "parameter": {
                                    "type": "string",
                                    "description": "Name of the URL query parameter that is incorrect",
                                    "example": "limit"
                                  }
                                }
                              },
                              {
                                "type": "object",
                                "properties": {
                                  "pointer": {
                                    "type": "string",
                                    "description": "Pointer to an invalid value in request body",
                                    "example": "/order_items/0/placements"
                                  }
                                }
                              }
                            ]
                          },
                          "valid_values": {
                            "type": "array",
                            "description": "List of valid values that could be used instead to avoid the error",
                            "items": {
                              "type": "string",
                              "example": "Front"
                            }
                          }
                        }
                      }
                    },
                    "_links": {
                      "type": "object",
                      "description": "HATEOAS links",
                      "readOnly": true,
                      "required": [
                        "self"
                      ],
                      "properties": {
                        "self": {
                          "description": "Link to the Generator Task details",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/mockup-tasks?id=597350033"
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

### GET /v2/mockup-tasks
**Summary:** Retrieve Mockup Generator tasks

Returns result of Mockup Generator tasks

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getMockupGeneratorTasks",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "id",
      "schema": {
        "type": "array",
        "items": {
          "type": "string"
        }
      },
      "explode": false,
      "examples": {
        "oneId": {
          "summary": "Return single specified mockup generator task result.",
          "value": "421699980"
        },
        "multipleIds": {
          "summary": "Return multiple specified mockup generator task results.",
          "value": "421699980,421699981"
        }
      },
      "required": true,
      "description": "One or more mockup generator task IDs to return only specified tasks. The IDs can be found in the response of the operation _[Create mockup generator tasks](#operation/createMockupGeneratorTasks)_."
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer",
        "minimum": 1,
        "maximum": 100,
        "default": 20
      },
      "required": false,
      "description": "The number of results to return per page."
    },
    {
      "in": "query",
      "name": "offset",
      "schema": {
        "type": "integer",
        "minimum": 0,
        "default": 0
      },
      "required": false,
      "examples": {
        "startAt100": {
          "summary": "Return results after the initial 100",
          "value": 100
        }
      },
      "description": "The number of results to not include in the response starting from the beginning of the list.\n\nThis can be used to return results after the initial 100. For example, sending offset 100\n"
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
              "paging",
              "_links"
            ],
            "properties": {
              "data": {
                "type": "array",
                "items": {
                  "type": "object",
                  "title": "MockupGeneratorTask",
                  "description": "Result of mockup generator task",
                  "required": [
                    "id",
                    "status",
                    "catalog_variant_mockups",
                    "items",
                    "failure_reasons",
                    "_links"
                  ],
                  "properties": {
                    "id": {
                      "type": "integer",
                      "description": "Unique task identifier used to check status of the task and retrieve the results once the task is ready.",
                      "example": 597350033
                    },
                    "status": {
                      "type": "string",
                      "enum": [
                        "completed",
                        "pending",
                        "failed"
                      ],
                      "description": "Task status:\n * `completed` \u2013 Mockup Generator task was successfully processed\n * `pending` \u2013 Mockup Generator task is still being processed\n * `failed` \u2013 Mockup Generator task failed\n"
                    },
                    "catalog_variant_mockups": {
                      "type": "array",
                      "description": "A list of mockups grouped by variant. Note that the same list of mockups can appear under multiple variants, this happens in cases where the variants have the same mockups, for example if the only difference is the size of the variant.",
                      "items": {
                        "type": "object",
                        "required": [
                          "catalog_variant_id",
                          "mockups"
                        ],
                        "properties": {
                          "catalog_variant_id": {
                            "type": "integer",
                            "description": "ID of a catalog variant for which the mockup was generated.",
                            "example": 4011
                          },
                          "mockups": {
                            "type": "array",
                            "description": "List of generated mockups",
                            "items": {
                              "type": "object",
                              "title": "Mockup",
                              "description": "Result of mockup generator tasks.",
                              "required": [
                                "placement",
                                "display_name",
                                "technique",
                                "style_id",
                                "mockup_url"
                              ],
                              "properties": {
                                "placement": {
                                  "type": "string",
                                  "description": "Placement name for which the mockup was generated",
                                  "example": "front"
                                },
                                "display_name": {
                                  "type": "string",
                                  "description": "This is a name that can be displayed to end customers.",
                                  "example": "Front Print"
                                },
                                "technique": {
                                  "type": "string",
                                  "description": "Technique name for which the mockup was generated",
                                  "example": "dtg"
                                },
                                "style_id": {
                                  "type": "integer",
                                  "description": "Mockup style identifier. Available mockup styles can be found under _[Retrieve catalog product mockup styles](#operation/retrieveMockupStylesByProductId)_.",
                                  "example": 1
                                },
                                "mockup_url": {
                                  "type": "string",
                                  "description": "Temporary URL to generated mockup image. Image will be removed from the hosting after a day so make sure to persist a copy if needed.",
                                  "example": "https://printful-upload.s3-accelerate.amazonaws.com/tmp/9c711aabb422cd386da3cb41735069f3/unisex-staple-t-shirt-white-front-659e8b94763bc.png"
                                }
                              }
                            }
                          }
                        }
                      }
                    },
                    "failure_reasons": {
                      "type": "array",
                      "items": {
                        "type": "object",
                        "title": "Error",
                        "readOnly": true,
                        "properties": {
                          "type": {
                            "type": "string",
                            "description": "a URI that uniquely identifies the validation rule that failed. If it\u2019s a URL, it should point to an explanation of the constraint in the documentation.",
                            "example": "https://developers.printful.com/docs/v2/errors#specific-validation-error"
                          },
                          "detail": {
                            "type": "string",
                            "description": "A human-readable explanation of the error",
                            "example": "Parameter `xyz` was incorrect"
                          },
                          "source": {
                            "description": "Source of the value that caused the issue",
                            "oneOf": [
                              {
                                "type": "object",
                                "properties": {
                                  "header": {
                                    "type": "string",
                                    "description": "Name of the header which is incorrect",
                                    "example": "X-PF-Language"
                                  }
                                }
                              },
                              {
                                "type": "object",
                                "properties": {
                                  "parameter": {
                                    "type": "string",
                                    "description": "Name of the URL query parameter that is incorrect",
                                    "example": "limit"
                                  }
                                }
                              },
                              {
                                "type": "object",
                                "properties": {
                                  "pointer": {
                                    "type": "string",
                                    "description": "Pointer to an invalid value in request body",
                                    "example": "/order_items/0/placements"
                                  }
                                }
                              }
                            ]
                          },
                          "valid_values": {
                            "type": "array",
                            "description": "List of valid values that could be used instead to avoid the error",
                            "items": {
                              "type": "string",
                              "example": "Front"
                            }
                          }
                        }
                      }
                    },
                    "_links": {
                      "type": "object",
                      "description": "HATEOAS links",
                      "readOnly": true,
                      "required": [
                        "self"
                      ],
                      "properties": {
                        "self": {
                          "description": "Link to the Generator Task details",
                          "title": "HateoasLink",
                          "example": {
                            "href": "https://api.printful.com/v2/mockup-tasks?id=597350033"
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
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
                    "description": "Link to the same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/mockup-tasks?id=1,2,3,4,5,6,7,8,9,10&limit=5"
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
                  },
                  "next": {
                    "description": "Link to the next page (absent if the current page is the last one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/mockup-tasks?limit=5&offset=60"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "previous": {
                    "description": "Link to the previous page (absent if the current page is the first one or if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/mockup-tasks?limit=5&offset=20"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "first": {
                    "description": "Link to the first page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/mockup-tasks?limit=5"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
                      }
                    ]
                  },
                  "last": {
                    "description": "Link to the last page (absent if there is only one page)",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/mockup-tasks?limit=5&offset=100"
                    },
                    "allOf": [
                      {
                        "$ref": "circular reference to #/components/schemas/HateoasLink"
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


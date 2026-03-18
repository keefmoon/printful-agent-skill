# Mockup Generator API

To generate mockups, first, you need to decide on which products you want them. API methods on retrieving products and
variants can be found in the [Catalog API](#tag/Catalog-API).

**Note**: Remember to distinguish the difference between a product id and a variant id. Some API endpoints require an id
from a variant and some from a product.

**Important**: Jewelry products are not supported via API.

### Print files

A print file defines resolution which should be used to create a mockup or to submit an actual order.

Information about product variant print files can be retrieved from the [print file endpoint](#operation/getPrintfiles).

For example, a 10×10 poster requires a 1500×1500 pixel print file to produce a 150 DPI print. You can use higher
resolution files to achieve a better result, but keep the side aspect ratio the same as the defined for the print file.
That means, if you use a 3000×3000 pixel file, it will produce a 300 DPI print. But if you use a 3000×1500 pixel file (
different aspect ratio) on a 10×10 poster, some cropping will occur. Print file's `fill_mode` parameter defines if
cropping will happen, or the file will be fitted on the resulting print area of the product.

Some print files can be rotated. `can_rotate` field defines this feature. This mostly applies to wall art products and
should be used if you want to generate a horizontal or a vertical product mockup.

Wall art print files are defined horizontally. If you wish to create a vertical mockup, you can rotate the file's print
file and the generated mockup will be in the given orientation. For example, 16×12 poster print file is 2400×1800 pixels
which generate it horizontally. If you wish to get a vertical mockup, you create the print file as 1800×2400 pixels. The
same strategy applies when you submit an order.

Print files are often re-used for multiple variants and products. For example, a 14×14 poster uses the same print file
as a framed poster. Most of the t-shirt front prints use the same print file too.

**Note**: When you generate mockups there is no need to provide a full-sized print file. Mockups are generated up to
2000px wide, so you can downscale your print file to 2000px. This will reduce the processing time on your and Printful's
side. Print file image file size limit: 50MB.

### Mockup generation

Mockup generation requires some time, that is why it cannot happen in real-time.

When you request a mockup to be generated, a task is created and you receive the task key which can then be used to
retrieve the generated mockup list. We cannot guarantee that after a certain time the mockups will be generated, so you
will have to check frequently if the task is done. The first request for a result should not be sooner than 10 seconds.
So plan that the generation task will be done in two steps - creating a task and the checking with intervals if the task
is ready.

<div class="alert alert-info">
<strong>Important</strong><br>
URLs to mockup images are temporary, they will expire after 72h, so you have to store them on your
server.
</div>

### Process flow

1. Decide which product variants you want to generate.

2. Retrieve the list of print files for chosen product and variants. Use the variant print file mappings to determine
   which print file you need to generate for specific placement on a specific product's variant.

3. Upload your file to a public URL that matches the print file size ratio (or provide positions for the generation
   request)

4. Create a mockup generation task and store the task key.

5. Use the task key to check if the task is completed. If still pending, repeat after an interval.

6. When the task is done, download and store mockups on your server. Mockup URLs are temporary and will be removed after
   a day.

### Available techniques

The `/mockup-generator/printfiles/{id}` and `/mockup-generator/templates/{id}` endpoint accept `technique` parameter.

The following table presents the available values of this parameter.

| Value         | Description           |
|---------------|-----------------------|
| `DIGITAL`     | Digital printing      |
| `CUT-SEW`     | Cut & sew sublimation |
| `UV`          | UV printing           |
| `EMBROIDERY`  | Embroidery            |
| `SUBLIMATION` | Sublimation           |
| `ENGRAVING`   | Engraving             |
| `DTG`         | DTG printing          |

### Usage example

Let's take an example. You want to offer users to design of their t-shirt.

We'll pick this shirt as an example
[Bella + Canvas 3001 Unisex T-shirt](https://www.printful.com/custom/mens/t-shirts/unisex-staple-t-shirt-bella-canvas-3001)

Its product id is `71`.

Let's fetch some variants available for this shirt:
`https://api.printful.com/products/71`

We'll choose a white and black shirt in M, L, XL sizes. Respective variant ids:
`4012`, `4013`, `4014`, `4017`, `4018` and `4019`.

Next, we need to get the print file sizes for these variants:
`https://api.printful.com/mockup-generator/printfiles/71`

We see that there are two placements available for this product - `front` and `back`. Posters, for example, will only
have one placement called `default`.

By looking up our picked variant ids, we see that they all use the same print file for back and front prints:

```json
{
  "product_id": 71,
  "available_placements": {
    "front": "Front print",
    "back": "Back print",
    "label_outside": "Outside label"
  },
  "printfiles": [
    {
      "printfile_id": 1,
      "width": 1800,
      "height": 2400,
      "dpi": 150,
      "fill_mode": "fit",
      "can_rotate": false
    }
  ],
  "variant_printfiles": [
    {
      "variant_id": 4012,
      "placements": {
        "front": 1,
        "back": 1
      }
    }
  ]
}
```

- `dpi` For given width and height, this is the resulting DPI on the actual product.

- `fill_mode` Possible values: "fit" or "cover". Indicates in what mode mockups will be generated.

- `can_rotate` Posters, for example, allow rotation. If you pass the image in horizontal positions.

- `placements.front` Printfile id.

We can see that the full print file size is 1800×2400 for back and front prints for chosen variants.

When we know the size of the print file, we need to calculate the positions. Position values are relative here, image
size does not have to match the width and height of positions. When mockup is generated we will fit the position area
inside the print area or will cover it, depending on the print file `fill_mode` value.

Positions given below would result in a square image centered vertically within the print area.

```json
{
  "area_width": 1800,
  "area_height": 2400,
  "width": 1800,
  "height": 1800,
  "top": 300,
  "left": 0
}
```

- `area_width` Relative width of the print area.

- `area_height` Relative height of the print area.

- `width` Relative width of your image.

- `height` Relative height of your image.

- `top` Relative image top offset within the area.

- `left` Relative image left offset within the area.

<div class="alert alert-info">
<strong>Important</strong><br>
For posters, canvas, and other products which print files allow rotation (<code>can_rotate</code> value in <a href="#operation/getPrintfiles">print file response</a>) you
can flip width and height to create a product mockup that is horizontal or vertical.
</div>

Once we have calculated the positions, we can perform the actual mockup generation using
the [mockup generator endpoint](#operation/createGeneratorTask):
`POST` to `https://api.printful.com/mockup-generator/create-task/71` with body parameters:

```json
{
  "variant_ids": [
    4012,
    4013,
    4014,
    4017,
    4018,
    4019
  ],
  "format": "jpg",
  "files": [
    {
      "placement": "front",
      "image_url": "http://your-site/path-to-front-printfile.jpg",
      "position": {
        "area_width": 1800,
        "area_height": 2400,
        "width": 1800,
        "height": 1800,
        "top": 300,
        "left": 0
      }
    },
    {
      "placement": "back",
      "image_url": "http://your-site/path-to-back-printfile.jpg",
      "position": {
        "area_width": 1800,
        "area_height": 2400,
        "width": 1800,
        "height": 1800,
        "top": 300,
        "left": 0
      }
    }
  ]
}
```

In response, you will receive the task key and current task status:

```json
{
  "task_key": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "status": "pending"
}
```

After an interval of a few seconds, you can try to check for the result by calling a `GET` request
on `https://api.printful.com/mockup-generator/task?task_key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
If the task is completed, the response will be like this:

```json
{
  "task_key": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "status": "completed",
  "mockups": [
    {
      "variant_ids": [
        4011,
        4012,
        4013
      ],
      "placement": "front",
      "mockup_url": "https://url-to/front-mockup.png"
    },
    {
      "variant_ids": [
        4011,
        4012,
        4013
      ],
      "placement": "back",
      "mockup_url": "https://url-to/back-mockup.png"
    },
    {
      "variant_ids": [
        4016,
        4017,
        4018
      ],
      "placement": "front",
      "mockup_url": "https://url-to/front-mockup.png"
    },
    {
      "variant_ids": [
        4016,
        4017,
        4018
      ],
      "placement": "back",
      "mockup_url": "https://url-to/back-mockup.png"
    }
  ]
}
```

At this point, you just have to download the mockup URLs and store them on your server and you're good to go!

### Layout templates

If you wish to build your mockup generator UI, this is the place to start. Using
the [layout template endpoint](#operation/getPrintfiles)
you can get template images and positions necessary to create a tool where your users can position their files on.

If you want to create a mug generator, for example, you call the endpoint `/mockup-generator/templates/19` with mug
product ID. By looking at the variant mapping field, we can determine that for variant `1320` 11oz mug we have to use
the template with ID `919`. This is what template structure looks like:

```json
{
  "template_id": 919,
  "image_url": "https://www.printful.com/files/generator/40/11oz_template.png",
  "background_url": null,
  "background_color": null,
  "printfile_id": 43,
  "template_width": 560,
  "template_height": 295,
  "print_area_width": 520,
  "print_area_height": 202,
  "print_area_top": 18,
  "print_area_left": 20,
  "is_template_on_front": true
}
```

- `printfile_id` We can retrieve the actual printfile size from the printfiles endpoint.

- `template_width` This is the main container width, pixels.

- `template_height` Main container height.

- `print_area_width` Inner area where positioning happens.

- `print_area_height` Inner area.

- `print_area_top` Offset from the main container.

- `print_area_left` Offset from the main container.

- `is_template_on_front` This indicates if we should show the user image below or above the template image.

Given this information, we can create a simple HTML markup:

```html

<div style="position: relative; width: 520px; height: 295px;">
    <div style="position: absolute; width: 520px; height: 202px; top:18px; left:20px; background:rgba(255,233,230,0.33)">
        <img alt="Printful logo" src="https://printful.com/static/images/layout/logo-printful.png"
             style="position: absolute; left: 43px; top: 77px; width: 140px; height: 63px;">
    </div>
    <div style="position: absolute; width: 560px; height: 295px; background:url(/files/generator/40/11oz_template.png) center center no-repeat"></div>
</div>
```

Which would look like this in the browser:

<div style="position: relative; width: 520px; height: 295px;">
    <div style="position: absolute; width: 520px; height: 202px; top:18px; left:20px; background:rgba(255,233,230,0.33)">
        <img src="https://printful.com/static/images/layout/logo-printful.png" alt="Printful logo"
             style="position: absolute; left: 43px; top: 77px; width: 140px; height: 63px;">
    </div>
    <div style="position: absolute; width: 560px; height: 295px; background:url(/files/generator/40/11oz_template.png) center center no-repeat"></div>
</div>

To generate mockups with positions above, we perform a `POST` request
to `https://api.printful.com/mockup-generator/create-task/19` with body parameters:

```json
{
  "variant_ids": [
    1320
  ],
  "format": "jpg",
  "files": [
    {
      "placement": "default",
      "image_url": "https://www.printful.test/static/images/layout/logo-printful.png",
      "position": {
        "area_width": 520,
        "area_height": 202,
        "width": 140,
        "height": 63,
        "top": 77,
        "left": 43
      }
    }
  ]
}
```

- `area_width` Value of print_area_width in the template.
- `area_height` Value of print_area_height in the template.
- `width` Image width.
- `height` Image height.
- `top` Image top offset in area.
- `left` Image left offset in area.


### Choosing mockup styles

To choose which mockup styles to generate, you have to specify `options` and `option_groups` parameters in the request. If these parameters are not present in the request, the system will generate the first available mockup. If you are not planning to utilize all available mockups, it is advised to limit the requested mockups. Not limiting requested mockups will cause bigger task processing times and overall resource waste.

To find available `options` and `option_groups` for a given product, you have to use `/mockup-generator/printfiles/{id}` endpoint and search for `options` and `option_groups` fields in the response. See examples below.

```
{
    "variant_ids": [4021],
    "format": "png",
    "option_groups": ["Flat"],
    "options": ["Front"],
    "files": [
        {
            "placement": "front",
            "image_url": "https://www.printful.com/static/images/layout/logo-printful.png"
        }
    ]
}
```

### Using lifelike effect

Lifelike is a feature that simulates how dark designs will look over dark colour products and is only used in mockup generation. For that, an extra file with special effect is created for each placement.

```
{
    "variant_ids": [4018],
    "format": "png",
    "product_options": {
      "lifelike": true
    },
    "files": [
        {
            "placement": "front",
            "image_url": "https://www.printful.com/static/images/layout/logo-printful.png"
        }
    ]
}
```

| Mockup without lifelike                        | Mockup with lifelike                        |
|------------------------------------------------|---------------------------------------------|
| ![Image](images/lifelike/without_lifelike.png) | ![Image](images/lifelike/with_lifelike.png) |


## Endpoints

### POST /mockup-generator/create-task/{id}
**Summary:** Create a mockup generation task

Creates an asynchronous mockup generation task.
Generation result can be retrieved using mockup generation task retrieval endpoint.<br>
**Rate limiting**: Up to 10 requests per 60 seconds for established stores;
2 requests per 60 seconds for new stores. Currently available rate is returned in response headers.
A 60 seconds lockout is applied if request count is exceeded. We also limit the number of files that may
be generated to 20,000 files per account in a 24-hour period.


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "createGeneratorTask",
  "security": [
    {
      "OAuth": []
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
      "description": "Product ID."
    },
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
          "title": "CreateGenerationTask",
          "description": "Mockup generation data.",
          "properties": {
            "variant_ids": {
              "type": "array",
              "description": "List of variant ids you want to generate.",
              "items": {
                "type": "integer"
              },
              "example": [
                4012,
                4013,
                4014,
                4017,
                4018,
                4019
              ]
            },
            "format": {
              "type": "string",
              "description": "Generated file format. PNG will have a transparent background, JPG will have a smaller file size.",
              "enum": [
                "jpg",
                "png"
              ],
              "example": "jpg"
            },
            "width": {
              "type": "integer",
              "description": "Width of the resulting mockup images (min 50, max 2000, default is 1000)"
            },
            "product_options": {
              "description": "Key-value list of product options (embroidery thread, stitch colors). Product options can be found in Catalog API endpoint. [See examples](#tag/Common/Options)",
              "type": "object",
              "additionalProperties": true
            },
            "option_groups": {
              "type": "array",
              "additionalProperties": true,
              "description": "List of option group names you want to generate. Product's option groups can be found in printfile API request.",
              "items": {
                "type": "string"
              }
            },
            "options": {
              "type": "array",
              "description": "List of option names you want to generate. Product's options can be found in printfile API request.",
              "items": {
                "type": "string"
              }
            },
            "files": {
              "type": "array",
              "items": {
                "type": "object",
                "title": "GenerationTaskFile",
                "description": "Placement and file mapping to be generated.",
                "properties": {
                  "placement": {
                    "type": "string",
                    "description": "Placement identifier (front, back, etc.).",
                    "example": "front"
                  },
                  "image_url": {
                    "type": "string",
                    "description": "Public URL where your file is stored.",
                    "example": "\u200bhttp://your-site/path-to-front-printfile.jpg"
                  },
                  "position": {
                    "type": "object",
                    "title": "GenerationTaskFilePosition",
                    "description": "Position",
                    "properties": {
                      "area_width": {
                        "type": "integer",
                        "description": "Positioning area width on print area in pixels",
                        "example": 1800,
                        "nullable": true
                      },
                      "area_height": {
                        "type": "integer",
                        "description": "Positioning area height on print area in pixels",
                        "example": 2400,
                        "nullable": true
                      },
                      "width": {
                        "type": "integer",
                        "description": "Width of the image in given area in pixels",
                        "example": 1800,
                        "readOnly": false
                      },
                      "height": {
                        "type": "integer",
                        "description": "Height of the image in given area in pixels",
                        "example": 1800,
                        "readOnly": false
                      },
                      "top": {
                        "type": "integer",
                        "description": "Image top offset in given area in pixels",
                        "example": 300
                      },
                      "left": {
                        "type": "integer",
                        "description": "Image left offset in given area in pixels",
                        "example": 0
                      }
                    }
                  },
                  "options": {
                    "type": "array",
                    "description": "Array of additional options for this file [See examples](#tag/Common/Options)",
                    "items": {
                      "type": "object",
                      "title": "FileOption",
                      "description": "File option",
                      "required": [
                        "id",
                        "value"
                      ],
                      "properties": {
                        "id": {
                          "type": "string",
                          "description": "Option id",
                          "example": "template_type"
                        },
                        "value": {
                          "type": "string",
                          "description": "Option value",
                          "example": "native"
                        }
                      }
                    }
                  }
                }
              }
            },
            "product_template_id": {
              "type": "integer",
              "description": "Product template ID. Use instead of files parameter.",
              "example": 123
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
                "type": "object",
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "GenerationTask",
                    "description": "GenerationTask",
                    "properties": {
                      "task_key": {
                        "type": "string",
                        "description": "Task identifier you will use to retrieve generated mockups.",
                        "example": "123456"
                      },
                      "status": {
                        "type": "string",
                        "description": "Status of the generation task.",
                        "enum": [
                          "pending",
                          "completed",
                          "failed"
                        ],
                        "example": "completed"
                      },
                      "error": {
                        "type": "string",
                        "description": "If task has failed, reason will be provided here."
                      },
                      "mockups": {
                        "type": "array",
                        "description": "If task is completed, list of mockups will be provided here.",
                        "items": {
                          "type": "object",
                          "title": "GenerationTaskMockup",
                          "description": "Generation task mockup.",
                          "properties": {
                            "placement": {
                              "type": "string",
                              "description": "Placement identifier.",
                              "example": "front"
                            },
                            "display_name": {
                              "type": "string",
                              "description": "This is a name that can be displayed to end customers.",
                              "example": "Front Print"
                            },
                            "variant_ids": {
                              "type": "array",
                              "description": "List of variant ids this mockup is used for. One mockup can be used for multiple variants.",
                              "items": {
                                "type": "integer"
                              },
                              "example": [
                                4011,
                                4012,
                                4013
                              ]
                            },
                            "extra": {
                              "type": "array",
                              "description": "Optional extra mockups.",
                              "items": {
                                "type": "object",
                                "title": "GenerationTaskExtraMockup",
                                "description": "Generation task extra mockup",
                                "properties": {
                                  "title": {
                                    "type": "string",
                                    "description": "Display name of the extra mockup."
                                  },
                                  "url": {
                                    "type": "string",
                                    "description": "Temporary URL of the mockup.",
                                    "example": "\u200bhttps://url-to/extra-mockup.png"
                                  },
                                  "option": {
                                    "type": "string",
                                    "description": "Style option name"
                                  },
                                  "option_group": {
                                    "type": "string",
                                    "description": "Style option group name"
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "printfiles": {
                        "type": "array",
                        "description": "If task is completed, list of printfiles will be provided here.",
                        "items": {
                          "type": "object",
                          "title": "GenerationTaskTemplateFile",
                          "description": "Generated File placements and URLs.",
                          "properties": {
                            "variant_ids": {
                              "type": "array",
                              "description": "List of variant IDs associated with printfiles.",
                              "items": {
                                "type": "integer"
                              },
                              "example": [
                                4012,
                                4013,
                                4014,
                                4017,
                                4018,
                                4019
                              ]
                            },
                            "placement": {
                              "type": "string",
                              "description": "Placement identifier (front, back, etc.).",
                              "example": "front"
                            },
                            "url": {
                              "type": "string",
                              "description": "Public URL where your file is stored.",
                              "example": "\u200bhttps://url-to/printfile.png"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/mockup-generator/create-task/71' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n                \"variant_ids\" : [4012, 4013, 4014, 4017, 4018, 4019],\n                \"format\": \"jpg\",\n                \"files\" : [\n                    {\n                        \"placement\": \"front\",\n                        \"image_url\": \"http://your-site/path-to-front-printfile.jpg\",\n                        \"position\": {\n                            \"area_width\": 1800,\n                            \"area_height\": 2400,\n                            \"width\": 1800,\n                            \"height\": 1800,\n                            \"top\": 300,\n                            \"left\": 0\n                        }\n                    },\n                    {\n                        \"placement\": \"back\",\n                        \"image_url\": \"http://your-site/path-to-back-printfile.jpg\",\n                        \"position\": {\n                            \"area_width\": 1800,\n                            \"area_height\": 2400,\n                            \"width\": 1800,\n                            \"height\": 1800,\n                            \"top\": 300,\n                            \"left\": 0\n                        }\n                    }\n                ]\n            }'"
    }
  ]
}
```
</details>

---

### GET /mockup-generator/printfiles/{id}
**Summary:** Retrieve product variant printfiles

List of printfiles available for products variants. Printfile indicates what file resolution should be used to create a mockup or submit an order.

<div class="alert alert-info">
This endpoint uses DTG as a default printing technique for products
with more than one technique available. For products with DTG and more
techniques available please specify the correct technique in query by using
the `technique` parameter. For more information read the <a href="#tag/Examples/Mockup-Generator-API-examples">examples</a>.
</div>


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getPrintfiles",
  "security": [
    {
      "OAuth": []
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
      "description": "Product ID."
    },
    {
      "in": "query",
      "name": "orientation",
      "schema": {
        "type": "string",
        "description": "Optional orientation for wall art product printfiles. Allowed values: horizontal, vertical",
        "enum": [
          "horizontal",
          "vertical"
        ]
      }
    },
    {
      "in": "query",
      "name": "technique",
      "schema": {
        "type": "string",
        "description": "Optional technique for product. This can be used in cases where product supports multiple techniques like DTG and embroidery"
      }
    },
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
                "type": "object",
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "PrintfileInfo",
                    "description": "Printfile info",
                    "properties": {
                      "product_id": {
                        "type": "integer",
                        "description": "Requested product id.",
                        "example": 71
                      },
                      "available_placements": {
                        "type": "object",
                        "description": "List of available placements. Key is placement identifier, value is display name. (e.g. {embroidery_front: Front, ..}).",
                        "example": {
                          "front": "Front print",
                          "back": "Back print",
                          "label_outside": "Outside label"
                        }
                      },
                      "printfiles": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "Printfile",
                          "description": "Printfile",
                          "properties": {
                            "printfile_id": {
                              "type": "integer",
                              "description": "Unique printfile identifier.",
                              "example": 1
                            },
                            "width": {
                              "type": "integer",
                              "description": "Width in pixels.",
                              "example": 1800
                            },
                            "height": {
                              "type": "integer",
                              "description": "Height in pixels.",
                              "example": 2400
                            },
                            "dpi": {
                              "type": "integer",
                              "description": "Resulting DPI for given width and height.",
                              "example": 150
                            },
                            "fill_mode": {
                              "type": "string",
                              "description": "Indicates if printfile will be used in cover or fit mode. Cover mode can produce cropping if side ratio does not match printfile.",
                              "enum": [
                                "cover",
                                "fit"
                              ],
                              "example": "fit"
                            },
                            "can_rotate": {
                              "type": "boolean",
                              "description": "Indicates if printfile can be rotated horizontally (e.g. for posters).",
                              "example": false
                            }
                          }
                        }
                      },
                      "variant_printfiles": {
                        "type": "array",
                        "items": {
                          "type": "object",
                          "title": "VariantPrintfile",
                          "properties": {
                            "variant_id": {
                              "type": "integer",
                              "example": 4012
                            },
                            "placements": {
                              "type": "object",
                              "description": "A key-value object mapping placement identifiers to printfile IDs.",
                              "example": {
                                "front": 1,
                                "back": 1,
                                "label_outside": 1
                              }
                            }
                          }
                        }
                      },
                      "option_groups": {
                        "type": "array",
                        "items": {
                          "type": "string"
                        }
                      },
                      "options": {
                        "type": "array",
                        "items": {
                          "type": "string"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/mockup-generator/printfiles/1' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### GET /mockup-generator/task
**Summary:** Mockup generation task result

Returns asynchronous mockup generation task result. If generation task is completed, it will contain a list of generated mockups.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getTask",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
    {
      "in": "query",
      "name": "task_key",
      "schema": {
        "type": "string"
      },
      "required": true,
      "description": "Task key retrieved when creating the generation task."
    },
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
                "type": "object",
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "GenerationTask",
                    "description": "GenerationTask",
                    "properties": {
                      "task_key": {
                        "type": "string",
                        "description": "Task identifier you will use to retrieve generated mockups.",
                        "example": "123456"
                      },
                      "status": {
                        "type": "string",
                        "description": "Status of the generation task.",
                        "enum": [
                          "pending",
                          "completed",
                          "failed"
                        ],
                        "example": "completed"
                      },
                      "error": {
                        "type": "string",
                        "description": "If task has failed, reason will be provided here."
                      },
                      "mockups": {
                        "type": "array",
                        "description": "If task is completed, list of mockups will be provided here.",
                        "items": {
                          "type": "object",
                          "title": "GenerationTaskMockup",
                          "description": "Generation task mockup.",
                          "properties": {
                            "placement": {
                              "type": "string",
                              "description": "Placement identifier.",
                              "example": "front"
                            },
                            "display_name": {
                              "type": "string",
                              "description": "This is a name that can be displayed to end customers.",
                              "example": "Front Print"
                            },
                            "variant_ids": {
                              "type": "array",
                              "description": "List of variant ids this mockup is used for. One mockup can be used for multiple variants.",
                              "items": {
                                "type": "integer"
                              },
                              "example": [
                                4011,
                                4012,
                                4013
                              ]
                            },
                            "extra": {
                              "type": "array",
                              "description": "Optional extra mockups.",
                              "items": {
                                "type": "object",
                                "title": "GenerationTaskExtraMockup",
                                "description": "Generation task extra mockup",
                                "properties": {
                                  "title": {
                                    "type": "string",
                                    "description": "Display name of the extra mockup."
                                  },
                                  "url": {
                                    "type": "string",
                                    "description": "Temporary URL of the mockup.",
                                    "example": "\u200bhttps://url-to/extra-mockup.png"
                                  },
                                  "option": {
                                    "type": "string",
                                    "description": "Style option name"
                                  },
                                  "option_group": {
                                    "type": "string",
                                    "description": "Style option group name"
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "printfiles": {
                        "type": "array",
                        "description": "If task is completed, list of printfiles will be provided here.",
                        "items": {
                          "type": "object",
                          "title": "GenerationTaskTemplateFile",
                          "description": "Generated File placements and URLs.",
                          "properties": {
                            "variant_ids": {
                              "type": "array",
                              "description": "List of variant IDs associated with printfiles.",
                              "items": {
                                "type": "integer"
                              },
                              "example": [
                                4012,
                                4013,
                                4014,
                                4017,
                                4018,
                                4019
                              ]
                            },
                            "placement": {
                              "type": "string",
                              "description": "Placement identifier (front, back, etc.).",
                              "example": "front"
                            },
                            "url": {
                              "type": "string",
                              "description": "Public URL where your file is stored.",
                              "example": "\u200bhttps://url-to/printfile.png"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/mockup-generator/task?task_key=3123' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### GET /mockup-generator/templates/{id}
**Summary:** Layout templates

Retrieve list of templates that can be used for client-side positioning.

<div class="alert alert-info">
This endpoint uses DTG as a default printing technique for product layouts
with more than one technique available. For products with DTG and more
techniques available please specify the correct technique in query by using
the `technique` parameter. For more information read the <a href="#tag/Examples/Mockup-Generator-API-examples">examples</a>.
</div>


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getTemplates",
  "security": [
    {
      "OAuth": []
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
      "description": "Product ID."
    },
    {
      "in": "query",
      "name": "orientation",
      "schema": {
        "type": "string",
        "description": "Optional orientation for wall art product printfiles. Allowed values: horizontal, vertical",
        "enum": [
          "horizontal",
          "vertical"
        ]
      }
    },
    {
      "in": "query",
      "name": "technique",
      "schema": {
        "type": "string",
        "description": "Optional technique for product. This can be used in cases where product supports multiple techniques like DTG and embroidery"
      }
    },
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
                "type": "object",
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "ProductTemplate",
                    "description": "Product Template",
                    "properties": {
                      "version": {
                        "type": "integer",
                        "description": "Resource version. If this changes, resources (positions, images, etc.) should be re-cached.",
                        "example": 1
                      },
                      "min_dpi": {
                        "type": "integer",
                        "description": "Recommended minimum DPI for given product.",
                        "example": 300
                      },
                      "variant_mapping": {
                        "type": "array",
                        "description": "List of product variants mapped to templates. From this information you can determine which template should be used for a variant.",
                        "items": {
                          "type": "object",
                          "title": "TemplateVariantMapping",
                          "description": "Template variant mapping",
                          "properties": {
                            "variant_id": {
                              "type": "integer",
                              "description": "Product variant ID.",
                              "example": 1
                            },
                            "templates": {
                              "type": "array",
                              "description": "Array of Template Variant Mapping items",
                              "items": {
                                "type": "object",
                                "title": "TemplateVariantMappingItem",
                                "description": "Template variant mapping item",
                                "properties": {
                                  "placement": {
                                    "type": "string",
                                    "description": "Placement ID.",
                                    "example": "front"
                                  },
                                  "template_id": {
                                    "type": "integer",
                                    "description": "Corresponding template id which should be used for this variant and placement combination.",
                                    "example": 1
                                  }
                                }
                              }
                            }
                          }
                        }
                      },
                      "templates": {
                        "type": "array",
                        "description": "List of templates. Use variant_mapping to determine which template corresponds to which product variant.",
                        "items": {
                          "type": "object",
                          "title": "Template",
                          "description": "Template variant mapping",
                          "properties": {
                            "template_id": {
                              "type": "integer",
                              "description": "Template ID.",
                              "example": 919
                            },
                            "image_url": {
                              "type": "string",
                              "description": "Main template image URL.",
                              "example": "https://www.printful.com/files/generator/40/11oz_template.png"
                            },
                            "background_url": {
                              "type": "string",
                              "description": "Background image URL (optional).",
                              "nullable": true,
                              "example": null
                            },
                            "background_color": {
                              "type": "integer",
                              "description": "HEX color code that should be used as a background color.",
                              "nullable": true,
                              "example": null
                            },
                            "printfile_id": {
                              "type": "integer",
                              "description": "Printfile ID that should be generated for this template. See [printfile API endpoint](#operation/getPrintfiles) for list of Printfiles.",
                              "example": 43
                            },
                            "template_width": {
                              "type": "integer",
                              "description": "Width of the whole template in pixels.",
                              "example": 560
                            },
                            "template_height": {
                              "type": "integer",
                              "description": "Height of the whole template in pixels.",
                              "example": 295
                            },
                            "print_area_width": {
                              "type": "integer",
                              "description": "Print area width (image is positioned in this area).",
                              "example": 520
                            },
                            "print_area_height": {
                              "type": "integer",
                              "description": "Print area height (image is positioned in this area).",
                              "example": 202
                            },
                            "print_area_top": {
                              "type": "integer",
                              "description": "Print area top offset (offset in template).",
                              "example": 18
                            },
                            "print_area_left": {
                              "type": "integer",
                              "description": "Print area left offset (offset in template).",
                              "example": 20
                            },
                            "is_template_on_front": {
                              "type": "boolean",
                              "description": "Should the main template image (image_url) be used as an overlay or as a background.",
                              "example": true
                            },
                            "orientation": {
                              "type": "string",
                              "description": "Wall art product orientation. Possible values: horizontal, vertical, any",
                              "enum": [
                                "horizontal",
                                "vertical",
                                "any"
                              ],
                              "example": "any"
                            }
                          }
                        }
                      },
                      "conflicting_placements": {
                        "type": "array",
                        "description": "List of conflicting placements. Used to determine which placements can be used together.",
                        "items": {
                          "type": "object",
                          "title": "TemplatePlacementConflict",
                          "description": "Template placement conflict",
                          "properties": {
                            "placement": {
                              "type": "string",
                              "description": "Placement ID",
                              "example": "label_outside"
                            },
                            "conflicts": {
                              "type": "array",
                              "description": "List Placement IDs that are conflicting with given placement",
                              "items": {
                                "type": "string"
                              },
                              "example": [
                                "back",
                                "label_inside"
                              ]
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request GET 'https://api.printful.com/mockup-generator/templates/71' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---


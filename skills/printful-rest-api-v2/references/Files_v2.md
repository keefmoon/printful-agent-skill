# Files v2

To avoid the need to upload every file again when the same item is ordered,
your print files are stored in the File Library and can be reused.

You can use this API to directly add files to the library, and later use
File IDs when creating orders. However, the more convenient way is to specify
the files by URL at the same time the order is created.

<div class="alert alert-info">
Most probably you will never need to use this API - just specify the file URL
when creating orders and the files will be added automatically.
</div>

File processing can be very time-consuming, so they are processed
asynchronously. After you add a file, it is saved with the status
`waiting` and downloaded and processed later. Afterward, the status
is changed to `ok` if the file was loaded successfully and was a valid
image file or `failed` if the process did not succeed. Some file
metadata fields like dimensions and resolution are only filled in
after the file has been processed.

If an order with a file has been confirmed before the file was processed,
and the file turns out to be invalid, then the order is reverted to a failed
state and needs to be corrected and confirmed again.

If you try to add a file that has an identical URL to an already
existing file, then no new file is created, and the system returns
the old one without refreshing its contents.

<div class="alert alert-info">
<strong>Remember</strong><br>
If you have changed the original, make sure that the URL is changed as well
for future orders, otherwise the old version will be reused.
</div>

You can add a “last modified” timestamp to the end of the URL to ensure
that the URL is different for changed files.

Files that are added through the API can be set not to show up in the
File library on the web,
just set the visible attribute to false when creating them.

**Caution: API endpoint "Get list of files" (/files) is removed and can no longer be used. Calling this endpoint will return a HTTP 410 (Gone) response.**


## Endpoints

### POST /v2/files
**Summary:** Add a new file

Adds a new File to the library by providing URL of the file.

If a file with identical URL already exists, then the original file is returned. If a file does not exist, a new file is created.

[See examples](#tag/Examples/File-Library-API-examples/Add-a-new-file)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "addFile",
  "security": [
    {
      "OAuth": [
        "file-library"
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
          "type": "object",
          "title": "File",
          "description": "Information about the added File",
          "required": [
            "url"
          ],
          "properties": {
            "role": {
              "type": "string",
              "enum": [
                "printfile",
                "label",
                "preview"
              ],
              "default": "printfile",
              "description": "Role of the file",
              "example": "printfile"
            },
            "url": {
              "type": "string",
              "description": "Source URL where the file is to be downloaded from. The use of .ai, .psd, and .tiff files has been deprecated, if your application uses these file types or accepts these types from users you will need to add validation.",
              "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
            },
            "filename": {
              "type": "string",
              "description": "If the filename is not provided, and something looking like a filename is present in the URL (e.g. \"something.jpg\"), it will be used.\nOtherwise, it will default to `{file_id}.{file_extension}`, with file extension determined based on the media type of the file.\n",
              "example": "shirt1.png"
            },
            "visible": {
              "type": "boolean",
              "description": "Show file in the Printfile Library",
              "default": true,
              "example": true
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
                "type": "object",
                "title": "File",
                "description": "Information about the File",
                "required": [
                  "id",
                  "url",
                  "hash",
                  "filename",
                  "mime_type",
                  "size",
                  "width",
                  "height",
                  "dpi",
                  "status",
                  "created",
                  "thumbnail_url",
                  "preview_url",
                  "visible",
                  "is_temporary",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "type": "integer",
                    "description": "File ID",
                    "example": 123,
                    "readOnly": true
                  },
                  "url": {
                    "type": "string",
                    "description": "Source URL where the file was downloaded from.",
                    "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                  },
                  "hash": {
                    "type": "string",
                    "description": "MD5 checksum of the file",
                    "example": "ea44330b887dfec278dbc4626a759547",
                    "nullable": true,
                    "readOnly": true
                  },
                  "filename": {
                    "type": "string",
                    "description": "File name",
                    "nullable": true,
                    "example": "shirt1.png"
                  },
                  "mime_type": {
                    "type": "string",
                    "description": "MIME type of the file",
                    "example": "image/png",
                    "nullable": true,
                    "readOnly": true
                  },
                  "size": {
                    "type": "integer",
                    "description": "Size in bytes",
                    "example": 45582633,
                    "readOnly": true
                  },
                  "width": {
                    "type": "integer",
                    "description": "Width in pixels",
                    "example": 1000,
                    "nullable": true,
                    "readOnly": true
                  },
                  "height": {
                    "type": "integer",
                    "description": "Height in pixels",
                    "example": 1000,
                    "nullable": true,
                    "readOnly": true
                  },
                  "dpi": {
                    "type": "integer",
                    "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                    "example": 300,
                    "nullable": true,
                    "readOnly": true
                  },
                  "status": {
                    "type": "string",
                    "example": "ok",
                    "enum": [
                      "waiting",
                      "ok",
                      "failed",
                      "deleted"
                    ],
                    "description": "File processing status",
                    "x-enumDescriptions": {
                      "waiting": "File is waiting to be processed",
                      "ok": "File processed correctly and ready to use",
                      "failed": "File failed to process",
                      "deleted": "File was deleted"
                    },
                    "readOnly": true
                  },
                  "created": {
                    "type": "string",
                    "description": "File creation date",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "thumbnail_url": {
                    "type": "string",
                    "description": "Small thumbnail URL",
                    "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                    "nullable": true,
                    "readOnly": true
                  },
                  "preview_url": {
                    "type": "string",
                    "description": "Medium preview image URL",
                    "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                    "nullable": true,
                    "readOnly": true
                  },
                  "visible": {
                    "type": "boolean",
                    "description": "Whether the file is shown in the Printfile Library.",
                    "example": true
                  },
                  "is_temporary": {
                    "type": "boolean",
                    "readOnly": true,
                    "description": "Whether it is a temporary printfile.",
                    "example": false
                  },
                  "_links": {
                    "type": "object",
                    "readOnly": true,
                    "description": "HATEOAS links",
                    "required": [
                      "self"
                    ],
                    "properties": {
                      "self": {
                        "description": "Link to same resource",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/files/123"
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

### GET /v2/files/{id}
**Summary:** Retrieve a single file from the file library

Get basic information about the file stored in the file library

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getFileById",
  "security": [
    {
      "OAuth": [
        "file-library/read"
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
      "description": "File ID."
    }
  ],
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
                "type": "object",
                "title": "File",
                "description": "Information about the File",
                "required": [
                  "id",
                  "url",
                  "hash",
                  "filename",
                  "mime_type",
                  "size",
                  "width",
                  "height",
                  "dpi",
                  "status",
                  "created",
                  "thumbnail_url",
                  "preview_url",
                  "visible",
                  "is_temporary",
                  "_links"
                ],
                "properties": {
                  "id": {
                    "type": "integer",
                    "description": "File ID",
                    "example": 123,
                    "readOnly": true
                  },
                  "url": {
                    "type": "string",
                    "description": "Source URL where the file was downloaded from.",
                    "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
                  },
                  "hash": {
                    "type": "string",
                    "description": "MD5 checksum of the file",
                    "example": "ea44330b887dfec278dbc4626a759547",
                    "nullable": true,
                    "readOnly": true
                  },
                  "filename": {
                    "type": "string",
                    "description": "File name",
                    "nullable": true,
                    "example": "shirt1.png"
                  },
                  "mime_type": {
                    "type": "string",
                    "description": "MIME type of the file",
                    "example": "image/png",
                    "nullable": true,
                    "readOnly": true
                  },
                  "size": {
                    "type": "integer",
                    "description": "Size in bytes",
                    "example": 45582633,
                    "readOnly": true
                  },
                  "width": {
                    "type": "integer",
                    "description": "Width in pixels",
                    "example": 1000,
                    "nullable": true,
                    "readOnly": true
                  },
                  "height": {
                    "type": "integer",
                    "description": "Height in pixels",
                    "example": 1000,
                    "nullable": true,
                    "readOnly": true
                  },
                  "dpi": {
                    "type": "integer",
                    "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                    "example": 300,
                    "nullable": true,
                    "readOnly": true
                  },
                  "status": {
                    "type": "string",
                    "example": "ok",
                    "enum": [
                      "waiting",
                      "ok",
                      "failed",
                      "deleted"
                    ],
                    "description": "File processing status",
                    "x-enumDescriptions": {
                      "waiting": "File is waiting to be processed",
                      "ok": "File processed correctly and ready to use",
                      "failed": "File failed to process",
                      "deleted": "File was deleted"
                    },
                    "readOnly": true
                  },
                  "created": {
                    "type": "string",
                    "description": "File creation date",
                    "format": "date-time",
                    "example": "2023-04-05T06:07:08Z"
                  },
                  "thumbnail_url": {
                    "type": "string",
                    "description": "Small thumbnail URL",
                    "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                    "nullable": true,
                    "readOnly": true
                  },
                  "preview_url": {
                    "type": "string",
                    "description": "Medium preview image URL",
                    "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                    "nullable": true,
                    "readOnly": true
                  },
                  "visible": {
                    "type": "boolean",
                    "description": "Whether the file is shown in the Printfile Library.",
                    "example": true
                  },
                  "is_temporary": {
                    "type": "boolean",
                    "readOnly": true,
                    "description": "Whether it is a temporary printfile.",
                    "example": false
                  },
                  "_links": {
                    "type": "object",
                    "readOnly": true,
                    "description": "HATEOAS links",
                    "required": [
                      "self"
                    ],
                    "properties": {
                      "self": {
                        "description": "Link to same resource",
                        "title": "HateoasLink",
                        "example": {
                          "href": "https://api.printful.com/v2/files/123"
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


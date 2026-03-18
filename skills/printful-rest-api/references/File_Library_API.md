# File Library API

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

### POST /files
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
        "file_library"
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
          "title": "File",
          "description": "Information about the File",
          "required": [
            "url"
          ],
          "properties": {
            "type": {
              "type": "string",
              "description": "Role of the file",
              "example": "default"
            },
            "id": {
              "type": "integer",
              "description": "File ID",
              "example": 10,
              "readOnly": true
            },
            "url": {
              "type": "string",
              "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
              "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
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
            },
            "hash": {
              "type": "string",
              "description": "MD5 checksum of the file",
              "example": "ea44330b887dfec278dbc4626a759547",
              "readOnly": true
            },
            "filename": {
              "type": "string",
              "description": "File name",
              "example": "shirt1.png"
            },
            "mime_type": {
              "type": "string",
              "description": "MIME type of the file",
              "example": "image/png",
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
              "readOnly": true
            },
            "height": {
              "type": "integer",
              "description": "Height in pixels",
              "example": 1000,
              "readOnly": true
            },
            "dpi": {
              "type": "integer",
              "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
              "example": 300,
              "readOnly": true
            },
            "status": {
              "type": "string",
              "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
              "example": "ok",
              "readOnly": true
            },
            "created": {
              "type": "integer",
              "description": "File creation timestamp",
              "example": 1590051937,
              "readOnly": true
            },
            "thumbnail_url": {
              "type": "string",
              "description": "Small thumbnail URL",
              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
              "readOnly": true
            },
            "preview_url": {
              "type": "string",
              "description": "Medium preview image URL",
              "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
              "readOnly": true
            },
            "visible": {
              "type": "boolean",
              "description": "Show file in the Printfile Library (default true)",
              "example": true
            },
            "is_temporary": {
              "type": "boolean",
              "readOnly": true,
              "description": "Whether it is a temporary printfile.",
              "example": false
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
                    "title": "File",
                    "description": "Information about the File",
                    "required": [
                      "url"
                    ],
                    "properties": {
                      "type": {
                        "type": "string",
                        "description": "Role of the file",
                        "example": "default"
                      },
                      "id": {
                        "type": "integer",
                        "description": "File ID",
                        "example": 10,
                        "readOnly": true
                      },
                      "url": {
                        "type": "string",
                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
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
                      },
                      "hash": {
                        "type": "string",
                        "description": "MD5 checksum of the file",
                        "example": "ea44330b887dfec278dbc4626a759547",
                        "readOnly": true
                      },
                      "filename": {
                        "type": "string",
                        "description": "File name",
                        "example": "shirt1.png"
                      },
                      "mime_type": {
                        "type": "string",
                        "description": "MIME type of the file",
                        "example": "image/png",
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
                        "readOnly": true
                      },
                      "height": {
                        "type": "integer",
                        "description": "Height in pixels",
                        "example": 1000,
                        "readOnly": true
                      },
                      "dpi": {
                        "type": "integer",
                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                        "example": 300,
                        "readOnly": true
                      },
                      "status": {
                        "type": "string",
                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                        "example": "ok",
                        "readOnly": true
                      },
                      "created": {
                        "type": "integer",
                        "description": "File creation timestamp",
                        "example": 1590051937,
                        "readOnly": true
                      },
                      "thumbnail_url": {
                        "type": "string",
                        "description": "Small thumbnail URL",
                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                        "readOnly": true
                      },
                      "preview_url": {
                        "type": "string",
                        "description": "Medium preview image URL",
                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                        "readOnly": true
                      },
                      "visible": {
                        "type": "boolean",
                        "description": "Show file in the Printfile Library (default true)",
                        "example": true
                      },
                      "is_temporary": {
                        "type": "boolean",
                        "readOnly": true,
                        "description": "Whether it is a temporary printfile.",
                        "example": false
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/files' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"url\": \"http://www.example.com/files/tshirts/example.png\"\n}'"
    }
  ]
}
```
</details>

---

### GET /files/{id}
**Summary:** Get file

Returns information about the given file.

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getFile",
  "security": [
    {
      "OAuth": [
        "file_library/read"
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
                "properties": {
                  "result": {
                    "type": "object",
                    "title": "File",
                    "description": "Information about the File",
                    "required": [
                      "url"
                    ],
                    "properties": {
                      "type": {
                        "type": "string",
                        "description": "Role of the file",
                        "example": "default"
                      },
                      "id": {
                        "type": "integer",
                        "description": "File ID",
                        "example": 10,
                        "readOnly": true
                      },
                      "url": {
                        "type": "string",
                        "description": "Source URL where the file is downloaded from. The use of .ai .psd and .tiff files have been depreciated, if your application uses these file types or accepts these types from users you will need to add validation.",
                        "example": "\u200bhttps://www.example.com/files/tshirts/example.png"
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
                      },
                      "hash": {
                        "type": "string",
                        "description": "MD5 checksum of the file",
                        "example": "ea44330b887dfec278dbc4626a759547",
                        "readOnly": true
                      },
                      "filename": {
                        "type": "string",
                        "description": "File name",
                        "example": "shirt1.png"
                      },
                      "mime_type": {
                        "type": "string",
                        "description": "MIME type of the file",
                        "example": "image/png",
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
                        "readOnly": true
                      },
                      "height": {
                        "type": "integer",
                        "description": "Height in pixels",
                        "example": 1000,
                        "readOnly": true
                      },
                      "dpi": {
                        "type": "integer",
                        "description": "Resolution DPI.<br>**Note:** for vector files this may be indicated as only 72dpi, but it doesn't affect print quality since the vector files are resolution independent.",
                        "example": 300,
                        "readOnly": true
                      },
                      "status": {
                        "type": "string",
                        "description": "File processing status:<br>**ok** - file was processed successfuly<br>**waiting** - file is being processed<br>**failed** - file failed to be processed",
                        "example": "ok",
                        "readOnly": true
                      },
                      "created": {
                        "type": "integer",
                        "description": "File creation timestamp",
                        "example": 1590051937,
                        "readOnly": true
                      },
                      "thumbnail_url": {
                        "type": "string",
                        "description": "Small thumbnail URL",
                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                        "readOnly": true
                      },
                      "preview_url": {
                        "type": "string",
                        "description": "Medium preview image URL",
                        "example": "https://files.cdn.printful.com/files/ea4/ea44330b887dfec278dbc4626a759547_thumb.png",
                        "readOnly": true
                      },
                      "visible": {
                        "type": "boolean",
                        "description": "Show file in the Printfile Library (default true)",
                        "example": true
                      },
                      "is_temporary": {
                        "type": "boolean",
                        "readOnly": true,
                        "description": "Whether it is a temporary printfile.",
                        "example": false
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
      "source": "curl --location --request GET 'https://api.printful.com/files/{123}' \\\n--header 'Authorization: Bearer {oauth_token}'"
    }
  ]
}
```
</details>

---

### POST /files/thread-colors
**Summary:** Return available thread colors from provided image URL

Returns colors in hexadecimal format.

Returned thread colors are matched as closely as possible to provided image colors.

[See examples](#tag/Examples/File-Library-API-examples/Suggest-thread-colors)


#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "threadColors",
  "requestBody": {
    "description": "POST request body",
    "required": true,
    "content": {
      "application/json": {
        "schema": {
          "type": "object",
          "properties": {
            "file_url": {
              "type": "string",
              "description": "URL to file",
              "example": "https://example.com/image.jpg"
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
                  "thread_colors": {
                    "type": "array",
                    "items": {
                      "type": "string",
                      "example": "#FFFFFF",
                      "description": "Suggested available thread colors"
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
  },
  "x-codeSamples": [
    {
      "lang": "Curl",
      "source": "curl --location --request POST 'https://api.printful.com/files' \\\n--header 'Authorization: Bearer {oauth_token}' \\\n--data-raw '{\n    \"url\": \"http://www.example.com/files/tshirts/example.png\"\n}'"
    }
  ]
}
```
</details>

---


# Countries v2

To create an order, you have to use country and state codes to specify the recipient address. Both country code and state code are mandatory for orders to the USA, Canada and Australia. For other countries only the country code is needed to create an order.

Country codes are based on the ISO 3166-1 alpha-2 standard and are two letters long.

State codes are based on the ISO 3166-2 standard by omitting the country code part of the code and are used only for the USA, Canada, Japan and Australia.

All state/country codes that Printful accepts can be listed by this API.


## Endpoints

### GET /v2/countries
**Summary:** Retrieve a list of countries

Get information about all countries where Printful is available

#### Full Endpoint Schema (Resolved)
<details>
<summary>Click to view</summary>

```json
{
  "operationId": "getCountries",
  "security": [
    {
      "OAuth": []
    }
  ],
  "parameters": [
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
    },
    {
      "in": "query",
      "name": "limit",
      "schema": {
        "type": "integer",
        "minimum": 1,
        "maximum": 250,
        "default": 20
      },
      "required": false,
      "description": "The number of results to return per page."
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
                  "title": "Country",
                  "required": [
                    "code",
                    "name",
                    "states"
                  ],
                  "properties": {
                    "code": {
                      "type": "string",
                      "description": "Country code",
                      "example": "AU"
                    },
                    "name": {
                      "type": "string",
                      "description": "Country name",
                      "example": "Australia"
                    },
                    "states": {
                      "type": "array",
                      "description": "This array contains all states available for a country. If states are not required or not applicable for a given country this array will be empty.",
                      "items": {
                        "type": "object",
                        "title": "State",
                        "properties": {
                          "code": {
                            "type": "string",
                            "description": "State code",
                            "example": "ACT"
                          },
                          "name": {
                            "type": "string",
                            "description": "State name",
                            "example": "Australian Capital Territory"
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
              },
              "_links": {
                "type": "object",
                "description": "HATEOAS links",
                "required": [
                  "self"
                ],
                "properties": {
                  "self": {
                    "description": "Link to same resource",
                    "title": "HateoasLink",
                    "example": {
                      "href": "https://api.printful.com/v2/countries?limit=10&offset=20"
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
                      "href": "https://api.printful.com/v2/countries?limit=10&offset=30"
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
                      "href": "https://api.printful.com/v2/countries?limit=10&limit=10"
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
                      "href": "https://api.printful.com/v2/countries?limit=10"
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
                      "href": "https://api.printful.com/v2/countries?limit=10&offset=90"
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


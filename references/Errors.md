# Errors

# General errors

| <div style="width:60px; ">Code</div> | <div style="width:250px;">Name</div> | Explanation                                                                                                 |
|:------------------------------------:|:-------------------------------------|:------------------------------------------------------------------------------------------------------------|
|                 401                  | `UNAUTHORIZED`                       | Indicates that the endpoint requires authentication                                                         |
|                 403                  | `FORBBIDEN`                          | Unauthorized requests. The client does not have access rights to the resource                               |
|                 404                  | `NOT FOUND`                          | The server cannot find the requested resource                                                               |
|                 405                  | `METHOD NOT ALLOWED`                 | The request `HTTP` method is known by the server but has been disabled and cannot be used for that resource |
|                 419                  | `RATE LIMITED WARNING`               | Returned by the API when the client is being rate limited                                                   |
|                 429                  | `TOO MANY REQUESTS`                  | The user has sent too many requests in a given amount of time (_rate limiting_)                             |
|                 430                  | `INVALID JSON`                       | The user has sent an invalid `JSON`.                                                                        |
|                 431                  | `INVALID PARAMETER`                  | The user has sent a parameter with the wrong `type`                                                         |

# API Errors

## Products API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div> | Explanation                                                                                                                                                   |
|:-----------------------------------:|:-------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                PR-1                 | `MISSING_VARIANT_ID`                 | The required field `variant_id` must be present                                                                                                               |
|                PR-2                 | `WAREHOUSE_VARIANT_NOT_AVAILABLE`    | The `warehouse_variant_id` provided in the `sync_variant` object is not available at the moment                                                               |
|                PR-3                 | `WAREHOUSE_VARIANT_NOT_APPROVED`     | The warehouse `variant` related to the provided `warehouse_variant_id` wasn't approved                                                                        |
|                PR-4                 | `WAREHOUSE_PRODUCT_NOT_APPROVED`     | The warehouse `product` related to the provided `warehouse_variant_id` wasn't approved                                                                        |
|                PR-5                 | `SYNC_PRODUCT_SOFT_DELETED`          | The `sync_product` related to the provided `id` doesn't have any `sync_variants` related. Please add at least one `sync_variant`                              |
|                PR-6                 | `INVALID_FILTER_STATUS`              | The provided `status` is not a valid filter. Must be one of the following: `all`, `synced`, `unsynced`, `ignored`, `imported`, `discontinued`, `out_of_stock` |

## Orders API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div> | Explanation                                                                                                          |
|:-----------------------------------:|:-------------------------------------|:---------------------------------------------------------------------------------------------------------------------|
|                OR-1                 | `ORDER_NO_LONGER_EDITABLE`           | The `order` is no longer editable. To be editable via API must have one of the following statuses: `DRAFT`, `FAILED` |
|                OR-2                 | `STORE_TERMINATED`                   | The `store` related to the account is terminated                                                                     |
|                OR-3                 | `INCOMPATIBLE_ORDER_OPTIONS`         | The `order` cannot be confirmed and saved as a draft at the same time.                                               |
|                OR-4                 | `INVALID_SHIPPING_METHOD`            | The shipping method selected for the `order` is invalid or not supported                                             |
|                OR-5                 | `INVALID_ORDER_ITEM`                 | The `item` provided is not an array                                                                                  |
|                OR-6                 | `RESTRICTED_VARIANT`                 | The `variant` that you are trying to order is currently restricted                                                   |
|                OR-7                 | `INCOMPATIBLE_SYNC_VARIANT_ITEM`     | Associating Sync Variant with the existing `item` is not allowed                                                     |
|                OR-8                 | `INCOMPATIBLE_PRODUCT_TEMPLATE`      | Associating the Product Template with the existing `item` is not allowed                                             |
|                OR-9                 | `LIMITED_SHIPPING_SERVICE`           | The `product` related to the provided `product_id` can be only ordered to the US territory                           |
|                OR-10                | `UNSUPPORTED_FILE_TYPE`              | The file `type` provided is not supported via API                                                                    |
|                OR-11                | `MULTIPLE_LABEL_PRODUCT`             | Inside and outside labels can't be combined with each other on the same `product`                                    |
|                OR-12                | `DUPLICATED_FILE_TYPE`               | The provided `item` contains multiple files for `placement`                                                          |
|                OR-13                | `EXTERNAL_ID_IN_USE`                 | The `external_id` provided already exists and cannot be used                                                         |

## File Library API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div>                   | Explanation                                               |
|:-----------------------------------:|:-------------------------------------------------------|:----------------------------------------------------------|
|                FL-1                 | `INVALID_FILE_FORMAT`                                  | The URL provided returns a `file` with an invalid format  |
|                FL-2                 | `INVALID_FILE_URL`                                     | The `file` URL provided is invalid                        |
|                FL-3                 | `UNKNOW_FILE_ID`                                       | The `file` ID provided was not found                      |
|                FL-4                 | `FILE_NOT_ACCESIBLE`                                   | The specified `file` does not belong to this `store`      |
|                FL-5                 | `PRINT_FILE_PREVIEW`                                   | Temporary print file preview `file` cannot be used        |
|                FL-6                 | `FILE_LICENSED_ASSETS`                                 | Print files with `licensed assets` cannot be used         |
|                FL-7                 | `INVALID_FILE_DATA`                                    | Posted `file` data doesn't contain valid image data       |
|                FL-8                 | `MISSING_URL_DATA`                                     | The fields `url` or `data` must be present in the request |
|                FL-9                 | `FAILED_FILE_THREAD_COLORS`                            | The `file` image could not be processed                   |


## Shipping Rates API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div>                   | Explanation                                                           |
|:-----------------------------------:|:-------------------------------------------------------|:----------------------------------------------------------------------|
|                SH-1                 | `INVALID_COUNTRY_CODE`                                 | The `country_code` provided is invalid or missing                     |
|                SH-2                 | `SHIPPING_DISABLED`                                    | This happens when the shipping is disabled to the `country` specified |
|                SH-3                 | `INVALID_CURRENCY_CODE`                                | The `currency_code` provided is invalid or not supported              |


## Ecommerce platform Sync API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div> | Explanation                                                                                                                                                   |
|:-----------------------------------:|:-------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                EC-1                 | `INVALID_FILTER_STATUS`              | The provided `status` is not a valid filter. Must be one of the following: `all`, `synced`, `unsynced`, `ignored`, `imported`, `discontinued`, `out_of_stock` |
|                EC-2                 | `INVALID_PARAMETER_LENGTH`           | The `search` parameter should be at least 3 characters long                                                                                                   |
|                EC-3                 | `INVALID_OPTIONS_ELEMENT`            | The `options` field is incorrect or invalid                                                                                                                   |
|                EC-4                 | `NON_LABEL_FILES_MISSING`            | Non-label `files` must be provided for this `product`                                                                                                         |
|                EC-5                 | `UNSUPPORTED_INTEGRATION_SYNC`       | The integration of this `store` (_store name_) does not support syncing Printful `items`.                                                                     |
|                EC-6                 | `JEWELRY_PRODUCTS_NOT_AVAILABLE`     | The jewelry `products` are not available with this endpoint.                                                                                                  |


## Tax Rate API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div> | Explanation                                              |
|:-----------------------------------:|:-------------------------------------|:---------------------------------------------------------|
|                TR-1                 | `MISSING_RECIPIENT`                  | The `recipient` parameter must be present in the payload |
|                TR-2                 | `RECIPIENT_ERRORS`                   | Related to the collected errors found in the `recipient` |


## Store Information API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div> | Explanation                    |
|:-----------------------------------:|:-------------------------------------|:-------------------------------|
|                SI-1                 | `INVALID_PACKING_SLIP`               | The `data` provided is invalid |

## Mockup Generator API

| <div style="width:60px;">Code</div> | <div style="width:260px;">Name</div> | Explanation                                                       |
|:-----------------------------------:|:-------------------------------------|:------------------------------------------------------------------|
|                MG-1                 | `INVALID_ORIENTATION`                | Invalid `orientation` given                                       |
|                MG-2                 | `INVALID_TECHNIQUE`                  | Invalid `technique` given                                         |
|                MG-3                 | `UNAVAILABLE_GENERATOR_PRODUCT`      | The mockup generator is not available for the `product` specified |
|                MG-4                 | `GENERATOR_PRODUCT`                  | Related to general errors in the `payload`                        |
|                MG-5                 | `INVALID_FORMAT`                     | The specified `format` is invalid                                 |
|                MG-6                 | `INVALID_WIDTH`                      | The specified `width` is invalid                                  |
|                MG-7                 | `MISSING_REQUIRED_OPTION`            | Either `file` array or `product_template_id` should be present    |
|                MG-8                 | `DAILY_ALLOWANCE_EXCEEDED`           | The daily file count allowance is exceeded                        |


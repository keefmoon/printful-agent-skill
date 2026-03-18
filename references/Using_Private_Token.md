# Using Private Token


Once you have generated a **Private Token** in the **Developer Portal**,
you can use it to authenticate your API requests.

For the **Store** access level token:

```bash
curl --location \
    --request GET 'https://api.printful.com/{request_path}' \
    --header 'Authorization: Bearer {private_token}'
```

For an **Account** access level token, a store context is required:

```bash
curl --location \
    --request GET 'https://api.printful.com/{request_path}' \
    --header 'Authorization: Bearer {private_token}' \
    --header 'X-PF-Store-Id: {store_id}'
```

**Private Tokens** have an expiration date. Make sure to generate a new token and update
it in your service before the current one expires, as expired tokens cannot be refreshed
and must be replaced manually.



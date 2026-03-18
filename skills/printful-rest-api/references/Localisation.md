# Localisation

Some of the resources returned by the API are translated into several languages. By default, they are returned in
English in the API responses.

If you want to get a response with texts in another language, you can use the `X-PF-Language`
HTTP header. Its value should be the long version of the locale to use (e.g. `es_ES` for Spanish).
Possible localisation parameters:
- en_US
- en_GB
- en_CA
- es_ES
- fr_FR
- de_DE
- it_IT
- ja_JP

### Example

Product details (`GET https://api.printful.com/products/71`) response with default locale (`en_US`):

```
{
    "code": 200,
    "result": {
        "product": {
            "type_name": "T-Shirt",
            "title": "Unisex Staple T-Shirt | Bella + Canvas 3001",
            ...
        }
    }
}
```

Product details response with Spanish locale (`X-PF-Language: es_ES`):

```
{
    "code": 200,
    "result": {
        "product": {
            ...
            "type_name": "Camiseta",
            "title": "Camiseta esencial unisex | Bella + Canvas 3001",
            ...
        }
    }
}
```



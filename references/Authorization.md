# Authorization

This guide introduces the key concepts behind Printful OAuth, including
the differences between **Private Tokens** and **Public Apps**, available **client types**,
and how **scopes** control API access.

We'll start by defining the terminology used throughout this documentation and then
walk through the **Developer Portal**, where you can create and manage your tokens and apps
and acquire the credentials needed to authenticate API requests.

## Private tokens and public apps

There are two different ways of using Printful OAuth – **Private Token** and **Public App**.

Use a **Private Token** if you are developing an API solution for your personal store
and only your own services will connect to this API.

Use a **Public App** if you are building a public OAuth application that will be used
by a wider audience and connected by third-party users.

## Client types

There are two different client types – **Store** and **Account**.

A **Store** access level client can only access a specific store.
For _Private Token_, this store is specified when the token is created,
but for _Public App_, it is specified when the app is installed.

An **Account** level token, on the other hand, can manage all stores associated with the account.
For endpoints that require a specific store context, requests should include the `X-PF-Store-Id` header.
Account-level access is currently available for _Private Token_ only.

## Scopes

Scopes are used to limit access to specific resources and methods.
Scopes are defined in the **Developer Portal** when creating a **Private Token** or **Public App**.
Customers installing a **Public App** will see all requested permissions on the grant screen,
so be thoughtful about which scopes you request.

List of currently available scopes:

| Scope                    | Client type     | Description                                  |
|--------------------------|-----------------|----------------------------------------------|
| `orders`                 | Store & Account | Read and Write access to Orders              |
| `orders/read`            | Store & Account | Read access to Orders                        |
| `sync_products`          | Store & Account | Read and Write access to SyncProducts        |
| `sync_products/read`     | Store & Account | Read access to SyncProducts                  |
| `file_library`           | Store & Account | Read and Write access access to File library |
| `file_library/read`      | Store & Account | Read access to File library                  |
| `webhooks`               | Store & Account | Read and Write access to Webhooks            |
| `webhooks/read`          | Store & Account | Read access to Webhooks                      |
| `product_templates`      | Account         | Read and Write access to Product templates   |
| `product_templates/read` | Account         | Read access to Product templates             |

## Developer Portal

[Developer Portal](https://developers.printful.com) is designed for developers who want
to interact with the **Printful API**. In the **Developer Portal**, you can create and manage
your **Private Tokens** and **Public Apps**.

### Acquiring Private Tokens

**Private Tokens** can be generated in the **Developer Portal** on [Your tokens](https://developers.printful.com/tokens) page.
These tokens do not require refreshing and remain valid until they expire or are manually deleted.
Copy the token after creation, as it will not be available for viewing later in the **Developer Portal**.
Instructions on how to use them are provided in the [Using Private Token](#tag/Using-Private-Token) section.

### Acquiring OAuth tokens for Public Apps

For a **Public App** to make API requests on behalf of a customer, the customer must first authorize the app.
After the customer approves the requested scopes, your application can exchange
the returned `code` for OAuth access and refresh tokens.

To set this up, follow these steps:

- create an app in the **Developer Portal** on [Your apps](https://developers.printful.com/apps) page
- generate an app-specific installation URL using the `Client id`, `Client secret`, and other parameters
- redirect the user to the app installation url
- once the users returns to `redirect_url`, use the `code` parameter to acquire OAuth tokens

This section covers the initial setup steps. For details on authorizing users and exchanging
the `code` for tokens, see [Public App authorization](#tag/Public-App-authorization).



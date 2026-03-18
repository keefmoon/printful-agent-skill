# Rate Limiting

Printful implements rate limiting to ensure the stability of the API while
keeping it fair for all users. Rate limiting is handled differently depending
on the version and endpoint being used.

### Header values

Printful uses the following header values to communicate rate limits:

```
X-Ratelimit-Limit
X-Ratelimit-Remaining
X-Ratelimit-Reset
X-Ratelimit-Policy
```

However, the meaning of these values changes depending on the algorithm used.

### V1 Rate limitting

V1 used a very simple rate limiting mechanism, you had a limit of 120, every
request would reduce that limit by 1, if it hits 0 you would receive a 429 error,
every minute the limit of 120 would be restored. See: [V1 Rate Limits](#tag/Rate-Limiting).

This system is good, because it's easy to understand for most users. However,
we found it had a number of disadvantages both for users and for Printful.

- A burst in traffic could lead to 120 requests in the first second. This is
  harmful for Printful and will lead to a bad user experience as all end users
  wait for the rate limit to be reset.
- Some users will have close to 120 requests a minute, but most sites don't have
  such uniform traffic.

### V2 Rate Limiting

#### Leaky Bucket

In V2 of the API Printful implements a [Leaky Bucket](https://en.wikipedia.org/wiki/Leaky_bucket)
rate limiting algorithm.

**Note:** The algorithm does not implement any queuing mechanisms in its
current form so it can also be described as a [Token Bucket](https://en.wikipedia.org/wiki/Token_bucket)
algorithm.

Like with the old rate limiting algorithm, when you make a request to the API,
the `X-Ratelimit-Remaining` header value will go down by one. However, the `X-Ratelimit-Reset`
does not work the same, it states how long it will take to "refill the bucket".
The bucket will be refilled one drop at a time instead of all at once, so the
`X-Ratelimit-Reset` will be much smaller after the first request.

By default, after the first request the headers will look like this:

```
X-Ratelimit-Policy: 120;w=60;
X-Ratelimit-Limit: 120
X-Ratelimit-Remaining: 119
X-Ratelimit-Reset: 0.5
```

This means 120 requests can be made in a single burst, one request has already
been made and `X-Ratelimit-Remaining` will reach 120 again in 0.5 seconds. The
`X-Ratelimit-Policy` header indicates that in a window of 60 seconds 120 tokens
(or "quota units") will be added to the bucket.

The headers are inspired by the format in [RateLimit header fields for HTTP](https://www.ietf.org/archive/id/draft-ietf-httpapi-ratelimit-headers-07.html) Internet-Draft.

##### What is rate-limited in V2

All endpoints are rate-limited with the leaky bucket rate limiter and the defaults.
However, it is planned for future endpoints to have higher or lower limits depending
on usage and the capacity of the API.

##### Using the leaky bucket effectively

There are a number of strategies you can use to take advantage of rate limiting.

- When receiving a 429 you can use the `retry-after` header to check when the next
  request can be made, then schedule a request to trigger at that time.
- You can implement a queue that only sends out requests at the rate indicated
  in the policy. You can calculate the rate of requests using the
  `X-Ratelimit-Policy` 120 ÷ 60 = 0.5, so the queue should send requests out at a 
  rate of once every 500 milliseconds. 
- Some combination of the two, for example, allow requests to trigger at any rate,
  start queuing after the first 429, then turn off the queue after no new requests
  are sent and the `X-Ratelimit-Reset` time has been met.




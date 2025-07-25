# Ittybit Go Library

[![fern shield](https://img.shields.io/badge/%F0%9F%8C%BF-Built%20with%20Fern-brightgreen)](https://buildwithfern.com?utm_source=github&utm_medium=github&utm_campaign=readme&utm_source=https%3A%2F%2Fgithub.com%2Fittybit%2Fsdk-go)

The Ittybit Go library provides convenient access to the Ittybit API from Go.

## Usage

Instantiate and use the client with the following:

```go
package example

import (
    client "github.com/ittybit/sdk-go/client"
    option "github.com/ittybit/sdk-go/option"
    context "context"
    sdk "github.com/ittybit/sdk-go"
)

func do() () {
    client := client.NewClient(
        option.WithToken(
            "<token>",
        ),
    )
    client.Automations.Create(
        context.TODO(),
        &sdk.AutomationsCreateRequest{
            Name: sdk.String(
                "My Example Automation",
            ),
            Description: sdk.String(
                "This workflow will run whenever new media is created.",
            ),
            Trigger: &sdk.AutomationsCreateRequestTrigger{},
            Workflow: []*sdk.WorkflowTaskStep{
                &sdk.WorkflowTaskStep{
                    Kind: sdk.WorkflowTaskStepKindDescription,
                },
                &sdk.WorkflowTaskStep{
                    Kind: sdk.WorkflowTaskStepKindImage,
                    Ref: sdk.String(
                        "thumbnail",
                    ),
                },
                &sdk.WorkflowTaskStep{
                    Kind: sdk.WorkflowTaskStepKindVideo,
                    Next: []*sdk.WorkflowTaskStepNextItem{
                        &sdk.WorkflowTaskStepNextItem{
                            Kind: sdk.String(
                                "subtitles",
                            ),
                            Ref: sdk.String(
                                "subtitles",
                            ),
                        },
                    },
                },
            },
            Status: sdk.AutomationsCreateRequestStatusActive.Ptr(),
        },
    )
}
```

## Environments

You can choose between different environments by using the `option.WithBaseURL` option. You can configure any arbitrary base
URL, which is particularly useful in test environments.

```go
client := client.NewClient(
    option.WithBaseURL(ittybit.Environments.Default),
)
```

## Errors

Structured error types are returned from API calls that return non-success status codes. These errors are compatible
with the `errors.Is` and `errors.As` APIs, so you can access the error like so:

```go
response, err := client.Automations.Create(...)
if err != nil {
    var apiError *core.APIError
    if errors.As(err, apiError) {
        // Do something with the API error ...
    }
    return err
}
```

## Request Options

A variety of request options are included to adapt the behavior of the library, which includes configuring
authorization tokens, or providing your own instrumented `*http.Client`.

These request options can either be
specified on the client so that they're applied on every request, or for an individual request, like so:

> Providing your own `*http.Client` is recommended. Otherwise, the `http.DefaultClient` will be used,
> and your client will wait indefinitely for a response (unless the per-request, context-based timeout
> is used).

```go
// Specify default options applied on every request.
client := client.NewClient(
    option.WithToken("<YOUR_API_KEY>"),
    option.WithHTTPClient(
        &http.Client{
            Timeout: 5 * time.Second,
        },
    ),
)

// Specify options for an individual request.
response, err := client.Automations.Create(
    ...,
    option.WithToken("<YOUR_API_KEY>"),
)
```

## Advanced

### Retries

The SDK is instrumented with automatic retries with exponential backoff. A request will be retried as long
as the request is deemed retryable and the number of retry attempts has not grown larger than the configured
retry limit (default: 2).

A request is deemed retryable when any of the following HTTP status codes is returned:

- [408](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/408) (Timeout)
- [429](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/429) (Too Many Requests)
- [5XX](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/500) (Internal Server Errors)

Use the `option.WithMaxAttempts` option to configure this behavior for the entire client or an individual request:

```go
client := client.NewClient(
    option.WithMaxAttempts(1),
)

response, err := client.Automations.Create(
    ...,
    option.WithMaxAttempts(1),
)
```

### Timeouts

Setting a timeout for each individual request is as simple as using the standard context library. Setting a one second timeout for an individual API call looks like the following:

```go
ctx, cancel := context.WithTimeout(ctx, time.Second)
defer cancel()

response, err := client.Automations.Create(ctx, ...)
```

## Contributing

While we value open-source contributions to this SDK, this library is generated programmatically.
Additions made directly to this library would have to be moved over to our generation code,
otherwise they would be overwritten upon the next generated release. Feel free to open a PR as
a proof of concept, but know that we will not be able to merge it as-is. We suggest opening
an issue first to discuss with us!

On the other hand, contributions to the README are always very welcome!
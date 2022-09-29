# Nimbus config
When creating the Nimbus instance, we pass a series of variables, only one of them is mandatory: the `baseUrl`. In this topic we'll see every
configurable property, what they do and examples of how to change the default behavior.

## baseUrl
The base url to use when dealing with relative URLs in both the frontend and the server driven views (JSONs). This should be the URL to the server
that provides the server driven views.

## ui
The list of libraries containing the [components](component.md), [actions](action.md) and [operations](operation.md) for building the server driven
views.

## logger
The application's [Logger](/core/index.md#logger).

Example of Logger: TODO.

## urlBuilder
The application's [Url builder](/core/index.md#url-builder).

Example of URL Builder: TODO.

## httpClient
The application's [HTTP client](/core/index.md#http-client).

Example of HTTP client: TODO.

## viewClient
The application's [View client](/core/index.md#view-client).

Example of View client: TODO.

## idManager
The application's [ID manager](/core/index.md#id-manager).

Example of ID manager: TODO.

## loadingView
The Composable function to render when a server driven view is loading. By default, it renders a spinner (`CircularProgressIndicator`).

Example of a custom loading component: TODO.

## errorView
The Composable function to render when a server driven view fails to load and no fallback was provided. The error view always receive 2 arguments:
1. the error itself (`throwable: Throwable`);
2. a retry function (`retry: () -> Unit`) that when called sends the request again, replacing the error component with the loading component.

By default, it renders a simple text message with a button to retry.

Example of a custom error component: TODO.

# openUrl
> Not implemented on Android yet.

This action opens a URL. The URL is delegated to operating system so it can open it with the user's preferred app. For instance, a URL for a website
must be opened by the default browser, while a URL with a custom scheme must be opened by an app that can handle it.

The properties for the `openUrl` action are:

```typescript
interface OpenUrlProperties {
  url: string,
}
```

Where `url` is the url to open.

## Example
```json
{
  "_:action": "openUrl",
  "properties": {
    "url": "https://www.google.com" 
  }
}
```

The action above opens up Google in the default browser.

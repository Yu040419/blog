---
title: Securing Your Web Content with Content Security Policy (CSP)
date: 2024-02-13 17:59:12
tags:
  - 資訊安全
---

## What is CSP

**Content Security Policy**, or **CSP**, is a security standard implemented by web browsers to mitigate the risks of Cross-Site Scripting (XSS) attacks. It serves as an added layer of protection by defining and enforcing a set of rules that dictate which resources a browser is allowed to load for a specific web page. These resources include **scripts**, **stylesheets**, **images**, **fonts**, and more.

> Sometimes you may see mentions of the `X-Content-Security-Policy` header, but that's an older version and you don't need to specify it anymore.

## How To Implement CSP

```
Content-Security-Policy: <policy-directive>; <policy-directive>;

<policy-directive>: <directive> <source> <source>
```

### HTTP Response Header

```python=
class DemoPage(TemplateView):
    @method_decorator(never_cache)
    def dispatch(self, request, *args, **kwargs):
        response = super().dispatch(request, *args, **kwargs)
        response["Content-Security-Policy"] = "default-src 'self'; img-src https://*; child-src 'none';"
        return response
```

### HTML meta

```htmlembedded=
<meta
  http-equiv="Content-Security-Policy"
  content="default-src 'self'; img-src https://*; child-src 'none';" />
```

## Directives

- **Default-src**：Specifies the default sources for content if no other directive applies.
- **Script-src**：Defines valid sources for JavaScript, allowing you to whitelist specific domains or sources.

  - Inline scripts are blocked by default.

  ```htmlembedded=
  <script>
    runInlineScript();
  </script>
  ```

  - The execution of all JS event handlers from inline HTML markup are blocked by default.

  ```htmlembedded=
  <button onClick="runInlineScript();">
    All JS Event Handlers Blocked
  </button>

  <a href="javascript:runInlineScript();">Won't Run</a>
  ```

- **Style-src**：Similar to script-src but for stylesheets.
  - Inline styles are blocked by default.
  ```htmlembedded=
  <div style="color:red">
    All Inline style CSS is blocked by default
  </div>
  ```
- **Img-src**：Specifies the allowed sources for images.
- **Font-src**：Determines the sources for web fonts.
- **Connect-src**：Specifies the valid sources for network requests.
  - Applies to `XMLHttpRequest (AJAX)`, `WebSocket`, `fetch()`, `<a ping>` or `EventSource`. If not allowed the browser emulates a 400 HTTP status code.
- **Form-action**：Restricts the URLs where forms can be submitted.
- **Frame-ancestors**：Controls which websites can embed your content in frames.

## Source

| Source                            | Description                                                                                                                                                                                 | Example                                                                                      |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| \*                                | Used as a wildcard. It allows any URL except the `data:`, `blob:` and the `filesystem:` schemes.                                                                                            | script-src \* ;                                                                              |
| 'none'                            | Prevents loading resources from any source.                                                                                                                                                 | media-src 'none' ;                                                                           |
| 'self'                            | Allows loading resources from the same origin (host, and port).                                                                                                                             | default-src 'self' ;                                                                         |
| data:                             | Allows loading resources via the data scheme (e.g. Base64 encoded images).                                                                                                                  | img-src data: ;                                                                              |
| specificDomain.com                | Allows loading resources from the specific domain                                                                                                                                           | media-src specificDomain.com ;                                                               |
| \*.example.com                    | Allows loading resources from any subdomain under example.com.                                                                                                                              | script-src \*.example.com                                                                    |
| https:                            | Allows loading resources only over HTTPS on any domain.                                                                                                                                     | object-src https: ;                                                                          |
| 'unsafe-inline'                   | Allows use of inline source elements such as style attribute, onclick, or script tag bodies (depends on the context of the source it is applied to) and javascript: URIs                    | script-src 'unsafe-inline' ;                                                                 |
| 'unsafe-eval'                     | Allows unsafe dynamic code evaluation such as JavaScript `eval()`                                                                                                                           | script-src 'unsafe-eval' ;                                                                   |
| 'nonce-<base64-value>'            | An allowlist for specific inline scripts using a cryptographic nonce (number used once).                                                                                                    | `<script nonce="2726c7f26c"> doSomethingBad() </script>` <br> script-src 'nonce-2726c7f26c'; |
| '<hash-algorithm>-<base64-value>' | A sha256, sha384 or sha512 hash of scripts or styles. This value consists of the algorithm used to create the hash followed by a hyphen and the base64-encoded hash of the script or style. | `<script>doSomething();</script>` <br> script-src 'sha256-RFWPLDbv2BY+rC';                   |

## Practice

Valid Image resource

```
maps.gstatic.com - loads various img assets for the map
maps.googleapis.com - loads tiles of the map
data:image/svg+xml - several resources are loaded as SVG using data URIs
khms0.googleapis.com - load satalite images for the map. This could be from a few different similar subdomains.
geo0.ggpht.com - loads street view images, this could be from a few different similar subdomains.
```

**ANSWER**

```
img-src data: maps.gstatic.com *.googleapis.com *.ggpht.com
```

## Tips

- **Open your devtool, and check Response Headers of Network tab**
  - Facebook
  - HackMD
  - MDN
- **Syntax Error**
  - No colon between directive and source `:`
  - Separate policy With semicolon `;`
  - Separate source with space ` `
  - Specific source with single quote `'self'`
- [**Testing Tools by Google**](https://csp-evaluator.withgoogle.com/)
- **Report Only Mode**: During implementation, use the [Content-Security-Policy-Report-Only](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only) header to receive reports about policy violations without enforcing the policy.
  - `Content-Security-Policy-Report-Only: report-to /csp-report/`
- **Blocked Content**: If you notice certain resources are not loading, check the browser console for CSP error messages. Adjust your policy accordingly.
- [**Browser Compatibility**](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP#browser_compatibility)

## References

- [Content-Security-Policy - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
- [Content-Security-Policy (v2014) | DEVCORE](https://devco.re/blog/2014/04/08/security-issues-of-http-headers-2-content-security-policy/)
- [Content Security Policy Cheat Sheet | OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [Content Security Policy (CSP) Quick Reference Guide](https://content-security-policy.com/)

## Conclusion

- Powerful
- Careful
- Keep changing

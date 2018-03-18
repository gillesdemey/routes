# octokit-rest-routes

> machine-readable, always up-to-date GitHub REST API route specifications

[![Build Status](https://travis-ci.org/gr2m/octokit-rest-routes.svg?branch=master)](https://travis-ci.org/gr2m/octokit-rest-routes)

⚠️ This module is work in progress. It does not follow [semantic versioning](https://semver.org/) yet. Feedback welcome :)

## Usage

```
const ROUTES = require('@gr2m/octokit-rest-routes')
```

returns an object with keys being the route scopes such as `activity`, `issues`,
`repositories`, etc (one for navigation header in the sidebar at https://developer.github.com/v3/).

The value for each scope is an array of REST API endpoint specification.

If you don’t need all routes definitions, you can require a scope or a specific
route definition instead

```js
const REPO_ROUTES = require('@gr2m/octokit-rest-routes/routes/repos')
const GET_REPO_ROUTE = require('@gr2m/octokit-rest-routes/routes/repos/get')
```

## Example

Here is an excerpt for reference

```
{
  ...
  "repositories": [
    {
      "name": "List your repositories",
      "enabledForApps": false,
      "method": "GET",
      "path": "/user/repos",
      "description": "List repositories that are accessible to ...",
      "params": [
        {
          "name": "visibility",
          "type": "enum",
          "options": [
            "all",
            "public",
            "private"
          ],
          "description": "Can be one of `all`, `public`, or `private`.",
          "default": "all",
          "required": false
        },
        ...
      ],
      "documentationUrl": "https://developer.github.com/v3/repos/#list-your-repositories"
    },
    ...
  ],
  ...
}
```

## How it works

This package updates itself using a daily cronjob running on Travis. All
[`routes/*.json`](routes/) files as well as [`index.js`](index.js) is
generated by [`bin/octokit-rest-routes.js`](bin/octokit-rest-routes.js). Run
 `node bin/octokit-rest-routes.js --usage` for instructions.

The update script is scraping [GitHub’s REST API](https://developer.github.com/v3/)
documentation pages and extracts the meta information using [cheerio](https://www.npmjs.com/package/cheerio)
and a ton of regular expressions :)

For simpler local testing and tracking of changes all loaded pages are cached
in the [`cache/`](cache/) folder.

### 1. Find documentation pages

- Index page cached in [`cache/v3/index.html`](cache/v3/index.html)
- Result cached in [`cache/pages.json`](cache/pages.json)

Opens https://developer.github.com/v3/, find all documentation page URLs
in the side bar navigation.

### 2. On each documentation page, finds sections

- Documentation pages cached in `cache/v3/*/index.html`, e.g. [`cache/v3/repos/index.html`](cache/v3/repos/index.html)
- Documentation sub pages cached in `cache/v3/*/*/index.html`, e.g. [`cache/v3/repos/branches/index.html`](cache/v3/repos/branches/index.html)
- All sections found on pages are cached next to the `index.html` files, e.g. [`cache/v3/repos/sections.json`](cache/v3/repos/sections.json)
- HTML of sections are stored next to the `index.html` files, e.g. [`cache/v3/repos/create.html`](cache/v3/repos/create.html)

Loads HTML of each documentation page, finds sections in page.

### 3. In each section, finds endpoints

- Some sections don’t have endpoints, such as [Notifications Reasons](https://developer.github.com/v3/activity/notifications/#notification-reasons)
- Some sections have multiple endpoints, see [#3](https://github.com/gr2m/octokit-rest-routes/issues/3)

Loads HTML of documentation page section. Turns it into [`routes/*.json`](routes/) file.
In some cases the HTML cannot be turned into an endpoint using the implemented patterns.
For these cases [custom overrides](lib/endpoint/overrides) are defined.

## LICENSE

[MIT](LICENSE.md)

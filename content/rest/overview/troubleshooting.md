---
title: Troubleshooting
intro: Learn how to resolve the most common problems people encounter in the REST API.
redirect_from:
  - /v3/troubleshooting
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
topics:
  - API
---



If you're encountering some oddities in the API, here's a list of resolutions to
some of the problems you may be experiencing.

{% ifversion api-date-versioning %}

## `400` error for an unsupported API version

You should use the `X-GitHub-Api-Version` header to specify an API version. For example:

```shell
curl {% data reusables.rest-api.version-header %} https://api.github.com/zen
```

If you specify a version that does not exist, you will receive a `400` error.

For more information, see "[AUTOTITLE](/rest/overview/api-versions)."

{% endif %}

## `404` error for an existing repository

Typically, we send a `404` error when your client isn't properly authenticated.
You might expect to see a `403 Forbidden` in these cases. However, since we don't
want to provide _any_ information about private repositories, the API returns a
`404` error instead.

To troubleshoot, ensure [you're authenticating correctly](/rest/quickstart), [your OAuth access token has the required scopes](/apps/oauth-apps/building-oauth-apps/scopes-for-oauth-apps), [third-party application restrictions][oap-guide] are not blocking access, and that [the token has not expired or been revoked](/authentication/keeping-your-account-and-data-secure/token-expiration-and-revocation).

## Not all results returned

Most API calls accessing a list of resources (_e.g._, users, issues, _etc._) support
pagination. If you're making requests and receiving an incomplete set of results, you're
probably only seeing the first page. You'll need to request the remaining pages
in order to get more results.

It's important to _not_ try and guess the format of the pagination URL. Not every
API call uses the same structure. Instead, extract the pagination information from
the link header, which is returned with every request. For more information about pagination, see "[AUTOTITLE](/rest/guides/using-pagination-in-the-rest-api)."

[oap-guide]: https://developer.github.com/changes/2015-01-19-an-integrators-guide-to-organization-application-policies/

{% ifversion fpt or ghec %}
## Basic authentication errors

On November 13, 2020 username and password authentication to the REST API and the OAuth Authorizations API were deprecated and no longer work.

### Using `username`/`password` for basic authentication

If you're using `username` and `password` for API calls, then they are no longer able to authenticate. For example:

```bash
curl -u YOUR-USERNAME:YOUR-PASSWORD https://api.github.com/user/repos
```

Instead, use a {% data variables.product.pat_generic %} or an access token for a {% data variables.product.prodname_github_app %} when testing endpoints or doing local development:

```bash
curl -H 'Authorization: Bearer YOUR-TOKEN' https://api.github.com/user/repos
```

For more information, see "[AUTOTITLE](/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)" and "[AUTOTITLE](/apps/creating-github-apps/authenticating-with-a-github-app/about-authentication-with-a-github-app)."

For {% data variables.product.prodname_oauth_app %}s, you should use the [web application flow](/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps#web-application-flow) to generate an OAuth token to use in the API call's header:

```bash
curl -H 'Authorization: Bearer YOUR-OAUTH-TOKEN' https://api.github.com/user/repos
```

## Timeouts

If  {% data variables.product.product_name %} takes more than 10 seconds to process an API request, {% data variables.product.product_name %} will terminate the request and you will receive a timeout response.

{% endif %}

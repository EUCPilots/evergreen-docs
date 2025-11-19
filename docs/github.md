---
layout: doc
---
# Working with GitHub rate limits

Over 150 applications supported by Evergreen are hosted on GitHub repositories or pull data from a GitHub repository for determining the latest version. Evergreen makes use of the GitHub REST API when requesting details for these applications.

GitHub implements rate limits on the API for unauthenticated requests. See [Rate limits for the REST API](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api) for details.

## Authenticating to the GitHub API

For environments that may consume more than 60 requests to the API in an hour, Evergreen supports authenticated requests by use of [personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens). [Fine-grained personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) should be used.

A token that has read-only access to public repositories only is recommended. This should enable you to use a long expiry time; however, secure management of this token is recommended.

![Fine-grained personal access token](/img/github-pat.png)

## GITHUB_TOKEN environment variable

Evergreen will use the token stored in the `GITHUB_TOKEN` or `GH_TOKEN` environment variables. Setting either of these variables with the value of the personal access token will allow Evergreen to use the token when querying the GitHub API, significantly increasing the number of available requests.

### GitHub Actions

Additionally, if you are running Evergreen in a GitHub Actions workflow, the `GITHUB_TOKEN` will be used, so no additional personal access tokens are required - [Use GITHUB_TOKEN for authentication in workflows](https://docs.github.com/en/actions/tutorials/authenticate-with-github_token).

For example, add the following code to the top of your workflow file:

```yaml
# Environment variables
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

# action.js

> GitHub API client for GitHub Actions

[![@latest](https://img.shields.io/npm/v/@octokit/action.svg)](https://www.npmjs.com/package/@octokit/action)
[![Build Status](https://github.com/octokit/action.js/workflows/Test/badge.svg)](https://github.com/octokit/action.js/actions)
[![Greenkeeper](https://badges.greenkeeper.io/octokit/action.js.svg)](https://greenkeeper.io/)

## Usage

<table>
<tbody valign=top align=left>
<tr><th>
Browsers
</th><td width=100%>

`@octokit/action` is not meant for browser usage.

</td></tr>
<tr><th>
Node
</th><td>

Install with `npm install @octokit/action`

```js
const { Octokit } = require("@octokit/action");
// or: import { Octokit } from "@octokit/action";
```

</td></tr>
</tbody>
</table>

You can pass `secret.GITHUB_TOKEN` or any of your own secrets to a Node.js script. For example

```yml
name: My Node Action
on:
  - pull_request
jobs:
  my-action:
    runs-on: ubuntu-latest
    steps:
      # Check out code using git
      - uses: actions/checkout@v2
      # Install Node 12
      - uses: actions/setup-node@v1
        with:
          version: 12
      # Node.js script can be anywhere. A good convention is to put local GitHub Actions
      # into the `.github/actions` folder
      - run: node .github/actions/my-script.js
        with:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Setting `GITHUB_TOKEN` on either [`with:`](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#jobsjob_idstepswith) or [`env:`](https://help.github.com/en/actions/reference/workflow-syntax-for-github-actions#env) will work.

```js
// .github/actions/my-script.js
const octokit = new Octokit();

// `octokit` is now authenticated using GITHUB_TOKEN
```

### Create an issue using REST API

```js
const octokit = new Octokit();
const [owner, repo] = process.env.GITHUB_REPOSITORY.split("/");

// See https://developer.github.com/v3/issues/#create-an-issue
const { data } = await octokit.request("POST /repos/:owner/:repo/issues", {
  owner,
  repo,
  title: "My test issue",
});
console.log("Issue created: %d", data.html_url);
```

You can also use `octokit.issues.create({ owner, repo, title })`. See the [REST endpoint methods plugin](https://github.com/octokit/plugin-rest-endpoint-methods.js/) for a list of all available methods.

### Create an issue using GraphQL

```js
const octokit = new Octokit();
const eventPayload = require(process.env.GITHUB_EVENT_PATH);
const repositoryId = eventPayload.repository.node_id;

const response = await octokit.graphql(
  `
  mutation($repositoryId:ID!, $title:String!) {
    createIssue(input:{repositoryId: $repositoryId, title: $title}) {
      issue {
        number
      }
    }
  }
  `,
  {
    repositoryId,
    title: "My test issue",
  }
);
```

### Hooks, plugins, and more

`@octokit/action` is build upon `@octokit/core`. Refer to [its README](https://github.com/octokit/core.js#readme) for the full API documentation.

## How it works

`@octokit/action` is simply a [`@octokit/core`](https://github.com/octokit/core.js#readme) constructor, pre-authenticate using [`@octokit/auth-action](https://github.com/octokit/auth-action.js#readme).

The source code is … simple: [`src/index.ts`](src/index.ts).

## License

[MIT](LICENSE)

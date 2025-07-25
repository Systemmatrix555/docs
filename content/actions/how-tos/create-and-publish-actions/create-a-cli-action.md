---
title: Creating a third party CLI action
shortTitle: Create a CLI action
intro: 'Learn how to develop an action to set up a CLI on {% data variables.product.prodname_actions %} runners.'
redirect_from:
  - /actions/creating-actions/developing-a-third-party-cli-action
  - /actions/sharing-automations/creating-actions/developing-a-third-party-cli-action
  - /actions/how-tos/sharing-automations/creating-actions/developing-a-third-party-cli-action
  - /actions/how-tos/creating-and-publishing-actions/creating-a-third-party-cli-action
versions:
  fpt: '*'
  ghec: '*'
topics:
  - Actions
---

## Introduction

You can write an action to provide a way for users to access your servers via a configured CLI environment on {% data variables.product.prodname_actions %} runners.

Your action should:

* Make it simple for users to specify the version of the CLI to install
* Support multiple operating systems
* Run in an efficient fashion to minimize run-time and associated costs
* Work across {% data variables.product.github %}-hosted and self-hosted runners
* Leverage community tooling when possible

This article will demonstrate how to write an action that retrieves a specific version of your CLI, installs it, adds it to the path, and (optionally) caches it. This type of action (an action that sets up a tool) is often named `setup-$TOOL`.

## Prerequisites

You should have an understanding of how to write a custom action. For more information, see [AUTOTITLE](/actions/how-tos/creating-and-publishing-actions/managing-custom-actions).

## Example

The following script demonstrates how you can get a user-specified version as input, download and extract the specific version of your CLI, then add the CLI to the path.

{% data variables.product.prodname_dotcom %} provides [`actions/toolkit`](https://github.com/actions/toolkit), which is a set of packages that helps you create actions. This example uses the [`actions/core`](https://github.com/actions/toolkit/tree/main/packages/core) and [`actions/tool-cache`](https://github.com/actions/toolkit/tree/main/packages/tool-cache) packages.

{% raw %}

```javascript copy
const core = require('@actions/core');
const tc = require('@actions/tool-cache');

async function setup() {
  // Get version of tool to be installed
  const version = core.getInput('version');

  // Download the specific version of the tool, e.g. as a tarball
  const pathToTarball = await tc.downloadTool(getDownloadURL());

  // Extract the tarball onto the runner
  const pathToCLI = await tc.extractTar(pathToTarball);

  // Expose the tool by adding it to the PATH
  core.addPath(pathToCLI)
}

module.exports = setup
```

{% endraw %}

To use this script, replace `getDownloadURL` with a function that downloads your CLI. You will also need to create an actions metadata file (`action.yml`) that accepts a `version` input and that runs this script. For full details about how to create an action, see [AUTOTITLE](/actions/creating-actions/creating-a-javascript-action).

## Further reading

This pattern is employed in several actions. For more examples, see:

* [`ruby/setup-ruby`](https://github.com/ruby/setup-ruby)
* [`google-github-actions/setup-gcloud`](https://github.com/google-github-actions/setup-gcloud)
* [`hashicorp/setup-terraform`](https://github.com/hashicorp/setup-terraform)

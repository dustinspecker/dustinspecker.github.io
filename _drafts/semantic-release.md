---
layout: post
title: Setting up Semantic Release
---
[![semantic-release](https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg?style=flat-square)](https://github.com/semantic-release/semantic-release)

# Introduction to Semantic Release
[Semantic Release](https://github.com/semantic-release/semantic-release) aims to automate and ease the npm publishing process.

When setup, it automates the following steps of a release:

1. Verify tests are succesful
1. Bump the package version in `package.json`
 - Semantic Release analyzes commits to determine if a patch, minor, or major should be bumped. By default Semantic Release expects commits to be appropriately labeled as a fix, new feature, or a breaking change.
1. Create a Git tag for the new version **with a change log**
 - The change log generation also examines the commit messages to determine any fixes, new features, or breaking changes.
1. Push the newly created tag to the project's Git repostiory
1. Perform a `npm publish`

# Set Up
Setting up Semantic Release on an existing npm module is simple.

1. Run `npm install --global semantic-release-cli` to install the Semantic Release command line tool.
 - This tool will be used to modify a project's `.travis.yml` and `package.json`.
1. In the npm package's directory, run `semantic-release-cli setup`.
 - It will ask a few questions about the project. It will also ask for your GitHub and npm credentials. It will then create a [Personal access token](https://github.com/settings/tokens) on GitHub for write permissions to the repository.

The setup process will result in a few changed files.

First, the `package.json` will include a `semantic-release` `devDependency`. There will also be a `semantic-releae` npm script created. Also, the `version` key will be missing from `package.json`. Semantic Release will add the version key right before performing the `npm publish` later on. Some Semantic Release users opt to have a `version` key with a value of `0.0.0-semantically-released` or similar to signify that an automated release is expected.

Next, `.travis.yml` will have a recommended configuration and will execute the `semantic-release` npm script.

At this point, it is fine to commit these changes. The setup for Semantic Release is complete.

# Using Semantic Release

From now on, Semantic Release will automatically run on Travis CI on every new commit pushed to the `master` branch. Semantic Release expects a particular Git commit message style, the Angular style.

A few examples of Angular style commits:

 - `feat(home): add link to user settings`
 - `fix(nav): prevent error when user submits empty form`
 - `docs(readme): add code coverage badge`

[Commitizen](https://github.com/commitizen/cz-cli) is a great tool for interactively creating this style of commits.

Any breaking changes should have the following format:

```
feat(add): accept only positive numbers

BREAKING CHANGE:

The add function now long accepts positive numbers. Passing
negative numbers will result in an `Error` being thrown.
```

Semantic Release is looking for `BREAKING CHANGE:` in commit messages to determine if the commit is a breaking change. **A potential gotcha is omitting the `:` and writing `BREAKING CHANGE` will prevent Semantic Release from determining the commit is a breaking change.**

# Gotcha Encountered

One gotcha that bit me for a while is the format of `repository` in `package.json`.

Instead of using the short form

```json
{
  "repository": "dustinspecker/package"
}
```

Semantic Release is expecting the long form

```json
{
  "repository": {
    "type": "git",
    "url": "https://github.com/dustinspecker/package.git"
  }
}
```

If `semantic-release-cli setup` hangs after answering the prompts I would recommend checking the `repository` field.

# Further Steps

There exist a few plugins for Semantic Release with more being actively developed. A few note-worthy ones:

 - [Cracks](https://github.com/semantic-release/cracks) - Runs the current release's tests against the next release's source. If these tests fail then the change is a breaking change.
 - [dont-break](https://github.com/semantic-release/semantic-release/issues/65) - The early phases of a plugin that runs the test suites of packages that depend on the package to be published. If these tests fail then a change is probably a breaking change.
- [india](https://github.com/semantic-release/semantic-release/issues/66) - India aims to analyze any JSDoc comments to determine breaking changes.

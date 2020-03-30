# GitHub Actions and GitHub Packages

## 将 js package 发布到 GitHub Packages

Official Doc: [Publishing packages to npm and GitHub Packages](https://help.github.com/en/actions/language-and-framework-guides/publishing-nodejs-packages#publishing-packages-to-npm-and-github-packages)

Example: [baurine/grafana-value-formats](https://github.com/baurine/grafana-value-formats)

```yaml
name: Node.js Package
on:
  push:
    branches:
      - release
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      # Setup .npmrc file to publish to GitHub Packages
      - uses: actions/setup-node@v1
        with:
          node-version: '12.x'
          registry-url: 'https://npm.pkg.github.com'
          scope: '@baurine' # Defaults to the user or organization that owns the workflow file
      - run: yarn
      - run: yarn publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      - uses: actions/setup-node@v1
        with:
          registry-url: 'https://registry.npmjs.org'
      - run: yarn
      - run: yarn publish --access public
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

`secrets.GITHUB_TOKEN` 是 GitHub 内置的，不需要额外设置。`secrets.NPM_TOKEN`，需要先在 npm.js 注册后在个人设置中创建 auth token，然后在 GitHub 相应 repo 的 setting 中加入到 secrets 面板中。

## 在本地如何使用 GitHub Packages 中的包

Official Doc: [Installing a package](https://help.github.com/en/packages/using-github-packages-with-your-projects-ecosystem/configuring-npm-for-use-with-github-packages#installing-a-package)

Example: [baurine/pd-client-js](https://github.com/baurine/pd-client-js)

1. 在 GitHub 上创建 personal access token

   ["Settings" -> "Developer settings" -> "Personal access tokens" -> "Generate new token"](https://github.com/settings/tokens/new) 路径, "Select scopes" 至少选中 "repo" 和 "read:packages"。

1. 编辑用户级别的 `~/.npmrc`，添加一条新记录：

   ```
   //npm.pkg.github.com/:_authToken=[YOUR_PERSONAL_ACCESS_TOKEN]
   ```

1. 在项目根目录下创建 `.npmrc`，写入如下内容：

   ```
   registry=https://npm.pkg.github.com/pingcap-incubator
   ```

   如果想从多个用户或组织的 GitHub Packages 中安装包，则 `.npmrc` 长这样：

   ```
   registry=https://npm.pkg.github.com/pingcap-incubator
   @baurine:registry=https://npm.pkg.github.com
   ```

   如果使用 yarn，则创建 `.yarnrc`，而不是 `.npmrc`，格式和内容如下：

   ```
   "@pingcap-incubator:registry" "https://npm.pkg.github.com"
   "@baurine:registry" "https://npm.pkg.github.com"
   ```

1. 安装

   ```sh
   $ npm install @pingcap-incubator/pd-client-js
   ```

   如果使用 yarn，则需要添加 `"@pingcap-incubator/pd-client-js": "^0.1.5"` 到 package.json 中，然后执行 `yarn install --update-checksums`，相关的 issue: [Integrity checked failed error](https://github.com/yarnpkg/yarn/issues/7552)。

## 在 GitHub Actions 安装来自 GitHub Packages 的包

首先，此项目肯定在本地已经测试 OK 了，也就是说按照上面的步骤创建了项目级别的 .npmrc 或 .yarnrc，并提交到 repo 中了，但用户级别的 `~/.npmrc` 没法提交到 repo 中，这时就要在 GitHub Actions 加上生成用户级别的 .npmrc 的步骤。

```yaml
jobs:
  release_assets:
    name: Release UI Assets
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@master
      # Setup .npmrc file
      - uses: actions/setup-node@v1
        with:
          node-version: '12.x'
          registry-url: 'https://npm.pkg.github.com'
```

Ref: [Example using a private registry and creating the .npmrc file](https://help.github.com/en/actions/language-and-framework-guides/using-nodejs-with-github-actions#example-using-a-private-registry-and-creating-the-npmrc-file)

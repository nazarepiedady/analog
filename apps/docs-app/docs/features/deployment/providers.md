import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Providers

Analog supports deployment to many providers with little or no additional configuration using [Nitro](https://nitro.unjs.io) as its underlying server engine. You can find more providers in the [Nitro deployment docs](https://nitro.unjs.io/deploy).

## Zerops

:::info
[Zerops](https://zerops.io) is the **official** deployment partner for AnalogJS.
:::

Analog supports deploying both static and server-side rendered apps to [Zerops](https://zerops.io) with a simple configuration file.

> One Zerops project can contain multiple Analog projects. See example repositories for [static](https://github.com/zeropsio/recipe-analog-static) and [server-side rendered](https://github.com/zeropsio/recipe-analog-nodejs) Analog apps for a quick start.

### Static (SSG) Analog app

If your project is not SSG Ready, set up your project for [Static Site Generation](/docs/features/server/static-site-generation).

#### 1. Create a project in Zerops

Projects and services can be added either through a [Project add](https://app.zerops.io/dashboard/project-add) wizard or imported using a YAML structure:

```yml
project:
  name: recipe-analog
services:
  - hostname: app
    type: static
```

This creates a project called `recipe-analog` with a Zerops Static service called `app`.

#### 2. Add zerops.yml configuration

To tell Zerops how to build and run your site, add a `zerops.yml` to your repository:

```yml
zerops:
  - setup: app
    build:
      base: nodejs@20
      buildCommands:
        - pnpm i
        - pnpm build
      deployFiles:
        - public
        - dist/analog/public/~
    run:
      base: static
```

#### 3. [Trigger the build & deploy pipeline](#build--deploy-your-code)

### Server-side rendered (SSR) Analog app

If your project is not SSR Ready, set up your project for [Server Side Rendering](/docs/features/server/server-side-rendering).

#### 1. Create a project in Zerops

Projects and services can be added either through a [Project add](https://app.zerops.io/dashboard/project-add) wizard or imported using a YAML structure:

```yml
project:
  name: recipe-analog
services:
  - hostname: app
    type: nodejs@20
```

This creates a project called `recipe-analog` with a Zerops Node.js service called `app`.

#### 2. Add zerops.yml configuration

To tell Zerops how to build and run your site, add a `zerops.yml` to your repository:

```yml
zerops:
  - setup: app
    build:
      base: nodejs@20
      buildCommands:
        - pnpm i
        - pnpm build
      deployFiles:
        - public
        - node_modules
        - dist
    run:
      base: nodejs@20
      ports:
        - port: 3000
          httpSupport: true
      start: node dist/analog/server/index.mjs
```

#### 3. [Trigger the build & deploy pipeline](#build-deploy-your-code)

---

### Build & deploy your code

#### Trigger the pipeline by connecting the service with your GitHub / GitLab repository

Your code can be deployed automatically on each commit or a new tag by connecting the service with your GitHub / GitLab repository. This connection can be set up in the service detail.

#### Trigger the pipeline using Zerops CLI (zcli)

You can also trigger the pipeline manually from your terminal or your existing CI/CD by using Zerops CLI.

1. Install the Zerops CLI.

```bash
# To download the zcli binary directly,
# use https://github.com/zeropsio/zcli/releases
npm i -g @zerops/zcli
```

2. Open [Settings > Access Token Management](https://app.zerops.io/settings/token-management) in the Zerops app and generate a new access token.

3. Log in using your access token with the following command:

```bash
zcli login <token>
```

4. Navigate to the root of your app (where `zerops.yml` is located) and run the following command to trigger the deploy:

```bash
zcli push
```

#### Trigger the pipeline using GitHub / Gitlab

You can also check out [Github Integration](https://docs.zerops.io/references/github-integration) / [Gitlab Integration](https://docs.zerops.io/references/gitlab-integration) in [Zerops Docs](https://docs.zerops.io/) for git integration.

## Netlify

Analog supports deploying on [Netlify](https://netlify.com/) with minimal configuration.

### Deploying the Project

<Tabs groupId="porject-type">
  <TabItem label="Create analog" value="create-analog">
In the build settings of your Netlify project, set the [publish directory](https://docs.netlify.com/configure-builds/overview/#definitions) to `dist/analog/public` to deploy the static assets and the [functions directory](https://docs.netlify.com/configure-builds/overview/#definitions) to `dist/analog` to deploy the server.
  </TabItem>

  <TabItem label="Nx" value="nx">
In the build settings of your Netlify project on the web UI, do the following.
1. Set the [build command](https://docs.netlify.com/configure-builds/overview/#definitions) to `nx build [your-project-name]`
2. Set the [publish directory](https://docs.netlify.com/configure-builds/overview/#definitions) to `dist/[your-project-name]/analog/public` to deploy the static assets
3. Set the [functions directory](https://docs.netlify.com/configure-builds/overview/#definitions) to `dist/[your-project-name]/analog` to deploy the server.

You can also configure this by putting a `netlify.toml` at the root of your repository. Below is an example config.

```toml
# replace "my-analog-app" with the name of the app you want to deploy
[build]
  command = "nx build my-analog-app"
  publish = "dist/my-analog-app/analog/public"
  functions = "dist/my-analog-app/analog"
```

  </TabItem>
</Tabs>

## Vercel

Analog supports deploying on [Vercel](https://vercel.com/) with no additional configuration.

### Deploying the Project

<Tabs groupId="porject-type">
  <TabItem label="Create analog" value="create-analog">
By default, when deploying to Vercel, the build preset is handled automatically.

1. Create a new project and select the repository that contains your code.

2. Click 'Deploy'.

And that's it!

  </TabItem>

  <TabItem label="Nx" value="nx">
In order to make it work with Nx, we need to define the specific app we want to build. There are several ways to do this, and you can choose one of the following methods (replace &#60;app&#62; with your app name):

1. Define the `defaultProject` in your `nx.json`

```json [nx.json]
{
  "defaultProject": "<app>"
}
```

2. Create a `vercel.json` file in the root of your project and define the `buildCommand`:

```json [vercel.json]
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "buildCommand": "nx build <app>"
}
```

3. Define the `buildCommand` in your `package.json`:

```json [package.json]
{
  "scripts": {
    "build": "nx build <app>"
  }
}
```

#### Nx and Vercel

When using Nx and reusing the build cache on the Vercel build platform, there is a possibility that the cache is reused if you have built it locally. This can lead to the output being placed in the wrong location. To resolve this issue, you can use the preset in the `vite.config.ts` file as a workaround.

  </TabItem>
</Tabs>

### Setting the Preset Manually

There might be a case where Vercel doesn't load the preset automatically. In that case, you can do one of the following.

- Set the `BUILD_PRESET` environment variable to `vercel`.
- Set the preset in the `vite.config.ts` file:

```ts [vite.config.ts]
import { defineConfig } from 'vite';
import analog from '@analogjs/platform';

// https://vitejs.dev/config/
export default defineConfig(({ mode }) => ({
  /// ...other config
  plugins: [
    analog({
      nitro: {
        preset: 'vercel',
      },
    }),
  ],
}));
```

## Cloudflare Pages

Analog supports deploying to [Cloudflare](https://cloudflare.com/) Pages with minimal configuration.

### Deploying to Cloudflare

To connect your repository and deploy automatically to Cloudflare:

1. Log in to the Cloudflare dashboard and select your account.
2. In Account Home, select Workers & Pages.
3. Select Create application > Pages > Connect to Git.
4. Enter `npm run build` as the `Build Command`.
5. Enter `dist/analog/public` as the `Build output directory`.
6. Leave the other default settings, click `Save and Deploy`.

The application deploys to Cloudflare's network on each push to the repository.

#### Nx and Cloudlfare

For Nx workspaces, the build output is under the app name. Update the `Build output directory` accordingly.

For example:

Build output directory: `dist/[your-project-name]/analog/public`

To test the build locally, run the following command:

```bash
BUILD_PRESET=cloudflare-pages npx nx build [your-project-name]
```

### Running the application locally using Wrangler

You can also preview the application running on Cloudflare locally:

1. Set the environment variable `BUILD_PRESET` to `cloudflare-pages` before running the build

```bash
BUILD_PRESET=cloudflare-pages npm run build
```

2. Use the `wrangler` CLI to run the application locally

```bash
npx wrangler pages dev ./dist/analog/public
```

## Firebase App Hosting

Analog supports [Firebase App Hosting](https://firebase.google.com/docs/app-hosting) with no additional configuration out of the box.

**Note**: You need to be on the **Blaze plan** to deploy Analog applications with Firebase App Hosting.

Follow the [Getting Started instructions](https://firebase.google.com/docs/app-hosting/get-started#step-1:) to connect your GitHub repository to Firebase App Hosting.

## Firebase Hosting

Analog supports [Firebase Hosting](https://firebase.google.com/docs/hosting) with Cloud Functions and [Firebase App Hosting](https://firebase.google.com/docs/app-hosting) out of the box.

See a [Sample Repo](https://github.com/brandonroberts/analog-angular-firebase-example) with Firebase configured

**Note**: You need to be on the **Blaze plan** to use Analog with Cloud Functions.

If you don't already have a `firebase.json` in your root directory, Analog will create one the first time you run it. In this file, you will need to replace `<your_project_id>` with the ID of your Firebase project.

This file should then be committed to version control. You can also create a `.firebaserc` file if you don't want to manually pass your project ID to your `firebase` commands (with `--project <your_project_id>`):

```json [.firebaserc]
{
  "projects": {
    "default": "<your_project_id>"
  }
}
```

Then, just add Firebase dependencies to your project:

```bash
npm install -D firebase-admin firebase-functions firebase-functions-test
```

### Using Firebase CLI

If prefer to set up your project with the Firebase CLI, which will fetch your project ID for you, add required dependencies (see above) and even set up automated deployments with GitHub Actions.

#### Install Firebase CLI globally

```bash
npm install -g firebase-tools
```

**Note**: You need to be on [^11.18.0](https://github.com/firebase/firebase-tools/releases/tag/v11.18.0) to deploy a nodejs18 function.

#### Initialize your Firebase project

Login to Firebase and select the **Hosting** and **Functions** options as shown below:

```bash
firebase login
firebase init
 ◉ Functions: Configure a Cloud Functions directory and its files
 ◉ Hosting: Configure files for Firebase Hosting and (optionally) set up
GitHub Action deploys
```

Unless you have an existing Firebase project, select **Create a new project** to continue. Firebase will provision a new project and provide the URL to access the web console to manage it.

Once your project is created, select **TypeScript** as the language to use to write Cloud Functions. Proceed with accepting the default parameters by pressing _Enter._

When prompted for the **public directory**, enter `dist/analog/public`.

In the next step, take the default option, N, on whether to configure as a **single-page app.** This is important! **Do not** configure your project as a single-page app.

After setup completes, ensure that the following properties are configured correctly in your `firebase.json` file. This ensures server-side rendering will work correctly with Cloud Functions:

```json [firebase.json]
{
  "functions": {
    "source": "dist/analog/server"
  },
  "hosting": [
    {
      "site": "<your_project_id>",
      "public": "dist/analog/public",
      "cleanUrls": true,
      "rewrites": [
        {
          "source": "**",
          "function": "server"
        }
      ]
    }
  ]
}
```

You can find more details in the [Firebase documentation](https://firebase.google.com/docs/hosting/quickstart).

### Firebase functions

Ensure that you set up Firebase functions as described in the previous section. Next, you must [configure Nitro](overview) correctly for Firebase Cloud Functions to work.

In `vite.config.ts` update the `nitro` property with the configuration options that fit your needs, like your Node.js version and preferred region.

```js [vite.config.ts]
nitro: {
  preset: 'firebase',
  firebase: {
    nodeVersion: '20',
    gen: 2,
    httpsOptions: {
      region: 'us-east1',
      maxInstances: 100,
    },
  },
},
```

### Alternatively, multiple AnalogJS projects (/app1, /app2) in a single Firebase Hosting site

This leverages cloud run services to host AnalogJS projects and uses rewrite rules for forwarding traffic from firebase to cloud run.

[Deploying with a custom URL prefix](/docs/features/deployment/overview#deploying-with-a-custom-url-prefix).

```json [firebase.json]
{
  "hosting": [
    {
      "site": "<your_project_id>",
      "public": "public",
      "cleanUrls": true,
      "rewrites": [
        {
          "source": "/app1",
          "run": {
            "serviceId": "app1",
            "region": "us-central1",
            "pinTag": false
          }
        },
        {
          "source": "/app1/**",
          "run": {
            "serviceId": "app1",
            "region": "us-central1",
            "pinTag": false
          }
        }
      ]
    }
  ]
}
```

### Local preview

You can preview a local version of your site to test things out without deploying.

```bash
BUILD_PRESET=firebase npm run build
firebase emulators:start
```

### Deploy to Firebase Hosting using the CLI

To deploy to Firebase Hosting, run the `firebase deploy` command.

```bash
BUILD_PRESET=firebase npm run build
firebase deploy
```

### Firebase Warnings

When configuring or deploying Firebase you may see warnings like:

```
npm WARN EBADENGINE Unsupported engine {
npm WARN EBADENGINE   package: undefined,
npm WARN EBADENGINE   required: { node: '18' },
npm WARN EBADENGINE   current: { node: 'v20.11.0', npm: '10.2.4' }
npm WARN EBADENGINE }
```

```
 ⚠  functions: Couldn't find firebase-functions package in your source code. Have you run 'npm install'?
```

These are benign errors and can be ignored, so long as you make sure your environment configuration matches `Nitro`.

## Render.com

Analog supports deploying on [Render](https://render.com/) with minimal configuration.

### Web Service Deployment

1. [Create a new Web Service](https://dashboard.render.com/select-repo?type=web) and select the repository that contains your code.

2. Ensure the 'Node' environment is selected.

3. [Specify your Node version for Render to use](https://render.com/docs/node-version) (v18.13.0 or higher recommended) - Render by default uses Node 14, which fails to correctly build an Analog site

4. Depending on your package manager, set the build command to `yarn && yarn build`, `npm install && npm run build`, or `pnpm i --shamefully-hoist && pnpm build`.

5. Update the start command to `node dist/analog/server/index.mjs`

6. Click 'Advanced' and add an environment variable with `BUILD_PRESET` set to `render-com`.

7. Click 'Create Web Service'.

### Static Site Deployment

If using Analog to pre-render static content, you can deploy a static site on Render with minimal configuration

1. [Create a new Static Site](https://dashboard.render.com/select-repo?type=static) and select the repository that contains your code.

2. Depending on your package manager, set the build command to `yarn && yarn build`, `npm install && npm run build`, or `pnpm i --shamefully-hoist && pnpm build`..

3. Set the publish directory to the `public` directory inside of the `dist` build directory (e.g. `dist/analog/public`)

4. Click 'Create Static Site'

## Edgio

Analog supports deploying on [Edgio](https://edg.io) with minimal configuration.

1. Install the Edgio CLI:

```bash
npm i -g @edgio/cli
```

2. In your project's directory, initialize Edgio:

```bash
edgio init --connector=@edgio/analogjs
```

3. Deploy To Edgio

```bash
edgio deploy
```

## GitHub Pages (Static Site Deployment)

Analog supports deploying a static site on [GitHub Pages](https://pages.github.com/).
When deploying your site to GitHub Pages, you must add an empty file called `.nojekyll` in the root directory of the `gh-pages` branch.

You can automate the deployment using the [Analog Publish Github Pages](https://github.com/marketplace/actions/analog-publish-github-pages) action:

```yaml
name: Build and Deploy

on:
  push:
    branches:
      - 'main'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      - uses: k9n-dev/analog-publish-gh-pages@v1.0.0
        with:
          access-token: ${{ secrets.ACCESS_TOKEN }}
          # further options are available.
          # see: https://github.com/marketplace/actions/analog-publish-github-pages
```

Or you can do it by your own like this:

```yaml
name: Build Deploy

on:
  push:
    branches:
      - '*' # deploy on all branches (but a --dry-run flag is added for branches (see code below))

env:
  TARGET_DIR: dist/analog/public

jobs:
  # build the project and push it to the gh-pages branch
  build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20.x'
      - name: Set environment variable based on branch
        run: |
          if [[ $GITHUB_REF == refs/heads/main || $GITHUB_REF == refs/heads/master ]]; then
            echo "Branch is main or master. Setting DRY_RUN_OPTION to empty."
            echo "DRY_RUN_OPTION=" >> $GITHUB_ENV
          else
            echo "Branch is not main or master. Setting DRY_RUN_OPTION to '--dry-run'."
            echo "DRY_RUN_OPTION=--dry-run" >> $GITHUB_ENV
          fi
      - name: Install
        run: npm ci
      - name: Build
        run: npm run build
      - name: Deploy Website (gh-pages branch)
        env:
          GH_TOKEN: ${{ secrets.ACCESS_TOKEN }} # A token must be created to be able to deploy on the gh-pages branch
          CNAME_OPTION: --cname=yourdomain.dev # omit if your not running it on a custom domain
        # run the deployment script to push the built project to the gh-pages branch
        # the default contributor is github-actions[bot]
        run: |
          echo "DRY_RUN_OPTION=$DRY_RUN_OPTION"
          npx angular-cli-ghpages --no-silent --dir="${{env.TARGET_DIR}}" \
            --name="github-actions[bot]" \
            --email="github-actions[bot]@users.noreply.github.com" \
            --branch="gh-pages" \
            --message="Deploy: $(git log -1 --pretty=%B)" \
            $DRY_RUN_OPTION

  # deploy from gh-pages branch
  deploy-pages:
    needs: build-and-push
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout gh-pages
        uses: actions/checkout@v4
        with:
          ref: gh-pages

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: '.'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

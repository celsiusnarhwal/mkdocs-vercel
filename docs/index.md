# MkDocs + Vercel

I wasn't satisfied with any of the existing guides for deploying an MkDocs site to [Vercel](https://vercel.com/), so
I wrote my own.

This method uses [GitHub Actions](https://github.com/features/actions) to build and deploy your site via the [Vercel CLI](https://vercel.com/docs/cli).

## Prerequisites

- [:simple-vercel: A Vercel account](https://vercel.com/signup)
- [:fontawesome-brands-node-js: Node.js](https://nodejs.org)

## Configuring Your Project

First and foremost, you'll need to install the Vercel CLI on your own machine.

=== ":simple-npm: npm"

    ```bash
    npm i -g vercel
    ```

=== ":fontawesome-brands-yarn: Yarn"

    ```bash
    yarn global add vercel
    ```

=== ":simple-pnpm: pnpm"

    ```bash
    pnpm i -g vercel
    ```

Connect it to your Vercel account:

```bash
vercel login
```

Then switch into your project's directory and link it to Vercel:

```bash
vercel link
```

After following the interactive prompts, a `.vercel` folder will appear in your project's directory.
Inside that folder is a `project.json` file with a `projectId` property and an `orgId` property. Add the values of
these properties as secrets in your project's GitHub repository under the names `VERCEL_PROJECT_ID` and `VERCEL_ORG_ID`,
respectively.

You can do this on GitHub.com or with the GitHub CLI:

=== ":fontawesome-brands-github: GitHub CLI"

    ```bash
    gh secret set VERCEL_PROJECT_ID
    gh secret set VERCEL_ORG_ID # (1)!
    ```

    1. You'll be interactively prompted for the respective secret after running each command.

!!! tip "Using `vercel.json`"

    You can use a [`vercel.json`](https://vercel.com/docs/concepts/projects/project-configuration) file in your
    project by simply dropping it into the `docs` folder. MkDocs will
    automatically include it in the build output.

## Getting a Token

You'll need to generate a Vercel token so the Vercel CLI can access your account from within GitHub Actions. Head over
to your [Vercel account settings](https://vercel.com/account/tokens), choose a name, appropriate scope, and expiration
date for your token, and click **Create**. Copy the token and set it as a repository secret under the name
`VERCEL_TOKEN`.

## Writing the Workflow

Finally, you need to write a GitHub Actions workflow that builds your documentation and deploys it to Vercel.
You have practically infinite flexibility in how this workflow looks, but it should probably contain the following
at a minimum:

```yaml
--8<--
docs/code/example.yml
--8<--
```

1.  To create a production deployment, tack on the `--prod` flag:

    ```bash
    npx vercel --yes --cwd site --token ${{ secrets.VERCEL_TOKEN }} --prod
    ```

    To emulate Vercel's branch-dependent creation of preview and production deployments, you can do something like this
    (replacing `main` with your production branch if necessary):

    ```bash
    npx vercel --yes --cwd site --token ${{ secrets.VERCEL_TOKEN }} ${{ github.ref == 'refs/heads/main' && '--prod'||'' }}
    ```

That's it. Your site will now be deployed to Vercel on every push that changes the contents of the `docs` folder,
`mkdocs.yml`, or anything you choose to specify in `on.push.paths`.

You can check out a fully-functioning example workflow at [this website's repository](https://github.com/celsiusnarhwal/mkdocs-vercel/blob/main/.github/workflows/docs.yml).

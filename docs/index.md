# MkDocs + Vercel

I wasn't satisfied with any of the existing guides for deploying an MkDocs site to [Vercel](https://vercel.com/), so
I wrote my own.

This method uses [GitHub Actions](https://github.com/features/actions) to build and deploy your site via the [Vercel CLI](https://vercel.com/docs/cli).

## Prerequisites

- [:simple-vercel: A Vercel account](https://vercel.com/signup)
- [:fontawesome-brands-node-js: Node.js](https://nodejs.org)

## Configuring your project

First and foremost, you'll need to install the Vercel CLI on your own machine.

=== ":simple-npm: npm"

    ```shell
    npm i -g vercel
    ```

=== ":fontawesome-brands-yarn: Yarn"

    ```shell
    yarn global add vercel
    ```

=== ":simple-pnpm: pnpm"

    ```shell
    pnpm i -g vercel
    ```

=== ":simple-bun: Bun"

    ```shell
    bun add -g vercel
    ```

Connect it to your Vercel account:

```shell
vercel login
```

Then switch into your project's directory and link it to Vercel:

```shell
vercel link
```

After following the interactive prompts, a `.vercel` folder will appear in your project's directory.
Inside that folder is a `project.json` file with a `projectId` property and an `orgId` property. Add the values of
these properties as secrets in your project's GitHub repository under the names `VERCEL_PROJECT_ID` and `VERCEL_ORG_ID`,
respectively.

You can do this [on GitHub.com](https://docs.github.com/en/actions/security-for-github-actions/security-guides/using-secrets-in-github-actions) 
or with the [GitHub CLI](https://cli.github.com):

=== ":fontawesome-brands-github: GitHub CLI"

    ```shell
    gh secret set VERCEL_PROJECT_ID
    gh secret set VERCEL_ORG_ID # (1)!
    ```

    1. You'll be interactively prompted for the respective secret after running each command.

## Getting a token

You'll need to generate a Vercel account token so the Vercel CLI can access your account from within GitHub Actions. Head over
to your [Vercel account settings](https://vercel.com/account/tokens), choose a name, appropriate scope, and expiration
date for your token, and click **Create**. Copy the token and set it as a repository secret under the name
`VERCEL_TOKEN`.

## Configuring `vercel.json`

It's optional, but strongly recommended, that you put a [`vercel.json`](https://vercel.com/docs/projects/project-configuration)
file in your `docs` folder with at least the following contents:

=== ":octicons-file-code-16: docs/vercel.json"

    ```json
    {
      "trailingSlash": true
    }
    ```

??? question "Why?"
    Relative internal links on your site will break if the request URL 
    [does not have a trailing slash](https://github.com/slorber/trailing-slash-guide). This isn't a problem on MkDocs' 
    _de facto_ standard hosting platform, [GitHub Pages](https://pages.github.com), which automatically enforces trailing slashes, so the 
    overwhelming majority of public MkDocs sites do not exhibit this issue. Yours will,
    though, if you deploy it on Vercel without configuring `vercel.json` to enforce trailing slashes.

## Writing the workflow

Finally, you need to write a GitHub Actions workflow that builds your documentation and deploys it to Vercel.
There's no one-size-fits-all solution here, but here are some basic examples for commonly-used package managers.


=== ":fontawesome-brands-python: pip"

    === ":octicons-file-code-16: `.github/workflows/docs.yml`"
        ```yaml
        --8<--
        docs/.snippets/pip.yml
        --8<--
        ```
    
        1.  To create a production deployment, add the `--prod` flag:
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} --prod
            ```
        
            To emulate Vercel's branch-dependent creation of preview and production deployments, you can do something like this
            (replacing `main` with your production branch if necessary):
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} ${{ github.ref_name == 'main' && '--prod' ||'' }}
            ```

        2.  If you don't want the Vercel CLI to send [telemetry](https://vercel.com/docs/cli/about-telemetry),
            you can add `#!yaml VERCEL_DISABLE_TELEMETRY: 1` to the `#!yaml env:` key:
            
            ```yaml
            env:
              VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
              VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
              VERCEL_DISABLE_TELEMETRY: 1
            ```

=== ":simple-uv: uv"

    === ":octicons-file-code-16: `.github/workflows/docs.yml`"
        ```yaml
        --8<--
        docs/.snippets/uv.yml
        --8<--
        ```
    
        1.  To create a production deployment, add the `--prod` flag:
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} --prod
            ```
        
            To emulate Vercel's branch-dependent creation of preview and production deployments, you can do something like this
            (replacing `main` with your production branch if necessary):
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} ${{ github.ref_name == 'main' && '--prod' ||'' }}
            ```

        2. If you have a seperate dependency group for your documentation, you might instead do something like:
            
            ```{ .shell .no-select }
            uv sync --only group docs
            ```

        3.  If you don't want the Vercel CLI to send [telemetry](https://vercel.com/docs/cli/about-telemetry),
            you can add `#!yaml VERCEL_DISABLE_TELEMETRY: 1` to the `#!yaml env:` key:
            
            ```yaml
            env:
              VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
              VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
              VERCEL_DISABLE_TELEMETRY: 1
            ```

=== ":simple-poetry: Poetry"

    === ":octicons-file-code-16: `.github/workflows/docs.yml`"
        ```yaml
        --8<--
        docs/.snippets/poetry.yml
        --8<--
        ```
    
        1.  To create a production deployment, add the `--prod` flag:
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} --prod
            ```
        
            To emulate Vercel's branch-dependent creation of preview and production deployments, you can do something like this
            (replacing `main` with your production branch if necessary):
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} ${{ github.ref_name == 'main' && '--prod' ||'' }}
            ```

        2. If you have a seperate dependency group for your documentation, you might instead do something like:
            
            ```{ .shell .no-select }
            poetry install --only docs
            ```

        3.  If you don't want the Vercel CLI to send [telemetry](https://vercel.com/docs/cli/about-telemetry),
            you can add `#!yaml VERCEL_DISABLE_TELEMETRY: 1` to the `#!yaml env:` key:
            
            ```yaml
            env:
              VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
              VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
              VERCEL_DISABLE_TELEMETRY: 1
            ```


=== ":simple-pdm: PDM"

    === ":octicons-file-code-16: `.github/workflows/docs.yml`"
        ```yaml
        --8<--
        docs/.snippets/pdm.yml
        --8<--
        ```
    
        1.  To create a production deployment, add the `--prod` flag:
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} --prod
            ```
        
            To emulate Vercel's branch-dependent creation of preview and production deployments, you can do something like this
            (replacing `main` with your production branch if necessary):
        
            ```{ .shell .no-select }
            npx vercel --token ${{ secrets.VERCEL_TOKEN }} ${{ github.ref_name == 'main' && '--prod' ||'' }}
            ```

        2. If you have a seperate dependency group for your documentation, you might instead do something like:
            
            ```{ .shell .no-select }
            pdm sync --group docs
            ```

        3.  If you don't want the Vercel CLI to send [telemetry](https://vercel.com/docs/cli/about-telemetry),
            you can add `#!yaml VERCEL_DISABLE_TELEMETRY: 1` to the `#!yaml env:` key:
            
            ```yaml
            env:
              VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
              VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}
              VERCEL_DISABLE_TELEMETRY: 1
            ```

You can also check out the [workflow that deploys this very website](https://github.com/celsiusnarhwal/mkdocs-vercel/blob/main/.github/workflows/docs.yml).

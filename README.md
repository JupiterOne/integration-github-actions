# Github actions for integration repositories
This repository consolidates the github actions used by the JupiterOne integration repositories.

## How to use
- Create a workflow file inside the `/.github` in the repo you want to add the new action.
- Add the action you want from this repo, e.g:
```yaml
  - name: Sample step
    uses: JupiterOne/integration-github-actions/action-test@BRANCH-OR-TAG
```
Note `@BRANCH-OR-TAG` can be optional, that's useful just for testing.

## How to contribute
In order to create a new action, just create a new folder with the name of the action and inside of it, add `action.yml` describing the workflow.
If a script needs to be executed, add the script inside the `src/` folder but in the workflow file, reference the transpiled file in the lib folder.

## Available actions
### Action test
Just a github action that you can use to test in your repo. This action just say Hello. This is an example of how to implement an action using a `.ts` file.
#### Example Usage:
```yaml
- name: Test action-test
  uses: JupiterOne/integration-github-actions/action-test@INT-6300
  with:
    who-to-greet: 'David'
```
### Bump integration deployment version
This GitHub Action can be added to graph-* repos to automatically create pull requests in the integration-deployments repository with the new version of the graph.

##### Inputs

| Id               | Description                                                                          | Required |
| ---------------- | ------------------------------------------------------------------------------------ | -------- |
| integrationName  | Integration name as it appears in the integration-deployments repo                   | Yes      |
| graphProjectName | The name of the graph project. Default: `graph-${ integrationName }`.                | No       |
| version          | New version of the graph npm package                                                 | No       |
| releaseNotes     | Release notes to include in pull request                                             | No       |
| githubToken      | GITHUB_TOKEN or a `repo` scoped Personal Access Token (PAT). Default: `github.token` | No       |
| npmAuthToken     | NPM_AUTH_TOKEN to install JupiterOne dependencies                                    | Yes      |
| mainBranch       | Base branch to pull changes from integration-deployments. Default: `main`            | No       |

##### Outputs

| Id               | Description                     |
| ---------------- | ------------------------------- |
| pull-request-url | URL of the created pull request |

##### Example Usage:
```yaml
  - name: Bump integration deployment version
    uses: JupiterOne/integration-github-actions/create-integration-deployment@v1.0.0
    id: create-version-pr
    with:
      integrationName:
        ${{ steps.get-integration-name.outputs.integrationName }}
      version: ${{ steps.get-version-number.outputs.versionNumber }}
      githubToken: ${{ secrets.AUTO_GITHUB_PAT_TOKEN }}
      npmAuthToken: ${{ secrets.NPM_AUTH_TOKEN }}
  - name: Print URL
    shell: bash
    run: echo "${{ steps.create-version-pr.outputs.pull-request-url }}"
```



#### Troubleshooting `create-integration-deplyment`

The `JupiterOne/integration-github-actions/create-integration-deployment` action is often used in conjunction with the `jupiterone/action-npm-build-releas` action. These actions require a number of configurations to ensure they can succeed. These include:

1. Access to the `NPM_AUTH_TOKEN` org secret
2. Access to the `AUTO_GITHUB_PAT_TOKEN` org secret
3. Permission for the `j1-internal-automation` github user to override branch protection rules
4. Configured without any required status checks

If your workflows are failing, start by running the following queries in j1.apps.us.jupiterone.io to find out if your repository is misconfigured.

##### Repos that don’t have the NPM_AUTH_TOKEN secret
```
FIND 
  github_repo WITH name ^= 'graph-' AND archived != true as r
  THAT !USES github_org_secret WITH name = 'NPM_AUTH_TOKEN'
RETURN r.name
ORDER BY r.name ASC
```

##### Repos that don’t have the AUTO_GITHUB_PAT_TOKEN secret
```
FIND 
  github_repo WITH name ^= 'graph-' AND archived != true as r
  THAT !USES github_org_secret WITH name = 'AUTO_GITHUB_PAT_TOKEN'
RETURN r.name
ORDER BY r.name ASC
```

##### Repos that don’t allow j1-internal-automation to bypass branch protection
```
FIND 
  github_repo WITH name ^= 'graph-' AND archived != true as r
  THAT !HAS >> github_branch_protection_rule 
  THAT OVERRIDES << github_user WITH name = 'j1-internal-automation' 
RETURN r.name
ORDER BY r.name ASC
```

##### Repos that have required status checks
```
FIND 
  github_repo WITH name ^= 'graph-' AND archived != true as r
  THAT HAS github_branch_protection_rule WITH requiredStatusChecks != undefined
RETURN r.name
ORDER BY r.name ASC
```

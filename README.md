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

##### Example Usage:
```yaml
  - name: Bump integration deployment version
    uses: JupiterOne/integration-github-actions/create-integration-deployment@v1.0.0
    with:
      integrationName:
        ${{ steps.get-integration-name.outputs.integrationName }}
      version: ${{ steps.get-version-number.outputs.versionNumber }}
      githubToken: ${{ secrets.AUTO_GITHUB_PAT_TOKEN }}
      npmAuthToken: ${{ secrets.NPM_AUTH_TOKEN }}
```

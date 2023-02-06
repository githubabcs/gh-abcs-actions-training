# 3 - Environments and Secrets
In this lab you will use environments and secrets.
> Duration: 10-15 minutes

References:
- [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
- [Encrypted secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)
- [Accessing your secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets#accessing-your-secrets)

## 3.1 Create new encrypted secrets

1. Follow the guide to create a new environment called `UAT`
    - [Creating an environment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#creating-an-environment)
    - Add a required reviewer for your environment
    - [Required reviewers](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#required-reviewers)
2. Follow the guide to create a new repository secret called `MY_REPO_SECRET`
    - [Creating encrypted secrets for a repository](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-a-repository)
3. Follow the guide to create a new env secret called `MY_ENV_SECRET`
    - [Creating encrypted secrets for an environment](https://docs.github.com/en/actions/security-guides/encrypted-secrets#creating-encrypted-secrets-for-an-environment)
4. Open the workflow file [environments-secrets.yml](/.github/workflows/environments-secrets.yml)
5. Edit the file and copy the following YAML content as first job (after the `jobs:` line):
```YAML

  use-secrets:
    name: Use secrets
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    steps:
      - name: Hello world action with secrets
        uses: actions/hello-world-javascript-action@v1
        with: # Set the secret as an input
          who-to-greet: ${{ secrets.MY_REPO_SECRET }}
        env: # Or as an environment variable
          super_secret: ${{ secrets.MY_REPO_SECRET }}
      - name: Echo secret is redacted in the logs
        run: |
          echo Env secret is ${{ secrets.MY_REPO_SECRET }}
          echo Warning: GitHub automatically redacts secrets printed to the log, 
          echo          but you should avoid printing secrets to the log intentionally.
          echo ${{ secrets.MY_REPO_SECRET }} | sed 's/./& /g'
```
6. Update the workflow to run on push and pull_request events
```YAML
on:
  push:
     branches: [main]
  pull_request:
     branches: [main]
  workflow_dispatch:    
```
7. Commit the changes into the `main` branch
8. Go to `Actions` and see the details of your running workflow


## 3.2 Add a new workflow job to deploy to UAT environment

1. Open the workflow file [environments-secrets.yml](/.github/workflows/environments-secrets.yml)
2. Edit the file and copy the following YAML content between the test and prod jobs (before the `use-environment-prod:` line):
```YAML

  use-environment-uat:
    name: Use UAT environment
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    needs: use-environment-test

    environment:
      name: UAT
      url: 'https://uat.github.com'
    
    steps:
      - name: Step that uses the UAT environment
        run: echo "Deployment to UAT..."
        env: 
          env_secret: ${{ secrets.MY_ENV_SECRET }}

```
7. Change the PROD job to depend on UAT deployment
```YAML
    needs: use-environment-uat
```
8. Commit the changes into the `main` branch
9. Go to `Actions` and see the details of your running workflow
10. Review your deployment and approve the pending UAT job
    - [Reviewing deployments](https://docs.github.com/en/actions/managing-workflow-runs/reviewing-deployments)
11. Go to `Settings` > `Environments` and update the `PROD` environment created to protect it with approvals (same as UAT)
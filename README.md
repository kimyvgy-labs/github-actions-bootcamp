# GitHub Actions Bootcamp

This is the sample app for GitHub Actions Presentation.

## WARNING

- Please fork this repository and practice on it.
- You must enable GitHub Actions on the forked-repository.
- You just click `I understand my workflows...` button. If not, the Workflows may not run.

![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/72750afe-9aca-45b1-8d9b-d7a4b5a8d2fa)

## Lab 1: Hello, World!

There is an workflow located at `.github/workflows/hello-world.yaml`.

Let's add a new step into the Job named `hello`. It should execute the following command:

```bash
cat requirements.txt
```

1. In the `main` branch, open `.github/workflows/hello-world.yaml` then click the edit button:
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/8f4a3d45-5965-4fae-aa0f-38df2894b6c4)
2. Add new step into the job `hello`:
```yaml
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: echo "Hello, World!"
      - run: cat requirements.txt
```
3. Commit to forked-repository
4. Next, open `Actions` tab in GitHub. You will see the workflow `hello-world`.
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/0d8edab8-0e0e-4883-83a7-1b12dc1f7ee0)
5. Finally, make sure that you see the new step completed successfully:
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/90eb9d7e-26bf-431a-8ea2-248f8a9dcf27)

## Lab 2: Django CI

Now, let's create a new workflow `django.yaml` and perform Continuous Integration (CI) for the app.

1. Open your repository page in GitHub
2. Click on `Actions` tab
3. Click on `New workflow` button in the left column
4. Next, search workflow templates using keyword `Django`
5. In the search results, you will see the Django workflow template. Click on `Configure` button.
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/a2b4576a-a752-4615-a567-351d97d86884)
6. Now, set the python-version to v3.10:
```yaml
      matrix:
        python-version: ["3.10"]
```
7. Commit and make sure that the workflow is running

## Lab 3: Secrets

In this lab, the `hello` job should print `Hello, $NAME!` instead of `Hello, World!`.

The `$NAME` is an user-defined repository secret variable that will be injected by GitHub Actions.

1. Open `Settings` page
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/c83389be-c6b4-47ea-a33e-687be05fcfa9)
2. Open `Secrets and variables` > `Actions`
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/76cd7f74-1c7b-4dcc-a6b4-cbf24aef8f9a)
3. Add your repository secrets
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/a86a55b8-5351-4b04-b2a8-68d738d6cf96)
4. Change the step:
```yaml
jobs:
  hello:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - env:
          NAME: ${{ secrets.NAME }}
        run: echo "Hello, $NAME!"
```
5. Check your workflow logs:
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/14b58bfc-fe3d-4b9e-9b9c-5ac8a1395b2e)

## Lab 4: Docker build & push

There is a `Dockerfile` file in this repository. It will be used to build the Docker image for this app.

Let's create a new workflow for automate-building the Docker images in GitHub Actions and push them to GitHub Container Registry.

Requirements:
- The workflow should only trigger when having the `push` and `pull_request` event on the `main` branch only.
- The workflow shouldn't push the images to the registry server during pull request.
- The workflow should push the image with tags: `latest` and `build-${{ vars.GITHUB_RUN_NUMBER }}`.

1. We use an Action [Build and push Docker images](https://github.com/marketplace/actions/build-and-push-docker-images) in [GitHub Marketplace](https://github.com/marketplace)
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/eada567a-8f19-4616-9f66-0e3ff0c73bcb)
2. Scrolling down to the `Git context` section and copy the sample workflow YAML
![image](https://github.com/devsuccess101/github-actions-bootcamp/assets/13513658/c9dd29af-7d50-4e17-bc28-0e024a4b79fb)
3. Prepare your Docker repository secret: `REGISTRY_SERVER=ghcr.io`
4. Create new workflow `docker.yaml` file, paste the sample workflow.
5. First, let's modify the event triggers. In this case, we want to trigger this workflow when having new PR/push to the `main` branch (only `main` branch).
```yaml
on:
  push:
    branches:
      - 'main'
  pull_request:
    branches:
      - 'main'
```
6. Next, make the login step use the registry server by reading Secrets:
```yaml
      - name: Login to the container registry server
        uses: docker/login-action@v3
        with:
          registry: ${{ secrets.REGISTRY_SERVER }}   # User-defined repository secret variable. Value: ghcr.io
          username: ${{ github.repository_owner }}   # Pre-defined variable by GitHub Actions
          password: ${{ secrets.GITHUB_TOKEN }}      # Pre-defined variable by GitHub Actions
```
7. Add some args for the `Build and push` step
```yaml
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          push: ${{ github.event_name == 'push' }}   # Only push Docker images to the registry server on the push event 
          tags: |-
            ${{ secrets.REGISTRY_SERVER }}/${{ github.repository }}:latest
            ${{ secrets.REGISTRY_SERVER }}/${{ github.repository }}:build-${{ vars.GITHUB_RUN_NUMBER }}
```
8. Check the workflow logs

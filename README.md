# CICD Workshop

Welcome to the CICD Workshop! In this workshop you will build your own CI/CD pipeline to deploy a Flutter web app to Google Cloud Run using GitHub Actions.

The Flutter app template was inspired by: https://github.com/flutter/games/tree/main/templates/card

## What you will build

By the end of this workshop you will have:

- A **Continuous Integration (CI)** pipeline that builds, lints, and tests your Flutter app on every pull request.
- A **Continuous Deployment (CD)** pipeline that builds a Docker image and deploys it to Google Cloud Run on every merge to `main`.

## Initial Setup

Before we start, let's get your repository ready:

1. Create your own repository in GitHub.
2. Ensure that GitHub Actions is enabled on the `Actions` tab.
3. Create a branch protection rule to prevent direct commits to `main`.
4. Copy all the contents of this repo into your newly created repository. Make sure to include **all folders and files**, including hidden ones (those starting with `.`, such as `.github`). Download or clone this repo locally, then copy everything into your own repo folder:
   ```bash
   # Copy all files (including hidden) from this repo into your own repo folder
   cp -r ./cicd-workshop-fdc/* my-repo/
   ```
5. Create a new branch, make a small change, and open a pull request against `main` **on your new repo**:
   ```bash
   git checkout -b my-first-branch
   # make any small change, e.g. edit this README
   git add .
   git commit -m "my first commit"
   git push --set-upstream origin my-first-branch
   ```
   Then open the pull request on GitHub.
6. Verify that the CI step is now running.

## Continuous Integration (CI)

The CI pipeline is already provided in `.github/workflows/ci.yml`. It runs automatically on every pull request and executes the following steps:

1. `flutter pub get` — fetches and installs the project dependencies.
2. `flutter build web` — compiles the Flutter project into a web application.
3. `dart format --output=none --set-exit-if-changed .` — checks that all code follows the Dart style guide.
4. `flutter analyze` — analyzes the code for potential issues and best practice violations.
5. `flutter test` — runs the unit and widget tests.

Open the `.github/workflows/ci.yml` file, read through it, and make sure you understand each step. Then go back to the pull request you opened in the setup and verify the CI workflow ran successfully.

## Continuous Deployment (CD)

Congratulations on getting the CI running! Now let's deploy the app to Google Cloud Run.

### Authentication

Authentication to GCP is already set up in the workflow using **Workload Identity Federation (WIF)**. Instead of storing long-lived service account keys, GitHub Actions exchanges a short-lived OIDC token with GCP at runtime — no static credentials needed. The WIF pool, provider, and service account bindings have already been configured on the GCP side.

### What you need to implement

Open `.github/workflows/ci.yml` and complete the two TODO steps:

**1. Build and push the Docker image**

Add the following commands to the `Build and Push Docker Image` step (after the `gcloud auth configure-docker` lines):

```bash
docker build -t europe-west3-docker.pkg.dev/project-ef801521-090f-4fa8-9b5/cicd-workshop/{FLUTTER_APP_NAME}:{VERSION} .
docker push europe-west3-docker.pkg.dev/project-ef801521-090f-4fa8-9b5/cicd-workshop/{FLUTTER_APP_NAME}:{VERSION}
```

- `FLUTTER_APP_NAME`: use `flutter-app-<your-cicd-id>`
- `VERSION`: start with `v1.0.0`

**2. Deploy to Cloud Run**

Add the following command to the `Deploy to Cloud Run` step:

```bash
gcloud run deploy {GOOGLE_CLOUD_RUN_SERVICE} \
  --image {IMAGE_TAG} \
  --region europe-west3 \
  --allow-unauthenticated
```

- `GOOGLE_CLOUD_RUN_SERVICE`: your `cicd-id`
- `IMAGE_TAG`: the full image tag you used in step 1

## You're live! 🎉

Your app is now running on Google Cloud Run. Every time you merge a pull request to `main`, the CD pipeline will automatically build a new Docker image and deploy it — your changes go live without any manual steps.

From here you can start implementing new features:

1. Create a new branch.
2. Make your changes.
3. Open a pull request — the CI will run automatically to validate your code.
4. Merge it — the CD will deploy the new version to Cloud Run.

You can also manage versions of your app by changing the Docker image tag (e.g. `v1.1.0`, `v2.0.0`) each time you deploy. This lets you track exactly what is running in production and roll back to a previous image at any time if something goes wrong.

That's the power of CI/CD: ship faster, with confidence 🚀.


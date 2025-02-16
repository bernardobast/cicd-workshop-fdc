# cicd-workshop-fdc# CICD Workshop fdc

Welcome to the CICD Workshop here you will build your own CICD to deploy a Flutter App. The initial template for the flutter App was inspired from the following repo: https://github.com/flutter/games/tree/main/templates/card

# Inital Setup:

By the end of this workshop, you will have your own CICD running! The first step consists of creating your own repo. Please follow the following steps: 
1. Create your own repository in GitHub. 
2. Ensure that GitHub Actions is enabled on the `Actions` tab.
3. Create branch protection rules to avoid commit directly to main. 
4. Copy this repo to your newly created repository. 
5. Create a new branch and a pull request against main **on your new repo**. 
6. Verify that the CI step is now running :)

# Let's buid a Continuous Integration (CI):

In this part of the workshop, we will be building a Continuous Integration (CI) system. The CI plays a crucial role in code development as it acts as the first line of defense in the pipeline. Here, we will build, lint, and test our code to ensure that our changes can be safely merged into the main branch.

We will run the following commands in our CI pipeline:

1. The `flutter pub get` command fetches and installs the dependencies listed in your Flutter project's `pubspec.yaml` file.
2. The `flutter build web` command compiles your Flutter project into a web application by generating static files that can be hosted on a web server.
3. The `dart format --output=none --set-exit-if-changed .` command formats Dart code files according to the official Dart style guide.
4. The `flutter analyze` command analyzes the code for potential issues and enforces Dart best practices and coding standards.
5. The `flutter test` command runs unit and widget tests in a Flutter project.

Follow the existing steps carefully, commit your changes, and watch the CI work!

# Let's build a Continuous Deployment (CD)

Congratulations! You have successfully merged your first pull request (PR). Now, let's deploy it to Google Cloud Run. For simplicity, some steps involved in setting up the Google Cloud Project have already been completed. If you would like to set up a project for yourself, please refer to the "Setting up GCP Project" section.

The first step in the Continuous Deployment (CD) process is authentication. You need to provide GitHub with credentials that will allow you to access your GCP project. Follow these steps:

1. Go to Settings > Secrets and Variables > Actions > Repository Secrets and add a secret named `GCP_SA_KEY` with the provided credentials.
2. Add a secret named `GCP_PROJECT_ID` using the provided GCP project ID.
3. Add a secret named `GCP_ARTIFACT_REGISTRY` using the provided artifact registry.

Now that all the necessary information is correctly added to the secret environment variables, you can start implementing your CD pipeline. Follow these steps:

1. Uncomment the steps to authenticate to GCP and Docker registries.
2. Repeat the same steps as in the Continuous Integration (CI) process: build, lint, and test your Flutter app before starting the deployment phase.
3. Create a step to build and push the Docker image. The Dockerfile is already provided in the root folder of the project—please read it carefully and ask any questions you may have. The tag to be used is provided below.
4. Create a step to deploy to Google Cloud Run, as outlined below.

For step 3, here is the docker tag you should use:
- `FLUTTER_APP_NAME`: replace with your flutter-app-`cicd-id`.
- `VERSION`: replace with the version of the app you are creating. Start with `v1.0.0`. 
```
europe-west3-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/${{ secrets.GCP_ARTIFACT_REGISTRY }}/{FLUTTER_APP_NAME}:{VERSION}
```
For step 4, this is how you should deploy your app to cloud run:
- `GOOGLE_CLOUD_RUN_SERVICE`: your `cicd-id`
- `IMAGE_TAG`: The image tag you just created on step 3.
```
gcloud run deploy {GOOGLE_CLOUD_RUN_SERVICE} \
 --image {IMAGE_TAG} \
 --region us-central1
```

## Do you have extra time? 🤔

Now that you have both CI and CD running didnt you notice that you have repeated code to build, lint and test the Flutter app? Let's change that! Create your own `build-lint-test.yml` GitHub workflow and call it from the `ci.yml` pipeline. Here is an example:

```
name: Dummy Reusable Workflow

on:
  workflow_call:
    inputs:
      message:
        description: "A message to display"
        required: true
        type: string

jobs:
  echo-message:
    runs-on: ubuntu-latest
    steps:
      - name: Display the message
        run: echo "Message from caller: ${{ inputs.message }}"
```
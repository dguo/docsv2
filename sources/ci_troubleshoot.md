page_title: Shippable FAQ
page_description: Commonly asked questions that will help with troubleshooting
page_keywords: concepts, documentation, shippable, CI/CD


#Troubleshooting Errors

This document helps in troubleshooting errors generated on the Shippable platform while running Continuous Integration. The document is divided into two parts: 

1. Setup: Troubleshooting errors that occur during initial setup and prior to initiating a CI build. 
2. Continuous Integration (CI): Troubleshooting errors that occur during the CI process and is shown in the Console output.

For non-errors and questions, refer our [FAQ section](ci_faq.md).

---

## Setup
### Owner not found
When enabling a project on Bitbucket, the following error is generated:

```
(Id: 2002) Owner not found for projectId::5762d2b72x8192902x23xxx2
warn: caller:5762x2x501346x0x00959x0x|projects|enableById:5762x2x72x8192902x23xfx2|_getValidOwner
```
Reason: Admin permissions on the repository are required to enable it as a project on Shippable's platform. This ensures Shippable can add a webhook to the repository and listen for commits and pull requests to trigger builds.

**How to avoid:** Ensure a user with Admin permissions [enables the project](ci_subscriptions/#enabling-a-project) on Shippable's platform. If you still get the error, then the permissions may be out of date and need to be udpated. Synchronize your permissions by clicking on the Account settings (gear icon in the top navigation bar) and clicking the `Sync` button.

---

## Continuous Integration
### failed to find yml file
```
- Alerts
  - Errors
    - failed to find yml file
```
Reason: All build configuration on Shippable happens through the `shippable.yml` file present at the root of your source contorl repository. If this file is missing, we don't know how to run your build and you see the error.

**How to avoid:** For any repository you enable on Shippable, create a `shippable.yml` file at the root of your repo in your source control. At a minimum, include the language used in your repo, the version used & commands for tests that you are running. This example below shows a basic shippable.yml file that uses Node.js v5.3 and runs an npm test:

``` 
language: node_js

node_js:
  - 5.3

build:
  ci:
    - npm install
    - npm test
```
Shippable builds all branches that have a shippable.yml at the root, unless they are explicitly excluded through the yml configuration.

The shippable.yml reference guide is the best resource to learn what's possible with Shippable and explore the full capabilities supported on the platform.

---


### Integration name specified in yml does not match
```
- Alerts
  - Errors
    - Integration name specified in yml does not match integrations present in project settings: null
```
Reason: Notification and Hub integrations need to be set in two places - In the UI and in the `shippable.yml` file. You'll get this error if the name for the integration does not match in both places. 

**How to avoid:** Ensure the integration name are exactly the same in both `shippable.yml` and the integration in the UI. Read our [documentation on enabling integrations](ci_projects/#enabling-integrations) for more details.

---

### common2|_cleanRunYml|callerId:
```
- Alerts
  - Errors
    - common2|_cleanRunYml|callerId:!xxxxxxxxx prjectId: yyyyy
```
Reason: The language configured in the `shippable.yml` file should have the correct syntax in order to be recognized

**How to avoid:** Ensure the correct syntax is used when specifying a language in the `shippable.yml`. All supported langagues and configuration syntax is [available here](ci_configure/#specifying-language-and-runtime).

---

### invalid yml format
```
- Alerts
  - Errors
    - invalid yml format
```
Reason: This error is shown when the overall structure and syntax of the `shippable.yml` file needs to be fixed

**How to avoid:** Use online tools such as [YAML Lint](http://www.yamllint.com/) or [YAML Online Parser](http://yaml-online-parser.appspot.com/) to check the overall structure and syntax of the `shippable.yml` file

---

### bad YML data
```
- Alerts
  - Errors
    - Bad YML data in build.ci. Only strings allowed
```
Reason: This error is shown when a particular line or section within the `shippable.yml` file needs to be fixed.

**How to avoid:** Refer [our documentation](ci_configure/#shippableyml-structure)on the `shippable.yml` file for the syntax of a specific section called out in the error.

---


### Permission denied (publickey)
Build fails with the following error in the console:

```
- git_sync
  - ssh-agent bash -c 'ssh-add /tmp/ssh/01_deploy; git clone ssh://git@bitbucket.org/..........'
    Permission denied (publickey).
    fatal: Could not read from remote repository.
    Please make sure you have the correct access rights and the repository exists.
```
Reason: The webhook to the source control system needs to be reset. 

**How to avoid:** Follow the steps below:
- Click on your project from the Shippable dashboard
- Click the `Settings` tab
- Scroll all the way down and click the `Reset` button under the 'Reset' section
- Click `Confirm`. 

The reset action will do the following things:
1. Reset the webhook for Shippable
2. Generate a new deploy key and update the repository

If you are using encrypted variables for this project, they'll need to be re-encrypted. Integrations and other settings will not be affected. 

---

### Host key verification failed
Build fails with the following error in the console:

```
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

Reason: There are two reasons why you get this error:

1. You are using a custom image & the $HOME environment variable is not set. When we update your ~/.ssh/config file to insert `StrictHostKeyChecking` to `no`, it throws up the error.
2. If you are using an image from **shippableimages** repo. We have deprecated support for those images and our official images are now in the [drydock repo on Docker Hub](https://hub.docker.com/u/drydock/).

How to avoid: For the reasons above, follow these steps:

1. To fix this error, you can either update your Dockerfile to set the $HOME environment variable, or you can set it in the `pre_ci_boot` section in the options tag within `shippable.yml` as shown:

    ```
build:
  pre_ci_boot:
    image_name: your/image
    image_tag: your_tag
    options: "-e HOME=/root"
	```

2. Start using one of the new, official images to avoid running into this error. Switching to the new yml format automatically selects one of the new, official images for your build, by default. The naming convention for these images is [explained here](http://docs.shippable.com/ci_configure/#setting-your-build-image). 

For more information, view the [migration guide](http://blog.shippable.com/migrating-to-the-new-build-platform) and the [Top 5 tips for a successful migration](http://blog.shippable.com/5-tips-for-a-successful-migration).

---

### gcloud: command not found
Build fails with the following error:

```
gcr_login
   - cd /tmp && gcloud -q auth activate-service-account --key-file key.json
    /root/xxx.sh: line 58: gcloud: command not found 

```

Reason: When you are using a custom image for your CI & you have specified a Google Container Registry (GCR) in the hub integration and in your `shippable.yml`, we try to log into GCR from inside the CI build container. Hence you would need the gcloud SDK (or awscli for ECR integrations) installed inside your custom image. 

**How to avoid:** To avoid this, set the `agent_only: true` in the `shippable.yml` as shown below: 

```
integrations:
  hub:
    - integrationName: gcr_int_name
      type: gcr
      agent_only: true
```
If you would like to push your image to GCR, then use the `push` section after the `build: ci` section, which ensures the image is pushed after the CI is completed and this command is run outside the build container. 

Refer [our documentation on this topic](http://docs.shippable.com/ci_configure/#gcrecr-and-custom-images) for more details.

---

### Error: Allowed memory size exhausted

Build fails as it runs out of memory while installing PHP or Drupal, etc. with the following error:

```
Error: Allowed memory size of 134217728 bytes exhausted (tried to
allocate 913408 bytes)
```

Reason: The default memory_limit in the php.ini has been reached. 

**How to avoid:** You can add/change the settings of the php.ini file located at `~/.phpenv/versions/$(phpenv version-name)/etc/php.ini`. Add the following to the `ci` section of the `shippable.yml`:

```
- echo "memory_limit = 256M" >> $HOME/.phpenv/versions/$(phpenv version-name)/etc/php.ini
```

---

### no basic auth credentials
When pushing a Docker image to a Docker Registry (Docker Hub, Amazon ECR, Google Container Registry, etc.), build fails with the following error:

```
- docker push 604622019445.dkr.ecr.us-east-1.amazonaws.com/xxxxxx/node:latest 0s  
The push refers to a repository [xxxxxx.dkr.ecr.us-east-1.amazonaws.com/xxxxx/node] (len: 1)
25ce9d9cdec1: Preparing
Post https://xxxxxx.dkr.ecr.us-east-1.amazonaws.com/v2/xxxxx/node/blobs/uploads/: no basic auth credentials
```
Reason: The Hub integration needs to be configured correctly. 

**How to avoid:** Check the following settings to ensure the Hub integration has been configured correctly.

1. Ensure the Hub integration has been correctly set in the 'Account' settings based on the [Docker Registry used](int_docker_registries/) 
2. Check the 'Project' Settings to ensure the above integration is listed under the 'Hub Integration'. [Read instructions on setting it up](ci_projects/#enabling-integrations), if it is not.
3. Ensure it is listed in the `shippable.yml` file under the `integration` section and the `integrationName` is exactly the same as the one specified in the UI.
4. Ensure the indentation in the `shippable.yml` is correct. Here is an example
   
```
integration:
  hub:
    - integrationName:
      type:
      agent_only:
      branches:
```
---

### The Docker Engine version is less than the minimum
When using Docker Compose in the `post_ci` section, the build fails with the following error:

```
The Docker Engine version is less than the minimum required by Compose. Your current project requires a Docker Engine of version 1.10.0 or greater
```

Reason: The standard AMI uses Docker Engine of version 1.9. Our normal policy is to upgrade Docker version on our AMI every quarter after extensive testing. Since Docker Compose will not work on this version, we've created a new AMI and allow users to choose that for the build from the 'Subscription Settings'

***How to avoid:** Navigate to the subscription settings page and select the unstable image which has docker version 1.11.1 available on it and all the builds for your subscription will be using this image to run your builds. For more info check out the [Machine Images Section](ci_subscriptions.md#selecting-the-machine-images).

---


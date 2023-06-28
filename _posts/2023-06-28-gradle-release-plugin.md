---
layout: post
title: "Gradle Release Plugin: A step-by-step tutorial for GitHub Actions, GitLab CI/CD, and Jenkins projects"
description: "Automating the application version release process"
date: 2023-06-28 09:34:00 +0200
background: '/img/posts/post02.jpg'
image: 'https://ibb.co/NpkcmY8'
---

# Automating the application version release process

Hello! Today, I want to talk about the Gradle Release Plugin.
I will show you how to use it to automate the process of releasing a new version of your application.
I will also demonstrate how to configure it to work with GitHub Actions, GitLab CI/CD and Jenkins.

But why do we need to automate the release process? I think the answer is obvious. Automation saves time and reduces the risk of human error.
When you automate the release process, you can focus on more important things, like writing code or fixing bugs.
Just imagine – all you need to do is click a button, and voila, your application is released! Isn't that fantastic?

&nbsp;
## Table of Contents

1. [Gradle Release Plugin](#release-plugin-for-gradle-projects)
    1. [What exactly does the Gradle Release Plugin do in operation?](#what-exactly-does-the-gradle-release-plugin-do-in-operation)
    2. [Install Gradle Release Plugin in the project](#install-gradle-release-plugin-in-the-project)
    3. [Configure Gradle Release Plugin](#configure-gradle-release-plugin)
2. [Implementing the release process in CI/CD](#implementing-the-release-process-in-cicd)
    1. [GitHub Actions](#github-actions)
    2. [GitLab CI/CD](#gitlab-cicd)
    3. [Jenkins](#jenkins)
3. [Summary](#summary)

&nbsp;
## Release Plugin for Gradle projects

Most popular build tools have their own release plugins. For example, for Maven projects, we have the Maven Release Plugin.
For Gradle projects, we have the Gradle Release Plugin. It's a Maven-like plugin that was initially missing in Gradle.
This plugin offers practically the same functionality as the Maven Release Plugin.

In this post, our focus will primarily be on the basic functionality of the Gradle Release Plugin: automatic versioning and releasing of the application.

For simplicity, let's assume that our project is in trunk-based development.
In short: we have only one branch (called main/master/trunk). Developers create a new branch from the main one whenever a new feature needs to be added. 
Once the feature is ready, it is merged back into the main branch. In trunk-based development, there aren't any release branches; 
we release only from the main branch.

Of course, you can employ the Gradle Release Plugin with other branching models, 
but this won't be covered in this post. The trunk-based model is the simplest and ideal for demonstrating the functionality of the release plugin.


### What exactly does the Gradle Release Plugin do in operation?

The Gradle Release Plugin executes the following actions in the given sequence:
* **checking for uncommitted changes and unpushed commits**: the plugin ensures there are no uncommitted changes and that all commits have been pushed to the remote repository
* **building** (only if you set groups of build tasks in configuration): the project is built, tested, and final artifacts are created
* **versioning**: the plugin determines and updates the project version (which you can set in gradle.properties), commits the change, and pushes it to the remote repository; 
this includes creating a new VCS tag corresponding to the new version
* **snapshot versioning for continued development**: the project version is updated to the next snapshot version ($version-SNAPSHOT), the change is committed, and pushed to the remote repository for continued development

It's crucial to note that you can configure the plugin to skip some of these steps.

Below is an illustration showing sample commits resulting from using the Gradle Release Plugin:

<a href="https://ibb.co/74BWBM9"><img src="https://i.ibb.co/ySKyKv3/ex1.png" alt="ex1" border="0"></a>


&nbsp;
### Install Gradle Release Plugin in the project

To install the Release Plugin in your Gradle project, you need to add the following line to `build.gradle` file:

```groovy
plugins {
    id 'net.researchgate.release' version '3.0.2'
}
```

You can find the latest version number at: [https://plugins.gradle.org/plugin/net.researchgate.release](https://plugins.gradle.org/plugin/net.researchgate.release)

&nbsp;
### Configure Gradle Release Plugin

The Release Plugin can be configured in `build.gradle` file by overwriting selected properties in `release` section.
Below you can find a portion of the default configuration settings from [https://github.com/researchgate/gradle-release](https://github.com/researchgate/gradle-release):

```groovy
release {
    failOnCommitNeeded = true
    failOnPublishNeeded = true
    failOnSnapshotDependencies = true
    failOnUnversionedFiles = true
    failOnUpdateNeeded = true
    preTagCommitMessage = '[Gradle Release Plugin] - pre tag commit: '
    tagCommitMessage = '[Gradle Release Plugin] - creating tag: '
    newVersionCommitMessage = '[Gradle Release Plugin] - new version commit: '
    tagTemplate = '${version}'
    versionPropertyFile = 'gradle.properties'
    snapshotSuffix = '-SNAPSHOT'
    buildTasks = []
    
    git {
        requireBranch.set('main')
    }
}
```

Here are a few of my personal insights:
* Be aware that the Gradle Release Plugin reads (and overwrites) the project version from the `gradle.properties` file
* Pay attention to `release.git.requireBranch`. Don't forget to set this property to the name of the branch from which you plan to release your application
* `release.buildTasks` is a list of tasks that will be executed during the release process. It's a good practice to set it to the build task (like ["build"]). 
Additionally, you can add tasks like "artifactoryPublish" to further develop your automation process

&nbsp;
## Implementing the release process in CI/CD workflow

&nbsp;
### GitHub Actions

Let's start with GitHub Actions. Here is a sample workflow that implements the release process:

```yaml
# gradle-release-plugin.yml

name: Create release

# Controls when the action will run
# Option bellow allows you to run the workflow manually
on:                                                              
  workflow_dispatch:
    inputs:
      release_version:
        description: 'Release new version'

# Allow GitHub Actions to push changes to the repository 
permissions:
  contents: write

jobs:
    release:
        name: Release
        runs-on: ubuntu-latest
        steps:
          
        - name: Checkout
          uses: actions/checkout@v2
          
        - name: Setup Gradle                                    
          uses: gradle/gradle-build-action@v2
          
        # Setup git config of the GitHub Actions Bot
        - name: Setup git config
          run: |
            git config user.name "GitHub Actions Bot"
            git config user.email "<>"
            
        # Run release task
        - name: Release with Gradle Release Plugin
          run: ./gradlew release
```

&nbsp;
### GitLab CI/CD

In order to empower our GitLab Runner to push new commits, 
we first need to generate a Personal Access Token. This token will grant the necessary permissions to our Runner,
enabling it to interact with our GitLab repository as if it were a user:

1. **Generate a Personal Access Token**:
- navigate to your profile in GitLab
- choose "Preferences" > "Access Tokens"
- create a new token with api, read_repository and write_repository permissions
2. **Add your token as an environment variable to your GitLab project**:
- go to your project in GitLab
- navigate to "Settings" > "CI/CD" > "Variables"
- add a new variable, for example, name it GIT_PUSH_TOKEN
- paste your Personal Access Token as the value of this new variable

Below is a sample GitLab CI/CD workflow that implements the release process:

```yaml
# .gitlab-ci.yml

# Use Docker image with Gradle installed
image: gradle:alpine

stages:
  - release

before_script:
  - export GRADLE_USER_HOME=`pwd`/.gradle
  - git fetch origin

create_release:
  stage: release
  script:
    # Setup git config of the GitLab Runner
    - git config user.name "GitLab Runner"
    - git config user.email ""
     
    # Use Personal Access Token to push changes to the repository
    - git remote set-url origin "https://oauth2:${GIT_PUSH_TOKEN}@gitlab.com/${CI_PROJECT_PATH}.git"
    
    # Checkout branch from which you want to release
    - git checkout $CI_COMMIT_REF_NAME
    
    # Run release task
    - ./gradlew release
    
  rules:
    # Run only on branch which you want to release 
    - if: $CI_COMMIT_BRANCH == "main"
      when: manual
    - when: never
```

&nbsp;
### Jenkins

If Jenkins is your chosen tool, you can use the following pipeline to implement the release process:

```groovy
// Jenkinsfile

pipeline {
    agent any

    // Run release process only when CREATE_RELEASE starting parameter is set to true
    parameters {
        booleanParam(
             name: 'CREATE_RELEASE',
             defaultValue: false,
             description: 'Do you want to create a release?'
        )
    }

    stages {
        
        // Checkout to the branch from which you want to release
        stage('Checkout') {
            when {
                expression { params.CREATE_RELEASE }
            }
            steps {
                // You can add also credentialsId parameter to below command
                git branch: "${env.BRANCH_NAME}", url: "${env.GIT_URL}"
            }
        }

        // Setup git config of the Jenkins
        stage('Setup git config') {
            when {
                expression { params.CREATE_RELEASE }
            }
            steps {
                script {
                    PROJECT_PATH = "${env.GIT_URL}".replaceFirst("https://", "")
                }
                sh "git config --global user.name 'Jenkins'"
                sh "git config --global user.email ''"
                
                // Option for GitLab repository
                sh "git remote set-url origin \"https://oauth2:${GIT_PUSH_TOKEN}@${PROJECT_PATH}\""
            }
        }

        // Run release task
        stage('Release') {
            when {
                expression { params.CREATE_RELEASE }
            }
            steps {
                sh "./gradlew release"
            }
        }
    }
}
```


If you're considering using Jenkins for a GitHub or GitLab repository,
Jenkins will need the ability to push new commits. 
This requires us to generate a Personal Access Token for either service.

The process to generate a PAT in GitLab is described in the GitLab CI/CD section above. 

To generate a PAT in GitHub, follow these steps:
* navigate to your profile in GitHub
* choose "Settings" > "Developer settings" > "Personal access tokens"
* create a new token with repo permissions

Next, add your token as a credential to your Jenkins project:
* go to your project in Jenkins
* navigate to "Security" > "Credentials" > "System" > "Global credentials" > "Add Credentials"
* choose 'Username with password' as the kind of credential
* set your username and Personal Access Token as password
* set ID of your credential to GITHUB_CREDENTIALS, for example
* save your credentials

Finally, incorporate your credentials into your Jenkins pipeline:
* go to your project in Jenkins
* navigate to "Configure" > "Git" > "Credentials" and choose your credentials from the list

You can also add your credentialsId as a parameter to your pipeline in the Checkout stage:

`git branch: "${env.BRANCH_NAME}", url: "${env.GIT_URL}", credentialsId: "GITHUB_CREDENTIALS"`

&nbsp;
## Summary

I trust that this article has highlighted the immense utility of the Gradle Release Plugin 
for automating the release process.
I've tried to show how to use it with different CI/CD tools. The Gradle Release Plugin is strong and flexible, 
so you can easily change it to fit your needs.

I encourage you to try out this plugin in your Gradle projects.
<br/><br/>

<script src="https://utteranc.es/client.js"
        repo="wszlosek/DevDawn"
        issue-term="title"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>

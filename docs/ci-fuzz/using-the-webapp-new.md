"

### Preparation

#### Git repository

First, you need a fuzzing project in a git repository. You can do this is by using the CI Fuzz local installation or in case of Web Fuzzing, the template project. You have to push your fuzz tests to your project repository.

$ git add .code-intelligence  
$ git commit -m ""Add ci-fuzz project setup and fuzz tests""  
$ git push

#### Run image

When running the fuzz tests, the CI Fuzz server will pull the run docker image you configured in project.yaml, so if you are using a custom one, make sure it is available at a docker registry like docker hub or gcr.io.

### Sign In to the Web App

Open https://app.code-intelligence.com (or the WWW address of your on-prem CI Fuzz) in your web browser. To sign in to the web app, you can use one of the configured authentication providers.

If you want to work on a project together with others, you can use an organization. To learn more about how organizations work, read Work Together Using Organizations.

### Setting up your project

In the CI Fuzz Web interface, a project is a space where you will see your fuzzing runs and fuzzing results, as well as access some functionality that will help you set up fuzzing in CICD.

When you click ""New Project"" you can decide if you want the project to be owned by your organization or if you want to create a personal project.

In the next step, you need to provide the git URL of your project repository.

  

### Running fuzz tests

When you first create the project, there are no fuzz tests visible. They will appear when you first upload a fuzzing artefact and start fuzzing.

You can continue with Continuous Fuzzing Setup.

"
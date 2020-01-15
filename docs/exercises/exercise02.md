# Exercise 2 - Building a Pipeline for CI/CD

In this exercise we will be creating a CI/CD Pipeline within OpenShift, once again deploying a simple application that can be continually built when developers commit code to their code repository. 

This exercise leverages an OpenShift Template, which defines and bundles a group of kubernetes objects, containers and pipelines and as a result, end to end application lifecycle, into a single parameterised deployable artefact.

The templated CI/CD Pipeline, uses three (3) separate projects, (by default named `cicd`, `dev` and `stage`). The cicd project contains all of the CI/CD tooling (Jenkins, Nexus, Gogs and SonarQube), used to continuously build and test the application. The DEV project is then used to deploy the application container for any additional integration testing. Once any testing within DEV is complete, the container can be deployed to STAGE. Promotion to STAGE requires human intervention to authorise this promotion.

Similar to the earlier exercise, this pipeline is a very simple example, yet it illustrates the power and flexibility of these processes within OpenShift.

Templates need to be parsed before the generated object definitions can be created within OpenShift. This can be completed either from the CLI, or if the template has already been imported into the cluster, from the WebUI.

The template deploys a simple `tasks` application, which is a Java based REST web application utilising JAX-RS.

## Login to the OpenShift Cluster

For this exercise an OpenShift cluster has been created and made available for your use. You can login using either the WebUI or CLI, both methods use the same credentials that have been provided.

To login via the WebUI, simply navigate to the console URL provided.

To login from an SSH terminal, use the OpenShift CLI (`oc`), in the following format.

$ oc login -u <your_username> -p <your_password> https://openshift-cluster.example.com

## Step 1

### Clone the CI/CD Demo Repository

Using the bastion host, or your local workstation, clone the repository.

$ git clone https://github.com/Purple-Sky-Pirates/exercise02-code.git

Checkout the `ocp-4.2` branch

$ git checkout ocp-4.2

### Create Base Projects

As mentioned, this template utilises three projects so lets create them now.

IMPORTANT: Kubernetes does not allow multiple projects to use the same name, so for this exercise, append your username to each of the project names, (eg `cicd-jbloggs`, `dev-jbloggs`, `stage-jbloggs`), to ensure you don't trample on each others projects.

$ oc new-project cicd-<username> --display-name="<username> - <username> CI/CD"
$ oc new-project dev-<username> --display-name="<username> - <username> Dev"
$ oc new-project stage-<username> --display-name="<username> <username> Stage"

### Create Jenkins Pod

Although we could include the creation of a Jenkins instance within the larger template, in this example we're using the standard Persistent Jenkins template to create a Jenkins Pod within our project.

$ oc new-app jenkins-persistent -n cicd-<username>

The Jenkins container will be deployed into your *CI/CD* project, but to allow it to deploy the built container into the Dev and Stage environment, we need to give it permission to do so.

$ oc policy add-role-to-group edit system:serviceaccounts:cicd-<username> -n dev-<username>
$ oc policy add-role-to-group edit system:serviceaccounts:cicd-<username> -n stage-<username>

### Examine the Template

Navigate to https://github.com/Purple-Sky-Pirates/exercise02-code/blob/ocp-4.2/cicd-template.yaml and take a look at the template provided. This is quite a complex, involved template which both sets up some base object, and runs a Kubernetes `job` to configure and create even more objects. Note that a Jenkinsfile is provided which encloses the entire Jenkins Pipeline code.

### Deploy the Template

$ oc new-app -n cicd-<username> -f cicd-template.yaml --param DEV_PROJECT=dev-<username> --param STAGE_PROJECT=stage-<usename> --param ENABLE_QUAY=true --param QUAY_USERNAME=<quay_username> --param QUAY_PASSWORD=<quay_password> --param QUAY_REPOSITORY=tasks-app --param QUAY_ORG=<quay_org> --param EPHEMERAL=false


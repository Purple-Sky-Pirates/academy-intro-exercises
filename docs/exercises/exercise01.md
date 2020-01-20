# Exercise 1 - Basic OpenShift Tasks

In this exercise we will be creating a base project within OpenShift, and deploying a simple application using the OpenShift catalogue. You can accomplish this using the WebUI or CLI method. Instructions for both methods are provided.

## Login to the OpenShift Cluster

For this exercise an OpenShift cluster has been created and made available for your use. You can login using either the WebUI or CLI, both methods use the same credentials that have been provided.

Any word in all caps preceded by a dollar sign is an environment variable. You can either manually replace these with your provided username and password (e.g. username = "Bob" and password = "p4ssword"), or you can set these as environment variables and (most) command lines will automatically substitute these with the set variable.

```
$ YOURUSERNAME="Bob"             # sets variable for current shell only
$ export YOURPASSWORD="p4ssword" # set for current shell and all processes
```

To check that these have been set type `echo $YOURUSERNAME` If successful this should output `Bob`.

To login via the WebUI, simply navigate to the console URL provided.

To login from an SSH terminal, use the OpenShift CLI (`oc`), in the following format.

```
$ oc login -u $YOURUSERNAME -p $YOURPASSWORD https://openshift-cluster.example.com
```

## Step 1

### Create the base Project using the WebUI

The WebUI provides many simple to use methods to create, run and manage projects, applications and other kubernetes objects, within OpenShift.

From the WebUI Developer Console, click *Projects* within the Advanced section.
Click *Create Project* and fill in `Name`, `Display Name` and optionally `Description`. (Display Name is also optional, but makes navigating around both the WebUI and CLI much more intuitive).

### Create the base Project using the CLI

The OpenShift CLI `oc` provides a number of helper functions over and above the default kubernetes CLI, `kubectl`, this includes a utility function to create projects.

```
$ oc new-project my-cool-app --display-name="My Cool Application" --description="Development project for my cool new application"
```

The `oc` CLI will automatically move your context so that you are *inside* the project you have just created. You can use `oc project` to verify this at any time.

```
$ oc project
```

Using project "my-cool-app" on server "https://api.luminiferous.597b.sandbox882.opentlc.com:6443"


### Have a Look Around

Now that you have a project, have a look around. You should note that some elements have been pre-populated for you as defaults for new projects. Project defaults are defined by the platform administrator.

Using the WebUI or CLI, examine the pre-created elements such as Service Accounts.


## Step 2

### Create a Basic Application using the WebUI

Again in the Developer Console, click on `Add` which will display a number of options. For this example, click *From Catalog* and scroll down to and click, the base Python (rather than either Django + PostgreSQL options) item in the Catalog, and then click *Create Application*.

Here you can simply choose from available Python versions, and the Git Repository that your code is stored, and application will be built from. For this example, leave the defaults as they are, but add the URL below to add the Git repository.

https://github.com/Purple-Sky-Pirates/exercise01-code.git

### Monitor the Build and Deployment using the WebUI

Click *Topology* from the side menu.
Click the `python` application, a drawer will open from the right.

Click `View Logs` in the Build section, to view build logs.
Click `View Logs` in the Pods section, to view runtime pod logs.

### Create a Basic Application using the CLI

Once again, `oc` provides a number of helper functions that enable the easy deployment of applications, with its `new-app` switch. `new-app` helps by automagically creating a number of objects depending upon your chosen strategy and provided variables.

Here we will be using `oc new-app` to deploy a simple Django based web application, based upon git repository that has been pre created for this exercise. This application is a *very* simple example, however it quickly shows some of the power that is available, as `new-app` created 

.Navigate to your just created project

```
$ oc project my-cool-app
```

.Create an Application

```
$ oc new-app python -p SOURCE_REPOSITORY_URL=https://github.com/Purple-Sky-Pirates/exercise01-code.git
```

### Monitor the Build and Deployment using the CLI

```
$ oc get events
```
To view any events that occur (such as pod creation, container build orders, etc)

```
$ oc logs bc/python
```
To view build logs

```
$ oc logs dc/python
```
To view runtime pod deployment logs
 

## View the Running Application using a Browser

The deployment can be viewed using a Web Browser, by simply navigating to the auto-magically created `route` that OpenShift created either from the Catalog, or using `oc new-app`.

The route itself can be found, either within the WebUI (in the Topology view click the `Open URL` arrow attached to the deployment), or by using `oc get route` from the CLI.


## Round Up

A lot of power is hidden within these simple commands, with the default project template enabling the user to define and create many objects, short circuiting many of the basic build options that can be quite complex within kubernetes.

In addition, the `new-app` functionality is using the baseline code, alongside a base builder image to both build and deploy the application container(s). The mechanism in this example is using S2I (Source to Image), which is an OpenShift mechanism to allow the build and deploy from application source code. Red Hat provide S2I based builder images for many languages including Python, Node.js, Javascript, Go, Java, PHP, .Net core, Ruby and Perl.

## Stretch Goal

Using the power of the internet (or your own inquisitiveness in the CLI/WebUI), scale your app to more than one running pod. 

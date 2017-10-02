# Reaction Platform

Reaction Commerce offers a managed deployment option, the [Reaction Platform](http://getrxn.io/reaction-platform).

The `reaction-cli` incorporates functionality for any team to deploy Reaction to multiple environments. [Visit reactioncommerce.com](http://getrxn.io/reaction-platform) to get a demo or request an **invite token**.

1.  [Request invite token](http://getrxn.io/reaction-platform).
2.  Register local environment
3.  Create, add, and publish SSH keys
4.  Create application environment
5.  Deploy application

## Configuration

As a user of the **Reaction Platform**, you'll receive an email containing a "invite token" from the Reaction Platform API ("Launchdock"). **Launchdock** is the name of our internal  orchestration management platform.

You will be asked for your [invite token](http://getrxn.io/reaction-platform) when you use the [reaction-cli](http://getrxn.io/reaction-cli) to register your local Reaction environment with the Reaction Platform API.

### register

`reaction register` to create keys locally before configuring your application environment. `reaction login` authenticates you to the Reaction Platform and syncronizes platform services.

```sh
# Register with invite token
reaction register

# or if you've already registered, login with your username and password
reaction login
```

### keys

You will need to set up an SSH key to securely communicate with Launchdock.

Set up an SSH key pair:

```sh
# https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/
#
# create a new SSH key pair
# prompts for filename
# suggest "~/.ssh/launchdock" for ease
ssh-keygen -t rsa -b 4096 -C "you@example.com"

# make sure the ssh-agent is running in the background
eval "$(ssh-agent -s)"

# add your new key to the agent
ssh-add -K ~/.ssh/<private key created above>

# add your public key to Launchdock
reaction keys add ~/.ssh/<keyname>.pub
```

## Applications

An "application" is an instance of Reaction Commerce plus any services, connectors, or container dependencies that are configured to deploy as one to an application environment.

### create

Creating an "application" configures the environment that the built application will be deployed into.

```sh
# Create and configure your new app deployment
reaction apps create \
  --name <appname> \
  -e REACTION_USER="yourname" \
  -e REACTION_AUTH="P@s5w0rd" \
  -e REACTION_EMAIL="you@example.com" \
  --registry  path/to/reaction.json \
  --settings path/to/settings.json
```

#### env

Configure environment variables for the application deployment.

```sh
# set/update environment variables
# (this triggers a redeploy of your app)
reaction env set \
  --app <appname> \
  -e SOME_API_KEY="ec89jmur3jim8e34" \
  -e SOME_OTHER_THING="dj8dr34ju3r@#$" \
  -e MAIL_URL="smtp://USERNAME:PASSWORD@NEW_HOST:PORT"

# unset environment variables
# (this triggers a redeploy of your app)
reaction env unset --app <appname> -e SOME_API_KEY -e SOME_OTHER_THING

# list your currently set environment variables
reaction env list --app <appname>
```

**--registry** pass a `reaction.json` to the environment

**--settings** pass a Meteor `settings.json` to the environment

#### domains

```sh
# add a custom domain for your app
# (first, update your DNS to point your domain at your app's default URL)
reaction domains add --app <appname> -d mycoolshop.com

# remove a custom domain from your app
reaction domains remove --app <appname> -d mycoolshop.com

#### apps list
# list your apps and their domains
reaction apps list
```

### deploy

Submit your application for a deployment. This will begin the CI/CD/CD process. This begins a build process that is unique for every project. Expect this entire process to take around 20 minutes.

-   test and build production bundle (continious integration)
-   build container image and publish (continious delivery)
-   deploy application cluster (continious deployment)

```sh
    # Deploy your application
    reaction deploy --name <appname>
```

Expect to recieve an email with the completed build status (failed or deployed).

#### CI/CD Configuration

[Continious Integration configuration](https://docs.gitlab.com/ee/ci/) should be commited in `.reaction/ci/config.yml`.

## Basic Example

Below is an example `simple-demo` application deployment, setting the minimum required settings.

```sh
# create the application environment
reaction apps create --name simple-demo

# deploy application to the environment

reaction deploy --app simple-demo

# open your app in your browser

reaction open simple-demo
```

## Full Example

  Below is a more complete example that sets up a SMTP mail server URL (for app emails), imports [Reaction registry](https://docs.reactioncommerce.com/reaction-docs/master/registry) settings and [Meteor settings](https://docs.meteor.com/api/core.html#Meteor-settings), and deploys the latest official Reaction image. Then we update the app with an API key environment variable.  And finally, we add a custom domain to the app and open it in our browser.

```sh
# create the app
reaction apps create \
  --name full-demo \
  -e REACTION_USER="yourname" \
  -e REACTION_AUTH="P@s5w0rd" \
  -e REACTION_EMAIL="you@example.com" \
  -e MAIL_URL="smtp://USERNAME:PASSWORD@NEW_HOST:PORT" \
  --registry  private/settings/reaction.json \
  --settings settings/settings.json

# deploy a Docker image
reaction deploy --app full-demo

# set/update an environment variable
reaction env set --app full-demo -e SOME_API_KEY="<secret API key>"

# add a custom domain
reaction domains add --app full-demo --domain mycoolshop.com

# list your apps to confirm your configuration, URL's, etc.
reaction apps list

# open your app in your browser
reaction open full-demo
```
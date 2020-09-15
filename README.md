# Cloud-enabled Amplify DataStore workshop using Vue

In this workshop we'll learn how to use Amplify DataStore to create `Chatty` a single room realtime multi-user chat app using Vue & [AWS Amplify](https://aws-amplify.github.io/).

![](./header.png)

### Topics we'll be covering:

- [Authentication](#adding-authentication)
- [GraphQL API with AWS AppSync](#adding-a-graphql-api)
- [Setup Amplify DataStore](#setup-amplify-datastore)
- [Deploying via the Amplify Console](#deploying-via-the-amplify-console)
- [Removing services](#removing-services)
- [Appendix and trobleshooting](#appendix)

## Pre-requisites

- Node: `14.7.0`. Visit [Node](https://nodejs.org/en/download/current/)
- npm: `6.14.7`. Packaged with Node otherwise run upgrade

```bash
npm install -g npm
```

## Getting Started - Creating the Application

To get started, we first need to create a new Vue project & change into the new directory using the [Vue CLI](https://github.com/vuejs/vue-cli).

If you already have it installed, skip to the next step. If not, either install the CLI & create the app or create a new app using:

```bash
npm install -g @vue/cli
vue create amplify-datastore
```

Vue CLI
- ? Please pick a preset: __default (babel, eslint)__
 
Now change into the new app directory and make sure it runs

```bash
cd amplify-datastore
npm run serve
```

## Installing the CLI & initializing a new AWS Amplify project

Let's now install the AWS Amplify API & AWS Amplify Vue library:

```bash
npm install --save aws-amplify @aws-amplify/ui-vue moment
```
> If you have issues related to EACCESS try using sudo: `sudo npm <command>`.

### Installing the AWS Amplify CLI

Next, we'll install the AWS Amplify CLI:

```bash
npm install -g @aws-amplify/cli
```

Now we need to configure the CLI with our credentials:

```js
amplify configure
```

> If you'd like to see a video walkthrough of this configuration process, click [here](https://www.youtube.com/watch?v=fWbM5DLh25U).

Here we'll walk through the `amplify configure` setup. Once you've signed in to the AWS console, continue:
- Specify the AWS Region: __eu-central-1 (Frankfurt)__
- Specify the username of the new IAM user: __amplify-datastore__
> In the AWS Console, click __Next: Permissions__, __Next: Tags__, __Next: Review__, & __Create User__ to create the new IAM user. Then, return to the command line & press Enter.
- Enter the access key of the newly created user:   
  accessKeyId: __(<YOUR_ACCESS_KEY_ID>)__   
  secretAccessKey:  __(<YOUR_SECRET_ACCESS_KEY>)__
- Profile Name: __default__

> To view the new created IAM User go to the dashboard at [https://console.aws.amazon.com/iam/home#/users/](https://console.aws.amazon.com/iam/home#/users/). Also be sure that your region matches your selection.

### Initializing a new project

```bash
amplify init
```

- Enter a name for the project: __amplify-datastore__
- Enter a name for the environment: __dev__
- Choose your default editor: __Visual Studio Code__   
- Please choose the type of app that you're building __javascript__   
- What javascript framework are you using __vue__   
- Source Directory Path: __src__   
- Distribution Directory Path: __dist__   
- Build Command: __npm run-script build__   
- Start Command: __npm run-script serve__
- Do you want to use an AWS profile? __Yes__
- Please choose the profile you want to use __default__

Now, the AWS Amplify CLI has iniatilized a new project & you will see a new folder: __amplify__. The files in this folder hold your project configuration.

```bash
<amplify-app>
    |_ amplify
      |_ .config
      |_ #current-cloud-backend
      |_ backend
      team-provider-info.json
```

## Adding authentication

To add authentication to our Amplify project, we can use the following command:

```sh
amplify add auth
```

> When prompted choose 
- Do you want to use default authentication and security configuration?: __Default configuration__
- How do you want users to be able to sign in when using your Cognito User Pool?: __Username__
- Do you want to configure advanced settings? __Yes, I want to make some additional changes.__
- What attributes are required for signing up? (Press &lt;space&gt; to select, &lt;a&gt; to 
toggle all, &lt;i&gt; to invert selection): __Email__
- Do you want to enable any of the following capabilities? (Press &lt;space&gt; to select, &lt;a&gt; to toggle all, &lt;i&gt; to invert selection): __None__

> To select none just press `Enter` in the last option.

Now, we'll run the push command and the cloud resources will be created in our AWS account.

```bash
amplify push

Current Environment: dev

| Category | Resource name      | Operation | Provider plugin   |
| -------- | ------------------ | --------- | ----------------- |
| Auth     | amplifyappuuid     | Create    | awscloudformation |
? Are you sure you want to continue? Yes
```


To quickly check your newly created __Cognito User Pool__ you can run

```bash
amplify status
```

> To access the __AWS Cognito Console__ at any time, go to the dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

### Configuring the Vue application

Now, our resources are created & we can start using them!

The first thing we need to do is to configure our Vue application to be aware of our new AWS Amplify project. We can do this by referencing the auto-generated `aws-exports.js` file that is now in our `src` folder.

To configure the app, open __main.js__ and add the following code below the last import:

```js
import Vue from 'vue'
import App from './App.vue'

import Amplify from 'aws-amplify';
import '@aws-amplify/ui-vue';
import aws_exports from './aws-exports';

Amplify.configure(aws_exports);

Vue.config.productionTip = false

new Vue({
  render: h => h(App),
}).$mount('#app')
```

Now, our app is ready to start using our AWS services.

### Using the Authenticator Component

AWS Amplify provides UI components that you can use in your App. Let's add these components to the project

In order to use the Authenticator Component add it to __src/App.vue__:

```html
<template>
  <div id="app">
    <amplify-authenticator>
      <div>
        <h1>Hey, {{user.username}}!</h1>
        <amplify-sign-out></amplify-sign-out>
      </div>
    </amplify-authenticator>
  </div>
</template>
```

Now, we can run the app and see that an Authentication flow has been added in front of our App component. This flow gives users the ability to sign up & sign in.

> To view any users that were created, go back to the __Cognito__ dashboard at [https://console.aws.amazon.com/cognito/](https://console.aws.amazon.com/cognito/). Also be sure that your region is set correctly.

Alternatively we can also use

```bash
amplify console auth
```

### Accessing User Data

By listening to authentication state changes using `onAuthUIStateChange` we can access the user's info once they are signed in as shown below.

```js
<script>
import { AuthState, onAuthUIStateChange } from '@aws-amplify/ui-components'

export default {
  name: 'app',
  data() {
    return {
      user: { },
    }
  },
  created() {
    // authentication state managament
    onAuthUIStateChange((state, user) => {
      // set current user and load data after login
      if (state === AuthState.SignedIn) {
        this.user = user;
      }
    })
  }
}
</script>
```

## Adding a GraphQL API

To add a GraphQL API, we can use the following command:

```sh
amplify add api
```

Answer the following questions

- Please select from one of the below mentioned services __GraphQL__
- Provide API name: __ChattyAPI__
- Choose the default authorization type for the API __API key__
- Enter a description for the API key: __(empty)__
- After how many days from now the API key should expire (1-365): __180__
- Do you want to configure advanced settings for the GraphQL API __Yes, I want to make some additional changes.__
- Configure additional auth types? __N__
- Configure conflict detection? __Y__
- Select the default resolution strategy __Auto Merge__
- Do you want to override default per model settings? __N__
- Do you have an annotated GraphQL schema? __N__ 
- Do you want a guided schema creation? __Y__   
- What best describes your project: __Single object with fields (e.g. “Todo” with ID, name, description)__   
- Do you want to edit the schema now? __Y__

> To select none just press `Enter`.

> When prompted, update the schema to the following:   

```graphql
type Chatty @model {
  id: ID!
  user: String!
  message: String!
  createdAt: AWSDateTime
}
```

This will allow us to display each user messages together with the creation date and time.

> Note: Don't forget to save the changes to the schema file!

Next, let's push the configuration to our account:

```bash
amplify push
```

- Are you sure you want to continue? __Yes__
- Do you want to generate code for your newly created GraphQL API __Yes__
- Choose the code generation language target __javascript__
- Enter the file name pattern of graphql queries, mutations and subscriptions __src/graphql/**/*.js__
- Do you want to generate/update all possible GraphQL operations - queries, mutations and subscriptions __Yes__
- Enter maximum statement depth [increase from default if your schema is deeply nested] __2__

Notice your __GraphQL endpoint__ and __API KEY__.

This step created a new AWS AppSync API. Use the command below to access the AWS AppSync dashboard. Make sure that your region is correct.

```bash
amplify console api
```

- Please select from one of the below mentioned services __GraphQL__

## Setup Amplify DataStore

### Installing the Amplify DataStore

Next, we'll install the necessary dependencies:

```bash
npm install --save @aws-amplify/core @aws-amplify/datastore
```

### Data model generation

Next, we'll generate the models to access our messages from our __ChattyAPI__

```bash
amplify codegen models
```

> Important: DO NOT forget to generate models every time you introduce a change in your schema.

Now, the AWS Amplify CLI has generated the necessary data models and you will see a new folder in your source: __models__. The files in this folder hold your data model classes and schema.

```bash
<amplify-app>
    |_ src
      |_ models
```

### Creating a message

Now that the GraphQL API and Data Models are created we can begin interacting with them!

The first thing we'll do is create a new message using the generated Data Models and save.

```js
import { DataStore } from "@aws-amplify/datastore";
import { Chatty } from "./models";

await DataStore.save(new Chatty({
  user: "amplify-user",
  message: "Hi everyone!",
  createdAt: new Date().toISOString()
}))
```

This will create a record locally in your browser and synchronise it in the background using the underlying GraphQL API. 

### Querying data

Let's now see how we can query data using Amplify DataStore. In order to query our Data Model we will use a query and a predicate to indicate that we want all records. 

```js
import { DataStore, Predicates } from "@aws-amplify/datastore";
import { Chatty } from "./models";

const messages = await DataStore.query(Chatty, Predicates.ALL);
```

This will return an array of messages that we can display in our UI.

Predicates also support filters for common types like Strings, Numbers and Lists.

> Find all supported filters at [Query with Predicates](https://aws-amplify.github.io/docs/js/datastore#query-with-predicates)


## Creating the UI

Now, let's look at how we can create the UI to create and display messages for our chat.

```html
<template>
  <div v-for="message of sorted" :key="message.id">
    <div>{{ message.user }} - {{ moment(message.createdAt).format('YYYY-MM-DD HH:mm:ss')}})</div>
    <div>{{ message.message }}</div>
  </div>
</template>
<script>
import { Auth } from "aws-amplify";
import { DataStore, Predicates } from "@aws-amplify/datastore";
import { Chatty } from "./models";
import moment from "moment";

export default {
  name: 'app',
  data() {
    return {
      user: {},
      messages: [],
    }
  },
  computed: {
    sorted() {
      return [...this.messages].sort((a, b) => -a.createdAt.localeCompare(b.createdAt));
    }
  },
  created() {
    this.currentUser();
  },
  methods: {
    moment: () => moment(),
    currentUser() {
      Auth.currentAuthenticatedUser().then(user => {
        this.user = user;
        this.loadMessages();
      });
    },
    loadMessages() {
      DataStore.query(Chatty, Predicates.ALL).then(messages => {
        this.messages = messages;
      });
    },
  }
}
</script>
```

## Creating a message

 Now, let's look at how we can create new messages.

```html
<template>
  <form v-on:submit.prevent>
    <input v-model="form.message" placeholder="Enter your message..." />
    <button @click="sendMessage">Send</button>
  </form>
</template>
<script>
export default {
  data() {
    return {
      form: {},
    };
  }, 
  methods: {
    sendMessage() {
      const { message } = this.form
      if (!message) return;
      DataStore.save(new Chatty({
        user: this.user.username,
        message: message,
        createdAt: new Date().toISOString()
      })).then(() => {
        this.form = { message: '' };
        this.loadMessages();
      }).catch(e => {
        console.log('error creating message...', e);
      });
    },
  }
}
</script>
```

## Deleting all messages

One of the main advantages of working using Amplify DataStore is being able to run batch mutations without having to use a series of individual operations. 

See below how we can use delete together with a predicate to remove all messages.

```js
DataStore.delete(Chatty, Predicates.ALL).then(() => {
  console.log('messages deleted!');
});
```

### GraphQL subscriptions

Next, let's see how we can create a subscription to subscribe to changes of data in our API.

To do so, we need to listen to the subscription, & update the state whenever a new piece of data comes in through the subscription.

When the component is destroyed we will unsubscribe to avoid memory leaks.

```html
<script>
export default {
  data() {
    return {
      subscription: undefined;
    };
  },
  created() {
    //Subscribe to changes
    this.subscription = DataStore.observe(Chatty).subscribe(msg => {
      console.log(msg.model, msg.opType, msg.element);
      this.loadMessages();
    });
  }, 
  destroyed() {
    if (!this.subscription) return;
    this.subscription.unsubscribe();
  },
}
</script>
```

## Deploying via the Amplify Console

We have looked at deploying via the Amplify CLI hosting category, but what about if we wanted continous deployment? For this, we can use the [Amplify Console](https://aws.amazon.com/amplify/console/) to deploy the application.

The first thing we need to do is [create a new GitHub repo](https://github.com/new) for this project. Once we've created the repo, we'll copy the URL for the project to the clipboard & initialize git in our local project:

```sh
git init

git remote add origin git@github.com:username/project-name.git

git add .

git commit -m 'initial commit'

git push origin master
```

Next we'll visit the Amplify Console in our AWS account at [https://eu-central-1.console.aws.amazon.com/amplify/home](https://eu-central-1.console.aws.amazon.com/amplify/home).

Here, we'll click __Get Started__ to create a new deployment. Next, authorize Github as the repository service.

Next, we'll choose the new repository & branch for the project we just created & click __Next__.

In the next screen, we'll create a new role & use this role to allow the Amplify Console to deploy these resources & click __Next__.

Finally, we can click __Save and Deploy__ to deploy our application!

Now, we can push updates to Master to update our application.


## Removing services

If at any time, or at the end of this workshop, you would like to delete a service from your project & your account, you can do this by running the `amplify remove` command:

```sh
amplify remove auth

amplify push
```

If you are unsure of what services you have enabled at any time, you can run the `amplify status` command:

```sh
amplify status
```

`amplify status` will give you the list of resources that are currently enabled in your app.


## Appendix

### Setting up your AWS Account

In order to follow this workshop you need to create and activate an Amazon Web Services account. 

Follow the steps [here](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account)

### Trobleshooting

> Message: The AWS Access Key Id needs a subscription for the service

Solution: Make sure you are subscribed to the free plan. [Subscribe](https://portal.aws.amazon.com/billing/signup?type=resubscribe#/resubscribed)


> Message: TypeError: fsevents is not a constructor

Solution: `npm audit fix --force`


> Behaviour: data seems not to be synchronising with the cloud and or viceversa

Solution: 

```
amplify update api
amplify push
```

Make sure you answer the following questions as
- Configure conflict detection? __Y__
- Select the default resolution strategy __Auto Merge__
- Do you want to override default per model settings? __N__
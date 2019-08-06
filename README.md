## From Zero to Production 🚀

Welcome to our SIGNAL workshop! Today, you will be building a simple messaging app using Functions, Debugger, and the newly launched Twilio CLI.

The purpose of this workshop is to demonstrate how to build, deploy, and debug a Twilio application using these developer tools, and how powerfully and quickly you can get something up and running in production on Twilio.

---
### Prerequisites
This is a beginner friendly workshop. Whether you are new to Twilio or new to coding, you'll be able to participate.

At any point, if you are stuck on something, raise your hand and a TA will come help take a look.

---
### Installations
For Windows or Linux, follow the setup instructions on the CLI Quickstart https://www.twilio.com/docs/twilio-cli/quickstart.

For Mac, proceed with the steps below.

#### Node
Check which version of node you are using    
`node --version`  
If you have version 8.10.0 or above, skip ahead to Twilio CLI installation.

If you don't already have nvm (node version manager), install it  
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash`  
and run  
`export NVM_DIR="${XDG_CONFIG_HOME/:-$HOME/.}nvm" [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"`

You may need to open a new terminal session to load nvm.

Install node v8.10.0 using nvm. This will take a minute or two.  
`nvm install 8.10.0`

#### Twilio CLI
One of the easiest way to install twilio-cli on Mac OSX is to use Homebrew. If you don't already have it installed, install it by running    
`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`  
For more information on Homebrew or installation, visit their website https://brew.sh/.

Once Homebrew is installed, run the following to install twilio-cli  
`brew tap twilio/brew && brew install twilio`  

#### Serverless plugin
`twilio plugins:install @twilio-labs/plugin-serverless`  
The serverless plugin allows you to locally develop and deploy to Twilio Serverless.

After installing the serverless plugin, double check if the version of node you are now using is still `8.10.0` or above. If not, tell nvm to switch to the proper version.  
`nvm use 8.10.0`

#### Watch plugin
`twilio plugins:install @twilio-labs/plugin-watch`  
The watch plugin accesses and streams your Twilio debugger logs along with your calls and messages.

---
### Setup
Log into your Twilio account (or sign up for one) at http://www.twilio.com.

Back in your terminal, configure twilio-cli to connect to Twilio by running  
`twilio login`  
You can name the new project "signal-2019" or anything you want. Get the Account SID and Auth Token from the console home dashboard.

#### Buy a Twilio number
`twilio phone-numbers:buy:local --country-code US`

Set webhook for the phone number  
`twilio api:core:incoming-phone-numbers:update --sid {phone-number-sid} --sms-url http://www.twilio.com`

#### Initialize serverless project
Go to your home directory or where you'd like to keep your project. Create a new directory here and cd into it.  
`mkdir signal-2019 && cd signal-2019`

Initialize a serverless project called "workshop-1". This will take a minute.  
`twilio serverless:init workshop-1`  

If you see any errors, re-run the command in DEBUG mode.  
`DEBUG=* twilio serverless:init workshop-1`

Once it's done initializing, go into the serverless project directory. Here you can find your functions and assets.  
`cd workshop-1`

---
### Set environment variables
It's not a good idea to store credentials or personal data in the source code, which is why, if you open up the `.env` file in your serverless project directory, you will find an `ACCOUNT_SID` and an `AUTH_TOKEN` set here.

Replace the default `ACCOUNT_SID` and `AUTH_TOKEN` values with the correct credentials from your console home dashboard.
```
ACCOUNT_SID={YOUR_ACCOUNT_SID}
AUTH_TOKEN={YOUR_AUTH_TOKEN}
```

You will also need two phone numbers for the messaging app. Create two new variables, `TO` and `FROM`, in the `.env` file. Set `TO` as your cell phone number and `FROM` as the Twiio number you just purchased.
```
TO={YOUR_PHONE_NUMBER}
FROM={TWILIO_NUMBER}
```

---

### Start local development
Start up your functions and assets locally. The `--live` flag enables live reloading so you don't have to stop and restart the command on every change.  
`twilio serverless:start --live`

1. Go to http://localhost:3000/index.html in your browser. Read this page for a quick intro to Functions, Assets, and a description of each example file in the project.

Cool, now that you have some context, let's get started!

2. Go to http://localhost:3000/hello-world in your browser.

Open up the default function `hello-world.js` in an editor of your choice.

To validate that live reloading is working, modify the TwiML response to something different. Hit save, and refresh the browser.

---

### Build your messaging app

With local server still running, replace `hello-world.js` with the snippet below.

The `getTwilioClient()` method authenticates your account by fetching these credentials from `.env` which is represented by `context` here. Similarly, here you're setting the `to` and `from` parameters as the environment variables you just added to `.env`.

```js
exports.handler = function(context, event, callback) {
    const client = context.getTwilioClient();
    client
        .messages
        .create({
            body: 'Hello, welcome to SIGNAL!',
            to: context.TO,
            from: context.FROM
        })
        .then(message => {
            callback(null);
        })
        .catch(err => {
            callback(err);
        });
};
```

Hit save.

While we're here, let's rename the function. First stop the local server, then    
`mv functions/hello-world.js functions/send-message.js`

Restart the local server  
`twilio serverless:start --live`  

In your browser, visit the updated url http://localhost:3000/send-message. You should now receive a text message!

---
### Use private assets
Let's say for privacy reasons, you don't want to store the message body in code. You can leverage private assets to make sure your message doesn't get exposed via source code.

In `send-message.js`, add the following snippet before `const client = context.getTwilioClient();`
```js
  const assets = Runtime.getAssets();
  const privateMessageAsset = assets['/message.js'];
  const privateMessagePath = privateMessageAsset.path;
  const privateMessage = require(privateMessagePath);
```

And update the message `body` parameter to `privateMessage()`
```js
  body: privateMessage(),
```

Hit save and refresh the browser. You should receive a message with the secret text!

You can open up `assets/message.js` and modify the return value of the `privateMessage()` function, if you'd like to customize the secret message you're sending.

---
### Deploy your functions and assets
Your app is now ready! Let's deploy it to production.  

First, stop the local server, then run    
`twilio serverless:deploy`  

Again, if you see any errors, try running it in DEBUG mode  
`DEBUG=* twilio serverless:deploy`

...

Upon successful deployment, take out your phone and send a text to the Twilio number.

...

... Wait for it ...

...

No message is coming through. What is going on?

Let's use the watch plugin to help debug. You can stream errors and warnings on your account, along with incoming and outgoing messages, by running  
`twilio watch`

Once `watch` begins, send a text to the number again and monitor the watch output.

...

There is an error! Code `11200` indicates a HTTP retrieval failure, which most likely means a webhook failure.  
(Currently, only the error code and message title is displayed. Very soon, we'll support displaying error details for even more helpful debugging.)

We forgot an important step. Let's fix it.

---
### Configure phone number webhook

By configuring your function url as a phone number's webhook, it will get executed when the number receives a message.

Earlier, we set the number's webhook to be http://www.twilio.com, which is an invalid webhook hence the reason we saw the error above. Now let's correctly set the webhook to be your function url, which you can find from the output of the deploy command.

First view your phone numbers and their SIDs. Note the SID of the `FROM` number you are using.  
`twilio phone-numbers:list`  

Set the number's sms webhook url to your function url (the /send-message endpoint)  
`twilio api:core:incoming-phone-numbers:update --sid {phone-number-sid} --sms-url {function-url}`

Watch with `--show-recent-history` to view events that occurred before watch begins  
`twilio watch --show-recent-history`

Now try sending another text to the number!

You should be able to see 3 new events in the watch log: `received`, `sent`, and `delivered`  

Congratulations, your messaging application is now live and working in production! 🎉

---
### Next: Choose your own adventure

Now that you have a working development setup, you can build on what we've started or switch gears to explore other Twilio products and developer tools.

##### Node-savvy and want to work on the existing app?
Continue building features on the function (i.e. integrate with an external API, create a simple web app).

##### Interested in building a Twilio app without writing a single line of code?
Check out Studio, our drag-and-drop visual application builder at https://www.twilio.com/studio. You can learn more by going through the Studio mission at https://www.twilio.com/quest/missions.

##### Interested in exploring other Twilio products?
Check out TwilioQuest at https://www.twilio.com/quest/missions! There, you can choose various missions that teach you about individual products. Gain experience points that can be used as trial credit.

##### Want to learn more about the CLI and plugins?
https://www.twilio.com/docs/twilio-cli/quickstart  
https://www.twilio.com/docs/labs/serverless-toolkit  
https://github.com/twilio-labs/plugin-serverless
https://github.com/twilio-labs/plugin-watch

---
### We want your feedback!
To leave comments and/or suggestions for Twilio CLI: `twilio feedback`  

---

# Zoom Node.js Chatbot Command Line Interface
This is a cli package that automatically sets up a Node.js Zoom chatbot project for you. This allows you to quickly start developing chatbots without having to worry about setting up a project, doing Zoom side oauth, and other boilerplate logic and allows you to immediately concentrate on developing your business logic.

## Installation

```js
 npm i @zoomus/chatbot-cli -g
```


## Usage

1. `zoomchatbot create examplebot` //Choose one development deployment method in terminal
2. `cd examplebot` `npm run start` // start your app demo, use `npm run dynamodb` to create local dynamodb tables first
3. Visit https://marketplace.zoom.us/, create a new bot and copy credentials to your environment file,
4. In your Zoom Chat, you can enter commands to your bot, our skeleton code has "help" and "vote"(in demo type,will have meet extra)
5. `npm run test` //use jest to test app



## Setup & Features

See [marketplace docs](https://marketplace.zoom.us/docs/guides/chatbots/build-a-chatbot) to learn how to create bot and get information you need to paste into your code's environment variables file: .development.env for general mode, serverless.development.json for serverless mode

Here you will see either .development.env for general mode or serverless.development.json for serverless mode. You will need to fill this out with your bot's information. Below is an example screenshot:

![general](https://s3.amazonaws.com/user-content.stoplight.io/10128/1582241210819)



You will also see a file called botConfig.js. In this file, we can add features to our bot. Let's go over the sections of botConfig.js:


* apis: Configure api endpoints for your bot, please read comments in below code to understand it. Basically, when your bot receives a request, we will call the function specified in "callback" and auto inject useful objects into res.locals for you to use.see [chatbot](https://www.npmjs.com/package/@zoomus/chatbot) to get more details of zoomApp,zoomWebhook

```
//zoomType = 'command'|'auth'
//command type is match to zoom marketplace botendpoint url bind,just like https://.../command
//auth type is match to zoom marketplace auth redirect url bind,like https://.../auth
//will auto inject zoomApp & zoomWebhook & zoomError in command type callback(callback details in botCommands and botActions)
//will auto inject zoomApp & zoomError in auth type callback.If zoomError not toBeFalsy,you can see zoomError about error message from middleware
//zoomApp can use for sendMessage,request zoom openapi,you can see ./src example code and  @zoomus/chatbot for more details

apis: [
  { url: '/command', method: 'post', zoomType: 'command' },//callbacks see botCommands&botActions
  { url: '/auth', method: 'get', callback: require('./src/auth'), zoomType: 'auth' },
  {url:'/test',method:'get',callback:function(req,res,next){}}//it is a general api 
]
```


* botCommands: Configure your bot’s commands. When your bot's user's enter commands, it will call the function specified in the "callback" for that command. We again will auto inject useful objects (zoomApp,zoomWebhook,zoomError) into res.locals for you to use in your callback function.
```
botCommands: [
  {
    command: 'help',
    callback: require('./src/help.js')
  },
  {
  callback:require('./src/noCommand.js') // no matched command,will call this function
  }
]
```

* botActions: Configure your bot’s UI actions, trigger callback whenever a user presses a button, clicks a dropdown, edits a textbox, etc on your bot’s messages. more command type,please see [zoom-message-with-buttons](https://marketplace.zoom.us/docs/guides/chatbots/customizing-messages/message-with-buttons)
zoom support interactive_message_select,interactive_message_actions,interactive_message_editable,interactive_message_fields_editable four types,you can see [zoom-message-with-dropdown](https://marketplace.zoom.us/docs/guides/chatbots/customizing-messages/message-with-dropdown) for more details about zoom action

```
botActions: [
  {
  command:'interactive_message_actions',
  callback:require('./src/interactive_message_actions.js')
  }
]

```


* log: Raw http request information which you can use to perform logging(we use node-fetch to request http)

```
//auto support three types log which you can see in this function
info:{
  type:'http',
  message:{
    request:{url,body,headers},
    response:{status,body},
    error //also trigger in error_notice when it not be falsely
  }
}
info:{
  type:'webhook',
  message:{
    request:{url},
    error
  }
}
info:{
  type:'error_notice',
  message:{error} //only happen when we have http error and webhook verify fail
}

log:function(info){
  console.log(info.type,info.message.request.url);
}
```

you can also log information in callback function,we will inject botLog instance after you bind log in botConfig

```js
module.exports=function(req,res){
  let {botLog} = res.locals;
  botLog({//will call result in log function of botConfig.js
    type:'your log type',
    message:{
      error,
      ...
    }
  });
}
```


## Included Demos
If you wish to quickly see a demo of how to use this package.

Prereqs:

The demo runs DynamoDB, so you must have Java installed on your computer, as well as AWS CLI set up. To Install AWS CLI: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html
After installing, run `aws configure`, then configure your access key Id, and secret access key, (These two values don’t have to be real, can just be made up for local testing).

To run demo:
1. `zoomchatbot create app`
2. select general-demo-withdynamodb or serverless-demo-withdynamodb in terminal.
3. Set up your bot’s environment variables: .development.env for general mode, serverless.development.json for serverless mode, described in Setup & Features.
4. Set up your bot on marketplace.
5. Use `npm run dynamodb` to start dynamoDB.
6. Use `npm run start` to start your app
7. (Optional) If you wish to use the demo's "meet" command, you must go to your bot's marketplace page and under "Scopes", add the scope "meeting:write:admin".
8. Go to your bot's marketplace page and under "Local Test", click "Install".


## Database

useDatabase: Although you can write database code in your callback function directly, we also support database config in botConfig.js. Once you declare your database functions in useDatabases, we will auto inject an object called databaseModels into res.locals of each callback. databaseModels will contain the functions you defined in useDatabase. Below is an example code using DynamoDB as an example:(see [botdynamodb](https://www.npmjs.com/package/botdynamodb) for how to write a custom database package,see [botdblocal](https://www.npmjs.com/package/botdblocal) for how to create local dynamodb tables)


```
useDatabase: {
  lib: require('some database library for zoom bot'),
  option: {
  tables: {
    zoom: {
      tableName: 'zoomtable',
      hashKey: 'zoom_account_id',
      schema: {
        zoom_account_id: joi.string(),
        zoom_access_token: joi.string()
      }
    }
  },
  port: 8089,
  region: 'us-east-1'
  }
}
```

And will auto inject databaseModels into res.locals of each callback:
```
  let { zoomApp, zoomError, databaseModels } = res.locals;
  await databaseModels.zoom.save({
  zoom_account_id: accountId
});
```

## others

* we have already install node-fetch,you can use node-fetch for request http.
* app.js is a general express code,you can change it by yourself.
* in app.js we use [botservice])(https://www.npmjs.com/package/botservice) as core lib to consume botConfig.js
* in views we use hbs for html template.
* tests have a endpoint jest test code,you can run 'npm run test' to pass test and change it by your self.


## Need Support?
The first place to look for help is on our [Developer Forum](https://devforum.zoom.us/), where Zoom Marketplace Developers can ask questions for public answers.

If you can’t find the answer in the Developer Forum or your request requires sensitive information to be relayed, please email us at developersupport@zoom.us.
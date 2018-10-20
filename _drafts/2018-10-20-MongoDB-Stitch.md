---
title:  "Using MongoDB Stitch to Trim Belly Fat"
date:   2018-10-20 15:04:23
tags: [MongoDB, Stitch, NoSQL]
layout: post
---

Recently MongoDB debuted [Stitch](https://www.mongodb.com/cloud/stitch), a serveless platform for their database in the cloud service, [Atlas](https://www.mongodb.com/cloud/atlas). Stitch offers some neat features such database triggers, which run functions in response to a database event, and services, which allow you to easily integrate with 3rd-party applications. Today we're going to look at how easy it is to combine these two to create simple but powerful features.

What We\'re Building
-------------------------------

I\'ve set up a very basic HTML page to allow me to input my weight and store it in a MongoDB Atlas cluster. It\'s based closely on the [Basic Blog](https://docs.mongodb.com/stitch/tutorials/build-blog/) Stitch tutorial, except instead of storing comments, we\'re storing my weight measurements. What we\'re going to add to this existing project is a Stitch trigger that will alert a Slack channel if I log a weight over a pre-defined threshold. With any luck, this kind of public shaming will help me think twice before I have that second slice of pie.

![Slack]({{ "/images/slack.png" | prepend: site.baseurl }})

*For the purposes of this blog post I\'m going to assume you already have a working Stitch app for tracking your weight. Follow the [Basic Blog](https://docs.mongodb.com/stitch/tutorials/build-blog/) official Stitch tutorial to set one up if you haven't yet. The full code for my custom HTML page is available on my [Github](https://gist.github.com/pca2/399bf17084b25fbfa58dc0a5c364b43d) if you\'d like to follow along.*

Create a New Service
---------------------------------

The first thing we\'re going to do is create a new HTTP service so we can send messages to Slack via its API. Open up your Stitch Admin Console and select the **Services** option under the \"Control\" section of the left-hand navigation.

![sidepanel]({{ "/images/sidepanel.png" | prepend: site.baseurl }})

Once on the Services page, hit the **Add a Service** button to set up a new service. Select **HTTP** as the type, and name it. I\'m going to call mine \"wt\_http\_service.\" We won\'t be adding a database webhook or custom rules in this project, so leave those blank.

Create a New Slack Webhook
---------------------------------------

Before we can create a new function, we need to set up a new incoming webhook in Slack that will allow us to post to Slack channels. Follow the [official Slack tutorial](https://api.slack.com/tutorials/slack-apps-hello-world) for details on setting up a basic webhook. The key thing we need is the webhook URL, which will look something like this:

`https://hooks.slack.com/services/T15HQQKLC/CD8JFWr5SFN/B5nlagGNYLuwrwapvYzfrrvzHe`

Create a new Trigger
---------------------------------

It's time to set up the new database trigger that will run every time we insert a new weight measurement into the collection. Click on the **Triggers** option under the \"Atlas Clusters\" heading of the left-hand navigation.

On the next page, click the button to **Add Database Trigger**. Here are the relevant fields to fill out:

- Trigger Name: I\'ve named mine \"weightTrigger.\"
- Enabled: Should be active by default, but if not, activate it now.
- Database Name: This is the DB that contains your weight measurements. In my case it is \"Weight.\"
- Collection Name: This is the collection that contains your weight   measurements. In my case it is \"weights.\"
- Operation Type: Select \"Insert\" only.

It\'s now time to create a new function that will be run by the trigger.  In the **Linked Function** Section of the page select \"+ New Function.\" Doing this will open up an in-browser editor where you can name and write your new function.

Create a New Function
----------------------------------

I\'ve gone with \"checkWeight\" for the name of my function. Here\'s the
[function code](https://gist.github.com/pca2/399bf17084b25fbfa58dc0a5c364b43d#file-weightcheck-js):

```javascript
exports = function(changeEvent) {
   const http = context.services.get("wt_http_service");
   const newWeight = changeEvent.fullDocument.weight;
   const thresholdWeight = 145;
   const slackURL = 'https://hooks.slack.com/services/T8M......';
   const slackMsg = `Uh-oh a weight was posted of ${newWeight}, better lay off the pie`;
    
   if(newWeight < thresholdWeight){
     return "Weight under threshold";
   } else {
     return http
       .post({ url: slackURL, body: JSON.stringify({"text": slackMsg})})
       .then(resp => {
         return resp;
       })
       .catch(err => console.log(err) );
   }
};
```
Here\'s the sections of the code you will want to change:

- http service name: In my case it was \"wt\_http\_service.\"
- `slackURL`: Use the webhook URL you received from Slack.
- `thresholdWeight`: Set to whatever you feel is appropriate.

Once everything is filled out, click the **Save** Button.

Wrapping Up
------------------------

That should be it. Now when we enter a weight above our threshold, everyone in our choosen Slack channel will be notified and I\'ll promise to have a salad for lunch.

Obviously, we\'re just scratching the surface with this little project. Hopefully you now see how easy it is to combine triggers and service integrations to create really impactful features. Check out the [official tutorials](https://docs.mongodb.com/stitch/tutorials/) to see what other neat things you can bring to life with MongoDB Stitch.

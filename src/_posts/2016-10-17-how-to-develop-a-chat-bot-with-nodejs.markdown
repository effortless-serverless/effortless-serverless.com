---
layout: post
title:  "How To Develop A Chat Bot With Node.js"
date: 2016-10-17 12:00:00 +0200
categories: Claudiajs Chatbots
author_name : Slobodan Stojanović
author_url : /author/slobodan
author_avatar: slobodan.jpg
twitter_username: slobodan_
show_avatar: true
read_time: 7
feature_image: "https://effortless-serverless.com/img/serverless-migration/figure-2.jpg"
show_related_posts: false
square_related: recommend-slobodan
---

In the past few months, chat bots have become very popular, thanks to Slack, Telegram and Facebook Messenger. But the chat bot idea is not new at all.

![](/img/chatbot-post-cover.png)

A chat bot interface is mentioned in the famous Turing test in 1950. Then there was Eliza in 1966, a simulation of a Rogerian psychotherapist and an early example of primitive natural language processing. After that came Parry in 1972, a simulation of a person with paranoid schizophrenia (and, yes, of course, [Parry met Eliza](http://www.theatlantic.com/technology/archive/2014/06/when-parry-met-eliza-a-ridiculous-chatbot-conversation-from-1972/372428/)).

In 1983, there was a book named The Policeman’s Beard Is Half Constructed, which was generated by Racter, an artificial intelligence computer program that generated random English-language prose, later released as a chat bot.

One of the most famous was Alice (artificial linguistic Internet computer entity), released in 1995. It wasn’t able to pass the Turing test, but it won the [Loebner Prize](https://en.wikipedia.org/wiki/Loebner_Prize) three times. In 2005 and 2006, the same prize was won by two Jabberwacky bot characters.

And in 2014, Slackbot made chat bots popular again. In 2015, Telegram and then Facebook Messenger released chat bot support; then, in 2016 Skype did the same, and Apple and some other companies announced even more chat bot platforms.

## What Do You Need To Know To Build A Chat Bot?

The answer to that mostly depends on what you want to build, of course.

In most cases, you can build a chat bot without knowing much about artificial intelligence (AI), either by avoiding it completely or by using some existing libraries for basic AI.

The same goes for natural language processing (NLP); it’s more important than AI, but you can build a chat bot using an NLP library or, for some platforms, simply by using buttons and UI elements instead of word processing.

And finally, do you even need to know programming? There are a lot of visual bot builders, so probably not. But it can be useful.

## How To Build A Facebook Messenger Bot

This is an article about building chat bots, so let’s finally dive deep into it. Let’s build a simple Facebook Messenger bot.

We’ll use Node.js, but you can build a chat bot with any programming language that allows you to create a web API.

Why Node.js? Because it’s perfect for chat bots: You can build a simple API quickly with hapi.js, Express, etc.; it supports real-time messages (RTM) for Slack RTM bots; and it’s easy to learn (at least easy enough to build a simple chat bot).

Facebook already has a sample chat bot written in Node.js, [available on GitHub](https://github.com/fbsamples/messenger-platform-samples). If you check the code, you’ll see that it uses the Express framework and that it has three webhooks (for verification, authentication and receiving messages). You’ll also see that it sends responses with Node.js’ Request module.

Sounds simple?

It is. But this complete sample bot has 839 lines of code. It’s not much and you probably need just half of that, but it’s still too much boilerplate code to start with.

What if I told you that we could have the same result with just five lines of JavaScript?

```javascript
var botBuilder = require('claudia-bot-builder');

module.exports = botBuilder(function (request) {
  return 'Thanks for sending ' + request.text;
});
```

Or even fewer if you use ECMAScript 6:

```javascript
const botBuilder = require('claudia-bot-builder');

module.exports = botBuilder(request => `Thanks for sending ${request.text}`);
```

## Meet The Claudia Bot Builder

The Claudia Bot Builder helps developers create chat bots for Facebook Messenger, Telegram, Skype and Slack, and deploy them to Amazon Web Services’ (AWS) Lambda and API Gateway in minutes.

The key idea behind the project is to remove all of the boilerplate code and common infrastructure tasks, so that you can focus on writing the really important part of the bot — your business workflow. Everything else is handled by the Claudia Bot Builder.

Why AWS Lambda? It’s a perfect match for chat bots: Creating a simple API is easy; it responds much faster to the first request than a free Heroku instance; and it’s really cheap. The first million requests each month are free, and the next million requests are just $0.20!

Here’s how easy it is to build a Facebook Messenger bot with Claudia Bot Builder:

<iframe src="https://player.vimeo.com/video/170647056?title=0&byline=0&portrait=0" width="640" height="360" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/170647056">Create chat-bots easily using Claudia Bot Builder</a> from <a href="https://vimeo.com/user49229162">Gojko Adzic</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

## Let’s Build A Space Explorer Bot

Space Explorer is a simple Messenger chat bot that uses NASA’s API to get data and images about space.

Before we begin, create a Facebook page and app, and add Messenger integration, as described in Facebook’s [Getting Started](https://developers.facebook.com/docs/messenger-platform/quickstart) guide.

Then, create a file named bot.js with the following content:

```javascript
const botBuilder = require('claudia-bot-builder');

module.exports = botBuilder(request => `Hello from space explorer bot! Your request was: ${request.text}`);
```

Install these dependencies:

```javascript
npm init;

npm install claudia-bot-builder -S;

npm install claudia -g;
```

![Create chat-bots easily using Claudia Bot Builder]()

Create a Lambda function and follow the instructions in the video above to connect it with your Facebook app:

```shell
claudia create --region us-east-1 --api-module bot --configure-fb-bot
```

That’s it! You’ve created your first chat bot for Facebook Messenger.

If you send a message to your page, your bot will reply. But the answer is too simple. Let’s add something more interesting!

## Integrate NASA’s API

Before we continue, visit [NASA’s API portal](https://api.nasa.gov/) and get an API key.

Then, add your API key as a `nasaApiKey` stage variable in API Gateway. You can do that from the UI or by running the following command:

```shell
aws apigateway create-deployment \
    --rest-api-id API_ID --stage-name latest \
    --variables nasaApiKey=YOUR_NASA_API_KEY
```

Here, `API_ID` is your API ID from the claudia.json file that was auto-generated in the previous step.

Let’s add a better answer to the text messages. Claudia Bot Builder has a simple builder for Facebook Messenger template messages ([the documentation is on GitHub](https://github.com/claudiajs/claudia-bot-builder/blob/master/docs/FB_TEMPLATE_MESSAGE_BUILDER.md)).

```javascript
const botBuilder = require('claudia-bot-builder');
const fbTemplate = botBuilder.fbTemplate;
const rp = require('minimal-request-promise');

module.exports = botBuilder((request, originalApiRequest) => {
  // If request is not postback
  if (!request.postback)
    // We'll get some basic info about the user
    return rp.get(`https://graph.facebook.com/v2.6/${request.sender}?fields=first_name&access_token=${originalApiRequest.env.facebookAccessToken}`)
      .then(response => {
        const user = JSON.parse(response.body)
        // Then let's send two text messages and one generic template with three elements/bubbles
        return [
          `Hello, ${user.first_name}. Welcome to Space Explorer! Ready to start a journey through space?`,
          'What can I do for you today?',
          return new fbTemplate.generic()
            .addBubble(`NASA's Astronomy Picture of the Day`, 'Satellite icon by parkjisun from the Noun Project')
              .addImage('https://raw.githubusercontent.com/stojanovic/space-explorer-bot/master/assets/images/apod.png')
              .addButton('Show', 'SHOW_APOD')
              .addButton('What is APOD?', 'ABOUT_APOD')
              .addButton('Website', 'http://apod.nasa.gov/apod/')
            .addBubble(`Photos from NASA's rovers on Mars`, 'Curiosity Rover icon by Oliviu Stoian from the Noun Project')
              .addImage('https://raw.githubusercontent.com/stojanovic/space-explorer-bot/master/assets/images/mars-rover.png')
              .addButton('Curiosity', 'CURIOSITY_IMAGES')
              .addButton('Opportunity', 'OPPORTUNITY_IMAGES')
              .addButton('Spirit', 'SPIRIT_IMAGES')
            .addBubble('Help & info', 'Monster icon by Paulo Sá Ferreira from the Noun Project')
              .addImage('https://raw.githubusercontent.com/stojanovic/space-explorer-bot/master/assets/images/about.png')
              .addButton('About the bot', 'ABOUT')
              .addButton('Credits', 'CREDITS')
              .addButton('Report an issue', 'https://github.com/stojanovic/space-explorer-bot/issues')
            .get();
        ];
      });
}
```

Now our bot has a nice welcome answer:

![](/img/initial-chatbot.png)

Much better!

Next, we want to handle postbacks. Let’s start with NASA’s Astronomy Picture of the Day:

```javascript
// In case of the 'SHOW_APOD' postback, we'll contact NASA API and get the photo of the day.
if (request.text === 'SHOW_APOD')
  return rp(`https://api.nasa.gov/planetary/apod?api_key=${originalApiRequest.env.nasaApiKey}`)
    .then(response => {
      const APOD = JSON.parse(response.body)
      return [
        `NASA's Astronomy Picture of the Day for ${APOD.date}`,
        `"${APOD.title}", © ${APOD.copyright}`,
        new fbTemplate.image(APOD.url).get(),
        APOD.explanation,
        new fbTemplate.button('More actions:')
          .addButton('Download HD', APOD.hdurl)
          .addButton('Visit website', 'http://apod.nasa.gov/apod/')
          .addButton('Back to start', 'MAIN_MENU')
          .get()
      ]
    });
```

And here are the Mars rovers (Curiosity, Opportunity and Spirit):

```javascript
// Common API call
function getRoverPhotos(rover, sol, nasaApiKey) {
  // If sol (Mars day) is not defined, take a random one.
  if (!sol)
    sol = (parseInt(Math.random() * 9) + 1) * 100;

  // Contact the API
  return rp(`http://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/photos?sol=${sol}&api_key=${nasaApiKey}`)
    .then(response => {
      let rawBody = response.body;

      let roverInfo = JSON.parse('' + rawBody);
      // Create generic template with up to 10 photos.
      let photos = roverInfo.photos.slice(0, 10);
      let roverImages = new fbTemplate.generic();

      photos.forEach(photo => {
        return roverImages.addBubble(photo.rover.name, 'At ' + photo.earth_date + ' (sol ' + photo.sol + '), using ' + photo.camera.full_name)
          .addImage(photo.img_src)
          .addButton('Download', photo.img_src)
      });

      // Send the message.
      return [
        `${roverInfo.photos[0].rover.name} rover`,
        `Landing Date: ${roverInfo.photos[0].rover.landing_date} \nTotal photos: ${roverInfo.photos[0].rover.total_photos}`,
        roverImages.get(),
        new fbTemplate.button('More actions:')
          .addButton('Show newest photos', `PHOTOS_${rover}_${roverInfo.photos[0].rover.max_sol}`)
          .addButton('Visit Wikipedia', `https://en.wikipedia.org/wiki/${rover}_(rover)`)
          .addButton('Back to start', 'MAIN_MENU')
          .get()
      ];
    })
    .catch(err => {
      // If the selected sol doesn't have any photos, take the photos from sol 1000.
      console.log(err);
      return getRoverPhotos(rover, 1000, nasaApiKey);
    });
}

// Curiosity photos
if (request.text === 'CURIOSITY_IMAGES')
  return getRoverPhotos('curiosity', null, originalApiRequest.env.nasaApiKey);

// Opportunity photos
if (request.text === 'OPPORTUNITY_IMAGES')
  return getRoverPhotos('opportunity', null, originalApiRequest.env.nasaApiKey);

// Spirit photos
if (request.text === 'SPIRIT_IMAGES')
  return getRoverPhotos('spirit', null, originalApiRequest.env.nasaApiKey);

// Rover photos by sol (Mars day)
if (request.text.indexOf('PHOTOS_') === 0) {
  const args = request.text.split('_')
  return getRoverPhotos(args[1], args[2], originalApiRequest.env.nasaApiKey);
}
```

Finally, add some static content to the end:

```javascript
// About Astronomy Picture of the Day
if (request.text === 'ABOUT_APOD')
  return [
    `The Astronomy Picture of the Day is one of the most popular websites at NASA. In fact, this website is one of the most popular websites across all federal agencies. It has the popular appeal of a Justin Bieber video.`,
    `Each day a different image or photograph of our fascinating universe is featured, along with a brief explanation written by a professional astronomer.`,
    new fbTemplate.button('More actions:')
      .addButton('Show photo', 'SHOW_APOD')
      .addButton('Visit website', 'http://apod.nasa.gov/apod/')
      .addButton('Back to start', 'MAIN_MENU')
      .get()
  ];

// About the bot
if (request.text === 'ABOUT')
  return [
    `Space Explorer is simple Messenger chat bot that uses NASA's API to get the data and images about the space`,
    `It's created for fun and also as a showcase for Claudia Bot Builder, node.js library for creating chat bots for various platform and deploying them on AWS Lambda`,
    new fbTemplate.button('More actions:')
      .addButton('Claudia Bot Builder', 'https://github.com/claudiajs/claudia-bot-builder')
      .addButton('Source code', 'https://github.com/stojanovic/space-explorer-bot')
      .get()
  ];

// Finally, credits
if (request.text === 'CREDITS')
  return [
    'Claudia Bot Builder was created by Gojko Adžić, Aleksandar Simović and Slobodan Stojanović',
    'Icons used for the bot are from the Noun Project',
    '- Rocket icon by misirlou, \n- Satellite icon by parkjisun, \n- Curiosity Rover icon by Oliviu Stoian, \n- Monster icon by Paulo Sá Ferreira',
    'This bot was created by Claudia Bot Builder team',
    new fbTemplate.button('More actions:')
      .addButton('Claudia Bot Builder', 'https://github.com/claudiajs/claudia-bot-builder')
      .addButton('The Noun Project', 'https://thenounproject.com')
      .addButton('Source code', 'https://github.com/stojanovic/space-explorer-bot')
      .get()
  ];
```

## Result

After minor refactoring, our code should look something like the [source on GitHub](source on GitHub).

And here’s how our bot works:

<iframe src="https://player.vimeo.com/video/172001135" width="640" height="700" frameborder="0" webkitallowfullscreen mozallowfullscreen allowfullscreen></iframe>
<p><a href="https://vimeo.com/172001135">Space Explorer chat bot for FB Messenger using Claudia Bot Builder</a> from <a href="https://vimeo.com/user53636250">Slobodan Stojanović</a> on <a href="https://vimeo.com">Vimeo</a>.</p>

You can try it live on your page or on the [Space Explorer bot](https://m.me/space-explorer-bot) page on Facebook Messenger.

![Space Explorer Bot Messenger code](/img/messenger-code.png)

That’s it!

You’ve successfully built your first chat bot using Claudia Bot Builder. It was easy, wasn’t it?

Now go and build more cool chat bots.

---

Originally published on: [Smashing magazine](https://www.smashingmagazine.com/2016/10/how-to-develop-a-chat-bot-with-node-js/).

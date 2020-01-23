# Messaging Window JavaScript SDK

A JavaScript wrapper for the LiveEngage Messaging Window API.

## Introduction

This Messaging Window SDK for the LiveEngage [Messaging Window API](https://developers.liveperson.com/messaging-window-api-overview.html) will make building custom messaging windows and JavaScript applications efficient and stable. The SDK does a lot of the work of connecting to LivePerson's messaging servers, subscribing to notifications, and managing the conversation for you. It was developed by Robert Lester. The documentation is maintained by Eden Kupermintz.

### Prerequisites

This library requires only an active LiveEngage account #.

**Table of contents**

[Quick Start](#quick-start)<br>
[Available Methods](#available-methods)<br>
[Event Callbacks](#event-callbacks)<br>
[Sample Code](#sample-code)<br>

## Quick Start

### Include the Attached JavaScript File

Download the `message-window.min.js` file from this repository and include it in your project (a beautified version of this file is also included in this repository for your convenience).

```html
<head>
	<script src="messaging-window.min.js" type="text/javascript"></script>
	...
```

### Initializing the Library

Initialize the library by instantiating an object with the necessary options (see below).

```javascript
var windowKit = new windowKit({
	account: <your LivePerson account number here>,
	campaignId: 12341234,
	engagementId: 12341234,
	//skillId: 12341234 - or optional skill ID
});
```

### Connecting to LiveEngage

Call the `connect()` method to connect to LiveEngage **first** (if you don't, some of the callbacks won't be available when you run them and an error will be thrown).

```javascript
windowKit.connect();
```

## Available Methods

Once you have connected to LiveEngage, you are set to receive and send messages to the conversation. However, you will need to handle various standard actions and events by using the custom methods and callbacks as listed below (for example, when an agent sends a message to the visitor, this message will need to be displayed on the screen, an interface for the user to send a message back needs to be developed and so on).

### Library Methods

| Method | Parameters | Description |
| --- | --- | --- |
| [connect](#connecting-to-liveengage) | - | Connects the library to LiveEngage, see above |
| [sendMessage](#sendmessage) | message | Sends the specified text to the conversation. |
| `sendReadState` | state, ids | Sends the status of read or accept for a specified incoming message. The states are READ or ACCEPT. You can also use `windowKit.readStates.read` or `windowKit.readStates.accept` |
| [sendChatState](sendchatstate) | state | Sends the specified chat state to the conversation, values of COMPOSING and PAUSE are accepted. You can use `windowKit.chatStates.composing` or `windowKit.chatStates.accept` |


### sendMessage

This callback takes a string passed to it and sends it to LiveEngage and the agent handling the conversation. In the example below, we pass a simple hardcoded string but you will probably need to write some code to dynamically grab the user's input from wherever they're typing it (like an `input` element for example). See the sample code below for an example on how to achieve this.

```javascript
windowKit.sendMessage('Hello World!');
```

### sendChatState

This callback sends the current state of the conversation. Useful for when you'd like to display  "agent is typing" indicators in a way that isn't covered by LiveEngage's default states (for example, you'd like the agent to only be set to typing for their first message but not the second).

```javascript
windowKit.sendChatState(windowKit.chatStates.composing);
```

## Event Callbacks

These event callbacks will be fired whenever their corresponding event occurs within the UMS framework. You can use these events to dynamically handle parts of the conversation, e.g. display a typing indicator when the Agent Chat State changes.

| Event | Arguments | Description |
| --- | --- | --- |
| [onAgentTextEvent](#onagenttextevent) | text | Event is triggered when the agent sends a plain text message |
| [onAgentRichContentEvent](#onagentrichcontentevent) | content | Event is triggered when the agent sends a structured content message |
| [onAgentChatState](#onagentchatstate) | state | Event is triggerd whenever the agent transitions from one state to another, i.e starts or stops typing |
| `onTextReceived` | text, change_details | Event is triggered when a text-only message is received. |
| `onReceived` | change_details | Event triggered on all messages (sent or received). |

### onAgentTextEvent

This callback will listen to agent text events (plain text messages) and pass their content via the `text` parameter. You will need to grab that `text` and append it to an element on your page in order to display it.

```javascript
windowKit.onAgentTextEvent(function(text) {
	// append the text to an element on the page
	console.log("Agent: " + text);
});
```

### onAgentRichContentEvent

This callback will listen to agent rich content events (structured content messages) and pass their content via the `content` parameter. You will then need to render that structured content into HTML. One recommended way to do so is by using LivePerson's [structured content rendering tool](https://github.com/LivePersonInc/json-pollock) (shortly, you can call the above tool's `render` method on the `content` parameter passed by this function to render it into HTML). You can then style the rendered HTML using CSS.

```javascript
windowKit.onAgentRichContentEvent(function(content) {
  var structuredText = JsonPollock.render(content);
	// do something with the rendered content saved in the variable above, like appending it to an element on the page
});
```

### onAgentChatState

This callback is triggered when the agent starts or stops typing. The different states are represented by 'composing' (agent is typing) and 'pause' (agent has stopped typing). By listening to the parameter passed by this callback, you can write code to display or hide typing indicators.

```javascript
windowKit.onAgentChatState(function (state) {
	if (state == 'COMPOSING') {
		//show your agent is typing element
} else {
	//agent has stopped typing so
	//hide your agent is typing element
}
});
```

## Sample code

In this very simple use case for the SDK, we accomplish three things:

* First, we use the `onAgentTextEvent` callback to listen for agent text events. This allows us to grab the opening text of the conversation and display any future messages by the agent.

* Then, we also use the `onAgentRichContentEvent` to listen for agent rich content events, since, in this use case, the agent is a bot and will be utilizing multiple choice questions as menus.

* Lastly, we listen for user selections on the different structured content items presented by the agent. We grab the text of those items and send them back to the agent using the `sendMessage` method. In LiveEngage, we've configured the bot to listen for these textual responses and trigger the appropriate menu (upon which the bot sends a rich content message, grabbed by the method used above). We also append user selections to the HTML, to create a conversation type flow.

* We don't allow the user in this usecase to type back questions or answers to the bot. They can only use the structured content options given to them. If we wanted to allow free text, we would add code to grab the user's input and send it to the conversation dynamically.

**Note**: in a more complex example, we'd use a callback to render the text to the HTML instead of hard coding it directly as we do here. That is, we'd send the text to LiveEngage using a callback then listen to the event in LiveEngage. Only when the message is received in LiveEngage would we then grab its contents and append them to the DOM. If you render it directly as here, the text messages won't be "saved" as part of the LiveEngage conversation and won't appear when the user refreshes their window, for example, since they were simply hardcoded into the DOM. You can [check out LivePerson's Knowledge Center](https://knowledge.liveperson.com) for a more in-depth example of how this SDK was used to build a complex bot experience (the website's homepage is a bot built using this SDK by Eden Kupermintz).

```javascript
var windowKit = new windowKit({
	account: <your LivePerson account number here>
	//skillId: 12341234 - optional skill ID
});

//connect to LE
windowKit.connect();

//when the agent sends a text message
windowKit.onAgentTextEvent(function(text) {
	//append the message's contents to the DOM
	$('#your_bot_container_here').append('<div class="agentText">' + text + '</div>');
	//grab all the agent texts so far
	var botTexts = document.getElementsByClassName('agentText');
	//find the last one
	var latestText = botTexts[botTexts.length - 1]
	//scroll the window to the last text. This is used to create a scroll effect in the conversation.
	$('body, html').animate({ scrollTop: $(latestText).offset().top }, 1000);
	console.log('Agent: ' + text);
});

//when the agent sends a rich content message
windowKit.onAgentRichContentEvent(function(content) {
	//render the structured content using JsonPollock
  var structuredText = JsonPollock.render(content);
	//append the results of the render to the DOM
	$('#your_bot_container_here').append(structuredText);
	//next three rows create the same scrolling effect as above
	var botTextsSC = document.getElementsByClassName('lp-json-pollock');
	var latestSC = botTextsSC[botTextsSC.length - 1];
	$('body, html').animate({ scrollTop: $(latestSC).offset().top }, 1000);
	console.log('Agent: ', structuredText);
	//when a user clicks on a structured content button
	$('.lp-json-pollock-element-button').on('click', function () {
		//grab the text of the button
		var scText = $(this).text();
		//send the text to LE for the bot to process
		windowKit.sendMessage(scText);
		//append the text to the DOM so it shows up as the user's side of the conversation
		$('#your_bot_container_here').append('<div class="consumerText">' + scText + '</div>');
		//same scroll effect as above
		var consumerTexts = document.getElementsByClassName('consumerText');
		var latestConsumerText = consumerTexts[consumerTexts.length - 1];
		$('body, html').animate({ scrollTop: $(latestConsumerText).offset().top }, 1000);
	});
});
```

### Licensing

All usage of the contents, documentation or code found in this repository is subject to the [LivePerson API Terms of Use](https://www.liveperson.com/policies/apitou). Please use the link above to read them carefully before utilizing the SDK.

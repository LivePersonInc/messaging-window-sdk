# Messaging Window JavaScript SDK

A JavaScript wrapper for the LiveEngage Messaging Window API.

## Introduction

This JavaScript SDK for the LiveEngage Messaging Window API will make building custom messaging windows and javascript applications efficient and stable.

This library requires only an active LiveEngage account #.

## Quick Start

### Include the Attached JavaScript File in Your <head>

```html
<head>
	<script src="js/messaging-window.min.js" type="text/javascript"></script>
	...
```

Initialize the library by instantiating an object with the necessary options (see below). Call the `connect()` method to connect to LiveEngage **first** (if you don't, some of the callbacks won't be available when you run them and an error will be thrown). Then, handle various LiveEngage actions and events by using the custom callbacks as listed below.

### Initializing the Library

```javascript
var windowKit = new windowKit({
	account: 12341234
	//skillId: 12341234 - optional skill ID
});
```

### Connecting to LiveEngage

```javascript
windowKit.connect();
```

## Available Methods

### Library Methods

| Method | Parameters | Description |
| --- | --- | --- |
| [connect](#connecting-to-liveengage) | - | Connects the library to LiveEngage |
| `[sendMessage](#sendmessage)` | message | Sends the specified text to the conversation. |
| `sendReadState` | state, ids | Sends the status of read or accept for a specified incoming message. The states are READ or ACCEPT. You can also use `windowKit.readStates.read` or `windowKit.readStates.accept` |
| `[sendChatState](sendchatstate)` | state | Sends the specified chat state to the conversation, values of COMPOSING and PAUSE are accepted. You can use `windowKit.chatStates.composing` or `windowKit.chatStates.accept` |


### sendMessage

This call back takes a string passed to it and sends it to LiveEngage and the agent handling the conversation. In the example below, we pass a simple, hardcoded string but you will probably need to write some code to grab the user's input from wherever they're typing it (like an `input` element for example).

```javascript
windowKit.sendMessage('Hello World!');
```

### sendChatState

This callback sends the current state of the conversation. Useful for when you'd like to display  "agent is typing" indicators.

```javascript
windowKit.sendChatState(windowKit.chatStates.composing);
```

### Event Callbacks

| Event | Arguments | Description |
| --- | --- | --- |
| [onAgentTextEvent](#onAgentTextEvent) | text | Event is triggered when the agent sends a plain text message |
| [onAgentRichContentEvent](#onAgentRichContentEvent) | content | Event is triggered when the agent sends a structured content message |
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

This callback will listen to agent rich content event (structured content messages) and pass their content via the `content` parameter. You will then need to render that structured content into HTML. One recommended way to do so is by using LivePerson's [structured content rendering tool](https://github.com/LivePersonInc/json-pollock) (shortly, you can call the above tool's `render` method on the `content` parameter passed by this function to render it into HTML). You can then style the rendered HTML using CSS.

```javascript
windowKit.onAgentRichContentEvent(function(content) {
  var structuredText = JsonPollock.render(content);
	// do something with the rendered content saved in the variable above
});
```

### Sample code

In this very simple use case for the SDK, we accomplish three things:

* First, we use the `onAgentTextEvent` callback to listen for agent text events. This allows us to grab the opening text of the conversation and display any future answers by the agent.

* Then, we also use the `onAgentRichContentEvent` to listen for agent rich content events, since, in this use case, the agent is a bot and will be utilizing multiple choices questions as menus.

* Lastly, we listen for user selections on the different structured content items presented by the agent. We grab the text of those items and send them back to the agent using the `sendMessage` method. In LiveEngage, we've configured the bot to listen for these textual responses and trigger the appropriate menu (upon which the bot sends a rich content message, grabbed by the method used above). We also append user selections to the HTML, to create a conversation type flow. 

**Note**: in a more complex example, we'd use a callback to render the text to the HTMl instead of hard coding it directly as we do here. If you render it directly as here, the text messages won't be "saved" as part of the LiveEngage conversation and won't appear when the user refreshes their window, for example, since they were simply hardcoded into the DOM.

```javascript
var windowKit = new windowKit({
	account: <your account here>
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

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

### onAgentTextEvent

This callback will listen to agent text events (plain text messages) and pass their content via the `text` parameter. You will need to grab that `text` and append it to an element on your page in order to display it.

```javascript
windowKit.onAgentTextEvent(function(text) {
	// append the text to an element on the page
	console.log("Agent: " + text);
});
```

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

## Available Methods

### Library Methods

| Method | Parameters | Description |
| --- | --- | --- |
| `connect` | - | Connects the library to LiveEngage |
| `sendMessage` | message | Sends the specified text to the conversation. |
| `sendReadState` | state, ids | Sends the status of read or accept for a specified incoming message. The states are READ or ACCEPT. You can also use `windowKit.readStates.read` or `windowKit.readStates.accept` |
| `sendChatState` | state | Sends the specified chat state to the conversation, values of COMPOSING and PAUSE are accepted. You can use `windowKit.chatStates.composing` or `windowKit.chatStates.accept` |


### Event Callbacks

| Event | Arguments | Description |
| --- | --- | --- |
| `onTextReceived` | text, change_details | Event is triggered when a text-only message is received. |
| `onReceived` | change_details | Event triggered on all messages (sent or received). |

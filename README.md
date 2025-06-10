# Deck App Websocket Protocol

Deck app protocol using an RPC schema to interface with SAMMI decks. This is only intended for developers wanting to make third-party tools for SAMMI, and not typical SAMMI users! 

## Introduction

Requests and responses are defined by their Operation (`op`) codes included in the payload. Check [Codes](#codes) the protocol expects to send and recieve.

To keep the connection alive, you need to listen for op code `1`, which is a ping pong event. when recieved, respond with the same payload back.

Some requests that return a response allow you to include an `id` along with their payloads, which accepts a string and will include it with the response to help you identify what response goes where. If you dont need to know the response to your request you can ommit adding an id.

## Codes

| Code | Name | Client Action | Description |
| --- | --- | --- | --- |
| 0 | Hello | Receive | First message sent from SAMMI immediately on client connection. |
| 1 | Ping Pong | Send/Receive | Fired periodically to keep the connection alive. |
| 2 | Identify | Send | Response to Hello message. Should contain app name and authentication if required. |
| 3 | Identified | Receive | Client is now identified and ready to start sending and receiving data. |
| 4 | Request | Send | Request to SAMMI. |
| 5 | Request Response | Receive | A response to client's op 4 request. |
| 6 | Event | Receive | An event from SAMMI. |
| 7 | Close Event | Send/Receive | Client or SAMMI is closing connection. |

As GMS is unable to send a specific close code when it terminates the websocket connection, the received regular payload will be in the following format before the connection is terminated from the server side. The client should close the connection on their own when it receives the following message:

```
{			
	"op": 7, 
	"errorCode": {int} // one of the error codes from the table above
}
```

| Close Code | Description | Explanation |
| --- | --- | --- |
| 4000 | Unknown error. | Something went wrong and the connection was closed. Try reconnecting. |
| 4001 | Unknown op code. | You sent an invalid or no op code. |
| 4002 | Decode error. | You sent an invalid payload. |
| 4003 | Not authenticated | You sent a payload prior to identifying. |
| 4004 | Authentication failed. | You sent the wrong authorization string. |
| 4005 | Closed by the client. | The connection was closed by the client. |
| 4006 | Closed by SAMMI. | The connection was closed by SAMMI, i.e. when user closes the application. |
| 4007 | Connection timed out. | The connection timed out due to not receiving heartbeat from the client. |


## Structures

op 0 Hello JSON Structure:

```
{			
	"op": 0,			
	"data": {			
			"sammiVersion": {string},
			"rpcVersion": {string}, // current protocol version			
			"authRequired": {bool},			
			"salt": {string}, // only if auth is required			
			"challenge": {string} // only if auth is required			
	}
}
```

op 1 Ping Pong JSON Structure:

```
{			
"op": 1			
}
```

op 2 Identify JSON Structure:

```
{			
	"op": 2,
	"id": {int},			
	"data": {			
			"clientName": {string},			
			"authentication": {string} // only if authRequired is true
	}			
}
```

op 3 Identified JSON Structure:

```
{	
"op": 3,
"id": {int}		
}
```

op 4 Request JSON Structure:

```
{			
	"op": 4,			
	"id": {int},	
	"data": {			
			"requestName": {string},			
			"requestData": {obj}			
	}			
}
```

op 5 Request Response JSON Structure:

```
{		
	"op": 5,		
	"id": {int}		
	"data": {		
			"requestName": {string},		
			"requestSuccess": {bool},		
				"responseData": {obj} // will only contain responseData.error object if request was not successful		
	}		
}
```

Error object:

```
{
    "errorCode": {int},
    "errorMessage": {string}
}
```

op 6 Event JSON Structure:

```
{	
	"op": 6	
	"data": {	
			"eventType": {string},	
			"eventData": {obj}	
	}	
}
```

## Requests

### Get Deck List

Request
```
data: 
{
    "requestName": "GetDeckList"	
}
```

Response
```
responseData: 
{
    "deckList": [  // array containing deck objects
        { "deckName": {string}, "deckId": {string}, "crc": {int}, "status" {bool}" },
        { "deckName": {string}, "deckId": {string}, "crc": {int}, "status" {bool}" },
    ]
}
```

### Get Deck

Request
```
data: 
{
    "requestName": "GetDeck",			
    "requestData": {
        "deckId": {string},  // accepts either, prioritizes deckId
        "deckName": {string} // accepts either

    }		
}
```

Response
```
{
    "deckData": {object}  // an object containing deck data
}
```

### Get Image

Request
```
data:
{
    "requestName": "GetImage",			
    "requestData": {
        "fileName": {string}  // image filename, looks in the img folder
    }		
}
```

Response
```
{
    "fileName": {string},  // image filename we requested
    "imageData": {base64 encoded image data}
}
```

### Get Sum

Request
```
data:
{
    "requestName": "GetSum",			
    "requestData": {
        "name": {string}  // deck ID or file name
    }		
}
```

Response
```
{
    "sum": {string}  // checksum hash value
}
```

### Trigger Button
Request
```
data:
{
    "requestName": "TriggerButton",			
    "requestData": {
        "buttonId": {string}
    }		
}
```

Response
```
{}
```

### Release Button
Request
```
data:
{
    "requestName": "ReleaseButton",			
    "requestData": {
        "buttonId": {string}
    }		
}
```

Response
```
{}
```

### Get Button Modifications

Request
```
data:
{
    "requestName": "GetModifications"				
}
```

Response
```
{
    "modifications": {object}  // modified button list
}
```

### Get Ongoing Buttons

Request
```
data:
{
    "requestName": "GetOngoingButtons"				
}
```
Response
```
{
    "buttons": [  // array containing ongoing button objects
        { 
            "buttonId": {string}, 
            "groupId": {string}, 
            "duration": {int},
            "releaseType": {bool},
            "overlappable": {bool},
            "elapsedTime": {int}         
        }
    ]
}
```

### Get Deck Status

Request
```
data:
{
    "requestName": "GetDeckStatus",
    "requestData": {
        "deckId": {string}
    }    				
}
```

Response
```
{
    "status": {bool}
}
```

### Request Response Error Codes:

Fired when `data.requestSuccess` is `false` and `data.responseData` only contains `error` object:

```
{
    "errorCode": {int},
    "errorMessage": {string}
}
```

| Error Code | Description |
| --- | --- |
| 100 | Unspecified error. |
| 101 | Request name is missing. |
| 102 | Request parameter is missing. |
| 103 | Incorrect parameter type. |
| 104 | Unsupported request. |
| 105 | Deck with the ID or name not found. |
| 106 | Requested file not found. |

## Events

### Button was triggered or released

```
{
"eventType": "ReleaseTriggered" || "ButtonTriggered",
"eventData": { 
    "buttonId": {string},
    "groupId": {string},
    "overlappable": {bool},
    "duration": {number}
    }	
}
```

### Button has ended or released button has ended

```
{
    "eventType": "ReleaseEnded" : "ButtonEnded",
    "eventData": { 
        "buttonId": {string},
        "groupId": {string},
        "overlappable": {bool}
    }	
}
```

### Deck has been updated

```
{
    "eventType": "DeckUpdated",
    "eventData": { 
        "deckData": {object} // an object containing full, updated deck's data. includes all buttons, their triggers, commands, everything. this needs to only return shallow buttons, not nested data in them. Fixme.
    }	
}
```

### Deck has been added

```
{
    "eventType": "DeckAdded",
    "eventData": { 
        "deckData": {
					"deckName": {object} // an object containing full added deck data. button list array empty.
				}  	
    }	
}
```

### Deck has been removed

```
{
    "eventType": "DeckRemoved",
    "eventData": { 
        "deckData": {
					"deckId:" {string},
					"deckName": {string}
				}  	
    }	
}
```

### Decks order has changed

```
{
    "eventType": "DecksOrderChanged",
    "eventData": { 
	    "deckData": [  // array containing deck objects reflecting their new order
	        { "deckName": {string}, "deckId": {string}, "crc": {int} },
	        { "deckName": {string}, "deckId": {string}, "crc": {int} },
	    ] 	
    }	
}
```

### Deck status has changed

```
{
    "eventType": "DeckStatusChanged",
    "eventData": { 
        "deckId": {string}, 
        "state": {bool}
    }	
}
```

### Button has been modified

```
{
    "eventType": "ButtonModified",
    "eventData": { 
        "buttonId": {string},
        "modifications": {
            "text": {string},
            "image": {string},
            "border": {}
        }
    }	
}
```

### SAMMI was reset

```
{
    "eventType": "SAMMIReset",
    "eventData": {}
}
```

### Command "Wait For Input" was ran, awaiting input

```
{
    "eventType": "WaitForInput",
    "eventData": {
        "commandName": {string},
        "requestId": {string},
        "instanceId": {string/number}, ?
        "buttonId": {string},
        "variableName": {string},
        "timeoutAfter": {number},
        "message": {string},
        "choices": {Array}, //looks like it sends array if exists, but sends a string of an empty one if not...??? probably bug fix
        "defaultInput": {string}
    }
}
```

### Command "Send JSON" was ran
contains custom user defined payload content

```
{
    "eventType": "SendJSON",
    "eventData": {
        "event": {string}, //user defined
        "json": {string}
    }
}
```

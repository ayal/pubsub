# pubsub

uses a WIX website in iframe for free pubsub service

To try it, open in two (or more) windows and send messages between them

https://ayal.github.io/pubsub/?room=wixmix

# embedded WIX website - the pubsub service provider

https://ayalg5.wixsite.com/pubsub

Implements pubsub using corvid realtime api:

## code

### page code for subscribe:

```javascript
import wixRealtime from 'wix-realtime';
import wixLocation from 'wix-location';
import { pubsub } from 'backend/realtime.jsw'
import wixWindow from 'wix-window';

const query = wixLocation.query;
let room = query.room || 'world';

$w.onReady(function () {
	console.log('wix pubsub ready');
	const channel = { "name": room };
	console.log('will subscribe to', channel);
	wixRealtime.subscribe(channel, (message, sourceChannel) => {
		console.log('got message on realtime, sending to parent window',
			message, 'from', sourceChannel);
		wixWindow.postMessage({ pubsub: message })
	});
});
```

### backend code for publish (realtime.jsw):

```javascript
import wixRealtime from 'wix-realtime-backend';

export async function pubsub(room, message) {
	console.log('will publish message to room', message, room);
	const channel = { name: room };
	return wixRealtime.publish(channel, message, { includePublisher: true });
}
```

#### http endpoint for publish (http functions):

```javascript
import {ok, badRequest} from 'wix-http-functions';
import { pubsub } from 'backend/realtime.jsw'

export function get_pubsub(request) {
	console.log('got a GET request', request.query);
	pubsub(request.query.room, request.query.message);
	const response = {
		"headers": {
			"Content-Type": "application/json",
			"Access-Control-Allow-Origin": "*"
		}
	};
	return ok(response);
}
```


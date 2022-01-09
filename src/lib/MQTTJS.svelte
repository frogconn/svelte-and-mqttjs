<script>
	import { onMount } from 'svelte';

	let _clientID;
	let client;
	onMount(() => {
		let script = document.createElement('script');
		script.src = 'https://unpkg.com/mqtt/dist/mqtt.min.js';
		document.head.append(script);

		script.onload = function () {
			// init
			const clientId = 'mqttjs_' + Math.random().toString(16).substr(2, 8);
			_clientID = clientId;

			const host = 'ws://broker.emqx.io:8083/mqtt';

			const options = {
				keepalive: 60,
				clientId: clientId,
				protocolId: 'MQTT',
				protocolVersion: 4,
				clean: true,
				reconnectPeriod: 1000,
				connectTimeout: 30 * 1000,
				will: {
					topic: 'WillMsg',
					payload: 'Connection Closed abnormally..!',
					qos: 0,
					retain: false
				}
			};

			console.log('Connecting mqtt client');
			client = mqtt.connect(host, options);

			client.on('error', (err) => {
				console.log('Connection error: ', err);
				client.end();
			});

			client.on('connect', () => {
				console.log('Client connected:' + clientId);
				// Subscribe
				client.subscribe('testtopic', { qos: 0 });
			});

			client.on('reconnect', () => {
				console.log('Client connected:' + clientId);
				// Subscribe
				client.subscribe('testtopic', { qos: 0 });
			});

			client.on('message', (topic, message, packet) => {
                console.log('on message');
				console.log('Received Message: ' + message.toString() + '\nOn topic: ' + topic);
			});

			// _client = client;
		};
	});

	function publish() {
        console.log('on publish');
		client.publish('testtopic', 'ws connection demo...!', { qos: 0, retain: false });
	}
</script>

<div>MQTT</div>
<div>ClientID : {_clientID}</div>
<button on:click={publish}> Publish</button>

<style>
</style>

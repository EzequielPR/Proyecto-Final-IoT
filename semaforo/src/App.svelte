<script>
	let count = 58;
	let running = 0;
	const format2Digit = (n) => n.toLocaleString('en-US', {minimumIntegerDigits: 2});
	const client = mqtt.connect('wss://broker.emqx.io:8084/mqtt');

	$: rc = format2Digit(120 - count);
	$: yc = format2Digit(62 - count);
	$: gc = format2Digit(58 - count);

	const funciones = {
		'epr-smfr/cruce': ({d}) => {
			if (58-count < 0) return;
			count = count + Math.floor((58-count)/2);
		},
		65: () => {
			client.publish('epr-smfr/cuenta', JSON.stringify({c: 51}));
		},
		0: () => client.publish('epr-smfr/color', JSON.stringify({c: 'verde'})),
		58: () => client.publish('epr-smfr/color', JSON.stringify({c: 'amarillo'})),
		62: () => client.publish('epr-smfr/color', JSON.stringify({c: 'rojo'})),
	};

	client.on('connect', () => {

		client.subscribe('epr-smfr/cruce', {qos: 1});

		client.on('message', (topic, payload) => {
			funciones[topic]?.(JSON.parse(payload.toString()));
		});

		running = setInterval(() => {
			count = count == 120 ? 0 : count + 1;
			funciones[count]?.();
		}, 1000);
	});
</script>


<main>
	<div id="luces">
		<div style="--c: red;"
			class:on={count >= 62}
		>
			<p class="tx">{rc}</p>
			<p class="bg">88</p>
		</div>

		<div style="--c: yellow;"
			class:on={count >= 58 && count < 62}
		>
			<p class="tx">{yc}</p>
			<p class="bg">88</p>
		</div>

		<div style="--c: lightgreen;"
			class:on={count < 58}
		>
			<p class="tx">{gc}</p>
			<p class="bg">88</p>
		</div>
	</div>
	<br />

	<h1>Semaforo
		<span class:on={running}>{running ? 'ON' : 'OFF'}</span>
	</h1>
</main>


<style>
	h1 {
		text-align: center;
	}
	h1 > span {
		padding: 4px;
		border-radius: 8px;
		background-color: red;
	}
	h1 > span.on {
		background-color: lightgreen;
	}

	div#luces {
		width: calc(100vw - 20px);
		background-color: #202020;
		display: flex;
		padding: 20px;
		gap: 20px;
		border-radius: 25px;
		border: 15px solid orange;
	}

	div#luces > div {
		flex-basis: 0;
		flex-grow: 1;
		aspect-ratio: 1 / 1;
		background-color: black;
		border-radius: 50%;
		display: grid;
		place-items: center;
	}
	div#luces > div.on {
		/* background-color: black;
		color: var(--c); */
		background-color: var(--c);
		color: black;
	}

	div#luces > div > p {
		font-family: 'DSEG7Classic';
		font-size: 15vw;
		display: none;
	}
	div#luces > div.on > p {
		display: block;
		opacity: 0.75;
	}
	div#luces > div > p.bg {
		position: absolute;
		opacity: 0.03;
	}
</style>

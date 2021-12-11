<script>
	let count = 0;
	let running = 0;
	const format2Digit = (n) => n.toLocaleString('en-US', {minimumIntegerDigits: 2});
	const client = mqtt.connect('wss://broker.emqx.io:8084/mqtt');
	const speech = new SpeechSynthesisUtterance();

	speech.rate = 5;
	speech.voice = speechSynthesis.getVoices()[0];
	speech.volume = speech.pitch = 1;

	const funciones = {
		'epr-smfr/cuenta': ({c}) => {
			count = c;
			running = setInterval(() => {
				count--;
				sayNumber();
				funciones[count]?.();
			}, 1000);
		},
		0: () => {
			speechSynthesis.cancel();
			clearInterval(running);
			running = 0;
		}
	};

	client.on('connect', () => {
		client.subscribe('epr-smfr/cuenta', {qos: 1});

		client.on('message', (topic, payload) => {
			funciones[topic]?.(JSON.parse(payload.toString()));
		});
	});

	function sayNumber() {
		speechSynthesis.cancel();
		speech.text = count;
		speechSynthesis.speak(speech);
	}
</script>


<main>
	<div id="contador">
		<div class:on={count > 0}>
			<p class="tx">{format2Digit(count)}</p>
			<p class="bg">88</p>
		</div>
	</div>
	<br />

	<h1>Contador
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

	div#contador {
		margin: auto;
		aspect-ratio: 1 / 1;
		height: 80vmin;
		background-color: #202020;
		display: flex;
		padding: 40px;
		gap: 20px;
		border-radius: 25px;
	}

	div#contador > div {
		aspect-ratio: 1 / 1;
		background-color: black;
		border-radius: 25px;
		display: grid;
		place-items: center;
	}
	div#contador > div.on {
		/* background-color: black;
		color: var(--c); */
		/* background-color: var(--c); */
		color: lightgreen;
	}

	div#contador > div > p {
		font-family: 'DSEG7Classic';
		font-size: 40vmin;
		display: none;
	}
	div#contador > div.on > p {
		display: block;
		opacity: 0.75;
	}
	div#contador > div > p.bg {
		position: absolute;
		opacity: 0.03;
	}
</style>

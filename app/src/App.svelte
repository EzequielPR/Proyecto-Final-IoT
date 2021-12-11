<script>
	import Swal from 'sweetalert2';
	let count = 0;
	let pending = 0;
	let sent = false;
	let running = 0;
	let distance = 3;
	let color = 'verde';
	let cruces = localStorage.getItem('cruces') ?? 0;
	const client = mqtt.connect('wss://broker.emqx.io:8084/mqtt');
	const speech = new SpeechSynthesisUtterance();

	speech.rate = 8;
	speech.voice = speechSynthesis.getVoices()[0];
	speech.volume = speech.pitch = 1;

	$: state = (!pending && !running && !sent && `🚦 semaforo\na ${distance} mts`)
							|| (pending && '⌛ Enviando señal de cruce ')
							|| (running && '🟢 Ya puede cruzar!')
							|| (sent && '🔴 Espere a la luz roja');
	// $: estatus = {
	// 	'verde': '',
	// 	'amarillo': '',
	// 	'rojo': '',
	// };

	const funciones = {
		'epr-smfr/cuenta': ({c}) => {
			if (!sent) return;
			count = c;
			running = setInterval(() => {
				count--;
				sayNumber();
				navigator.vibrate([100]);
				funciones[count]?.();

				if (count == 0) stopCount();
			}, 1000);
		},
		'epr-smfr/color': ({c}) => {
			color = c;
		},
		50: () => navigator.vibrate([400]),
		40: () => navigator.vibrate([400]),
		30: () => navigator.vibrate([400]),
		20: () => navigator.vibrate([400]),
		10: () => navigator.vibrate([400]),
		9: () => navigator.vibrate([150, 100, 150]),
		8: () => navigator.vibrate([150, 100, 150]),
		7: () => navigator.vibrate([150, 100, 150]),
		6: () => navigator.vibrate([150, 100, 150]),
		5: () => navigator.vibrate([150, 100, 150]),
		4: () => navigator.vibrate([150, 100, 150]),
		3: () => navigator.vibrate([150, 100, 150]),
		2: () => navigator.vibrate([150, 100, 150]),
		1: () => navigator.vibrate([1000]),
	};

	client.on('connect', () => {

		client.subscribe('epr-smfr/cuenta', {qos: 1});

		client.on('message', (topic, payload) => {
			funciones[topic]?.(JSON.parse(payload.toString()));
		});
	});

	function stopCount() {
		clearInterval(running);
		count = 0;
		running = 0;
		sent = 0;
		cruces++;
		localStorage.setItem('cruces', cruces);
	}

	function handleCruceStart() {
		if (running || pending || sent) return;

		pending = setTimeout(() => {
			client.publish('epr-smfr/cruce', JSON.stringify({d: 5}));
			sent = true;
			pending = 0;
		}, 5000);
	}

	function handleCruceCancel() {
		if (!pending) {
			if (running) stopCount();
			return;
		}


		clearInterval(pending);
		pending = 0;
	}

	function sayNumber() {
		speechSynthesis.cancel();
		speech.text = count;
		speechSynthesis.speak(speech);
	}

	function showStats() {
		Swal.fire({
			title: 'Estadisticas',
			html: `
				Número de cruces => ${cruces}<br>
				Metros cruzados => ${cruces * 2}<br>
				Pasos caminados => ${cruces * 20}<br>
			`
		});
	}
</script>


<main>
	<button id="estadisticas"
		on:click={showStats}
	>Estadisticas</button>

	<div id="cuenta">
		<p id="primary" class:on={count > 0}>{count}</p>
		<p id="secondary">{state}</p>
	</div>

	<div id="botones">
		<button style="--c: lightgreen"
			class:on={!running && !sent && !pending}
			on:click={handleCruceStart}
		>
			Cruzar
		</button>
		<button style="--c: red"
			class:on={(!running && pending) || running}
			on:click={handleCruceCancel}
		>
			Cancelar
		</button>
	</div>
</main>


<style>
	:global(body) {
		background-color: cornflowerblue;
	}

	button#estadisticas {
		z-index: 1;
		position: absolute;
		top: 10px;
		right: 10px;
		aspect-ratio: 3 / 1;
		border-radius: 5vh;
		border: none;
		padding: 0px 6px;
		background-color: yellow;
		color: black;
		font-size: 2vh;
		box-shadow: 4px 4px 2px 0px #00000020;
	}
	button#estadisticas:active {
		box-shadow: none;
	}

	main, div {
		position: relative;
		display: flex;
		flex-direction: column;
	}

	main {
		height: 100vh;
		aspect-ratio: 9 / 16;
		margin: auto;
		background-color: cornflowerblue;
	}

	#cuenta {
		flex-grow: 1;
		text-align: center;
		padding: 20px;
	}

	#cuenta > #primary {
		margin-top: 4vh;
		font-size: 38vh;
		line-height: 34vh;
		visibility: hidden;
		flex-grow: 1;
	}
	#cuenta > #primary.on {
		visibility: visible;
	}
	#cuenta > #secondary {
		font-size: 5vh;
		white-space: pre-wrap;
		/* line-height: 5vh; */
	}

	#botones {
		gap: 20px;
		padding: 20px;
	}

	#botones > button {
		aspect-ratio: 3 / 1;
		border-radius: 5vh;
		border: none;
		background-color: lightgray;
		font-size: 10vh;
		color: gray;
	}
	#botones > button.on {
		box-shadow: 6px 6px 2px 0px #00000020;
		background-color: var(--c);
		color: black;
	}
	#botones > button.on:active {
		box-shadow: inset 8px 8px 2px 0px #00000020;
	}
</style>

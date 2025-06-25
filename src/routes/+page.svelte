<script lang="ts">
	import { onMount } from "svelte";
	import { goto } from "$app/navigation";

	let listening = $state(false);
	let transcript = $state("");
	let recognition = $state(null);

	$effect(() => {
		$inspect(transcript);
		$inspect(recognition);
	});

	onMount(() => {
		const SpeechRecognition =
			window.SpeechRecognition ||
			window.webkitSpeechRecognition;
		if (SpeechRecognition) {
			recognition = new SpeechRecognition();
			recognition.continuous = false;
			recognition.lang = "id-ID";
			recognition.interimResults = false;

			recognition.onresult = (event) => {
				let currentTranscript = "";
				for (
					let i = event.resultIndex;
					i < event.results.length;
					++i
				) {
					if (event.results[i].isFinal) {
						currentTranscript +=
							event.results[i][0]
								.transcript;
					}
				}
				transcript = currentTranscript.toLowerCase();

				// Simple navigation commands
				if (transcript.includes("go to about")) {
					goto("/about");
				} else if (transcript.includes("go home")) {
					goto("/");
				}
			};

			const toggleListen = () => {
				if (listening) {
					recognition.stop();
					listening = false;
				} else {
					recognition.start();
					listening = true;
				}
			};

			// Expose the toggle function to the template
			window.toggleListen = toggleListen;
		} else {
			alert(
				"Speech Recognition not supported in this browser.",
			);
		}
	});
</script>

<h1>Welcome to SvelteKit</h1>
<p>
	Visit <a href="https://svelte.dev/docs/kit">svelte.dev/docs/kit</a> to read
	the documentation
</p>

<a href="/about">go to about</a>

<button onclick={() => window.toggleListen()}>
	{listening ? "Stop Listening" : "Start Listening"}
</button>

<p>Transcript: {transcript}</p>

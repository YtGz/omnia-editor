<script lang="ts">
	import { OmniaEditor } from '$lib';

	let editor: OmniaEditor;
	let editorCount: OmniaEditor;

	let value =
		'<p>Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Blandit libero volutpat sed cras. Purus semper eget duis at.</p>';

	let output = '';

	const getJSON = () => {
		output = JSON.stringify(editor.getJSON());
	};
	const getHTML = () => {
		output = editor.getHTML() ?? '';
	};
</script>

<h1>Output</h1>
<button onclick={getJSON}>Get JSON</button>
<button onclick={getHTML}>Get HTML</button>
{#if output}
	<textarea cols="30" rows="10" bind:value={output}></textarea>
{/if}
<OmniaEditor bind:this={editor} {value} />

<h1>History</h1>

<button onclick={editorCount.undo}>Undo</button>
<button onclick={editorCount.redo}>Redo</button>
<OmniaEditor bind:this={editorCount} {value} />

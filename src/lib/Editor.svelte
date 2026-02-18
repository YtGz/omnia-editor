<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { Editor, type FocusPosition } from '@tiptap/core';
	import StarterKit from '@tiptap/starter-kit';
	import BubbleMenu from '@tiptap/extension-bubble-menu';
	import FloatingMenu from '@tiptap/extension-floating-menu';
	import Focus from '@tiptap/extension-focus';
	import TextAlign from '@tiptap/extension-text-align';

	import Float from './Float.svelte';
	import Tooltip from './Tooltip.svelte';

	interface Props {
		/** Initial value of the content. */
		value?: string;
		/** Use spellcheck. */
		spellcheck?: boolean;
		/** Called when the editor is created. */
		oninit?: () => void;
		/** Called when the editor gains focus. */
		onfocus?: () => void;
		/** Called when the editor loses focus. */
		onblur?: () => void;
		/** Called when the editor content changes. */
		onchange?: () => void;
	}

	let { value = '', spellcheck = false, oninit, onfocus, onblur, onchange }: Props = $props();

	let editor: Editor | undefined = $state(undefined);

	let element: HTMLElement;
	let floating: HTMLElement;
	let tooltip: HTMLElement;

	/** Get the document as JSON. */
	export function getJSON() {
		return editor?.getJSON();
	}
	/** Get the document as HTML. */
	export function getHTML() {
		return editor?.getHTML();
	}
	/** Get the number of characters for the current document. */
	export function getCharacterCount() {
		return editor?.storage.characterCount?.characters?.() ?? editor?.getText().length ?? 0;
	}
	/** Returns `true` when Editor is focused. */
	export function isFocused() {
		return editor?.isFocused;
	}
	/** Returns `true` if there is no content. */
	export function isEmpty() {
		return editor?.isEmpty;
	}
	/** Removes focus from the editor. */
	export function blur() {
		return editor?.commands.blur();
	}
	/**
	 * Focus the editor at the given position.
	 *
	 * @param position - 'start' | 'end' | 'all' | number
	 */
	export function focus(position?: FocusPosition) {
		return editor?.commands.focus(position);
	}
	/** Undo recent changes. */
	export function undo() {
		return editor?.commands.undo();
	}
	/** Reapply reverted changes. */
	export function redo() {
		return editor?.commands.redo();
	}
	/** Destroy the editor. */
	export function destroy() {
		return editor?.destroy();
	}

	onMount(() => {
		editor = new Editor({
			element: element,
			extensions: [
				StarterKit,
				BubbleMenu.configure({
				}),
				FloatingMenu.configure({
					element: floating
				}),
				Focus.configure({
					className: 'focus'
				}),
				TextAlign.configure({
					types: ['heading', 'paragraph']
				})
			],
			content: value,
			onCreate: () => oninit?.(),
			onFocus: () => onfocus?.(),
			onBlur: () => onblur?.(),
			onUpdate: () => onchange?.(),
			onTransaction: () => {
				// force re-render so `editor.isActive` works as expected
				editor = editor;
			}
		});
	});

	onDestroy(() => {
		if (editor) {
			editor.destroy();
			editor = undefined;
		}
	});
</script>

<div class="omnia-editor">
	<!-- svelte-ignore element_invalid_self_closing_tag -->
	<div
		{spellcheck}
		bind:this={element}
		autocapitalize="off"
	>
		<div class="ProseMirror"></div>
	</div>
	<div bind:this={floating}>
		<Float {editor} />
	</div>
	<div bind:this={tooltip}>
		<Tooltip {editor} />
	</div>
</div>

<style lang="scss">
	@use 'sass/mixins';

	:global {
		.omnia-editor {
			--omnia-editor-bg-color: white;
			--omnia-editor-text-color: black;
		}

		.ProseMirror {
			width: 100%;
			margin-left: auto;
			margin-right: auto;
			text-align: justify;
			background-color: var(--omnia-editor-bg-color);
			color: var(--omnia-editor-text-color);
			outline: 0;

			> * {
				font-family: inherit;
			}

			> p,
			blockquote {
				line-height: 2.5rem;
				padding: 0 1rem;

				@include mixins.on-mobile {
					padding: 0;
					line-height: 1.5rem;
				}
			}

			> blockquote {
				text-align: center;
				font-style: italic;
			}

			&.ProseMirror-focused {
				> * {
					transition: opacity 0.1s linear;
					&:not(.focus) {
						opacity: 0.4;
					}
				}
			}
		}
	}
</style>

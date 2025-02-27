<script context="module" lang="ts">
	export { default as BaseChatBot } from "./shared/ChatBot.svelte";
</script>

<script lang="ts">
	import type { Gradio, SelectData, LikeData } from "@gradio/utils";

	import ChatBot from "./shared/ChatBot.svelte";
	import { Block, BlockLabel } from "@gradio/atoms";
	import type { LoadingStatus } from "@gradio/statustracker";
	import { Chat } from "@gradio/icons";
	import type { FileData } from "@gradio/client";
	import { StatusTracker } from "@gradio/statustracker";
	import type { FileMessage, MirageMessage } from "./shared/utils";

	export let elem_id = "";
	export let elem_classes: string[] = [];
	export let visible = true;
	export let value: [
		MirageMessage | null,
		MirageMessage | null
	][] = [];
	export let scale: number | null = null;
	export let min_width: number | undefined = undefined;
	export let label: string;
	export let show_label = true;
	export let root: string;
	export let _selectable = false;
	export let likeable = false;
	export let show_share_button = false;
	export let rtl = false;
	export let show_copy_button = false;
	export let sanitize_html = true;
	export let bubble_full_width = true;
	export let layout: "bubble" | "panel" = "bubble";
	export let render_markdown = true;
	export let line_breaks = true;
	export let latex_delimiters: {
		left: string;
		right: string;
		display: boolean;
	}[];
	export let gradio: Gradio<{
		change: typeof value;
		select: SelectData;
		share: ShareData;
		error: string;
		like: LikeData;
		clear_status: LoadingStatus;
	}>;
	export let avatar_images: [FileData | null, FileData | null] = [null, null];

	let _value: [
		MirageMessage | null,
		MirageMessage | null
	][];

	const redirect_src_url = (src: string): string =>
		src.replace('src="/file', `src="${root}file`);

	function normalize_messages(
		message: FileMessage | null
	): FileMessage | null {
		if (message === null) {
			return message;
		}
		return {
			file: message?.file as FileData,
			alt_text: message?.alt_text
		};
	}

	function process_message(msg: MirageMessage | null): MirageMessage | null {
		if (msg === null) {
			return msg;
		}
		msg.text = redirect_src_url(msg.text);
		msg.files = msg.files.map(normalize_messages);
		return msg;
	}

	$: _value = value
		? value.map(([user_msg, bot_msg]) => [
				process_message(user_msg),
				process_message(bot_msg)
			])
		: [];

	export let loading_status: LoadingStatus | undefined = undefined;
	export let height = 400;
	export let placeholder: string | null = null;
</script>

<Block
	{elem_id}
	{elem_classes}
	{visible}
	padding={false}
	{scale}
	{min_width}
	{height}
	allow_overflow={false}
>
	{#if loading_status}
		<StatusTracker
			autoscroll={gradio.autoscroll}
			i18n={gradio.i18n}
			{...loading_status}
			show_progress={loading_status.show_progress === "hidden"
				? "hidden"
				: "minimal"}
			on:clear_status={() => gradio.dispatch("clear_status", loading_status)}
		/>
	{/if}
	<div class="wrapper">
		{#if show_label}
			<BlockLabel
				{show_label}
				Icon={Chat}
				float={false}
				label={label || "Chatbot"}
			/>
		{/if}
		<ChatBot
			i18n={gradio.i18n}
			selectable={_selectable}
			{likeable}
			{show_share_button}
			value={_value}
			{latex_delimiters}
			{render_markdown}
			pending_message={loading_status?.status === "pending"}
			{rtl}
			{show_copy_button}
			on:change={() => gradio.dispatch("change", value)}
			on:select={(e) => gradio.dispatch("select", e.detail)}
			on:like={(e) => gradio.dispatch("like", e.detail)}
			on:share={(e) => gradio.dispatch("share", e.detail)}
			on:error={(e) => gradio.dispatch("error", e.detail)}
			{avatar_images}
			{sanitize_html}
			{bubble_full_width}
			{line_breaks}
			{layout}
			{placeholder}
		/>
	</div>
</Block>

<style>
	.wrapper {
		display: flex;
		position: relative;
		flex-direction: column;
		align-items: start;
		width: 100%;
		height: 100%;
	}
</style>

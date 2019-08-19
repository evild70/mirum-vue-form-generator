<template lang="html">
	<a
		class="u-reset-button u-color-blue-digital"
		tabindex="0"
		role="button"
		data-tooltip=""
		data-toggle="popover"
		data-trigger="focus"
		data-placement="top"
		aria-label="Tool tip"
		:title="header"
		:data-content="body"
		:data-has-image="hasImage"
		ref="tooltip"
	>
		<span class="o-svg-icon o-svg-icon--xl">
			<svg viewBox="0, 0, 24, 24" focusable="false" role="presentation" shape-rendering="geometricPrecision">
				<path d="M12,1.208 C17.957,1.208 22.792,6.043 22.792,12 C22.792,17.957 17.957,22.792 12,22.792 C6.043,22.792 1.208,17.957 1.208,12 C1.208,6.043 6.043,1.208 12,1.208 z M13.079,10.921 L10.921,10.921 L10.921,17.396 L13.079,17.396 L13.079,10.921 z M12,6.354 C11.266,6.354 10.671,6.949 10.671,7.683 C10.671,8.417 11.266,9.012 12,9.012 C12.734,9.012 13.329,8.417 13.329,7.683 C13.329,6.949 12.734,6.354 12,6.354 z" fill="currentColor" />
			</svg>
		</span>
	</a>
</template>

<script>
/* global $ */

export default {
	props: {
		header: {
			type: String,
			required: false,
		},
		body: {
			type: String,
			required: false,
		},
		hasImage: {
			type: Boolean,
			default: false,
		},
	},
	mounted() {
		// Init tooltips
		let template = '<div class="c-tooltip popover" role="tooltip"><h3 class="c-tooltip__header popover-header"></h3><div class="c-tooltip__body popover-body js-tooltip-body"></div></div>'; // eslint-disable-line
		if (this.hasImage) {
			template = '<div class="c-tooltip c-tooltip--image popover" role="tooltip"><h3 class="c-tooltip__header popover-header"></h3><div class="c-tooltip__body popover-body js-tooltip-body"></div></div>'; // eslint-disable-line
		}
		$(this.$refs.tooltip).popover({
			html: true,
			template,
		}).on("inserted.bs.popover", () => {
			// Ensure TFNs are dynamic inside of tooltips.
			window.tfn.init();
		});
	},
};
</script>

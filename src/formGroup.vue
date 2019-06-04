<template>
	<div class="form-group" :class="getFieldRowClasses(field)">
		<button v-if="field.tooltip"
			class="u-reset-button u-color-blue-digital"
			tabindex="0"
			role="button"
			data-toggle="popover"
			data-trigger="focus"
			data-placement="top"
			:title="field.tooltip.header"
			:data-content="field.tooltip.body"
		>
			<span class="o-svg-icon o-svg-icon--xl">
				<svg viewBox="0, 0, 24, 24" focusable="false" role="presentation" shape-rendering="geometricPrecision">
					<path d="M12,1.208 C17.957,1.208 22.792,6.043 22.792,12 C22.792,17.957 17.957,22.792 12,22.792 C6.043,22.792 1.208,17.957 1.208,12 C1.208,6.043 6.043,1.208 12,1.208 z M13.079,10.921 L10.921,10.921 L10.921,17.396 L13.079,17.396 L13.079,10.921 z M12,6.354 C11.266,6.354 10.671,6.949 10.671,7.683 C10.671,8.417 11.266,9.012 12,9.012 C12.734,9.012 13.329,8.417 13.329,7.683 C13.329,6.949 12.734,6.354 12,6.354 z" fill="currentColor" />
				</svg>
			</span>
		</button>
		<label v-if="fieldTypeHasLabel(field)" :for="getFieldID(field)" :class="field.labelClasses">
			<span v-html="field.label"></span>
		</label>

		<div class="field-wrap">
			<component ref="child"
				:is="getFieldType(field)"
				:vfg="vfg"
				:disabled="fieldDisabled(field)"
				:model="model"
				:schema="field"
				:formOptions="options"
				@model-updated="onModelUpdated"
				@validated="onFieldValidated">
			</component>
			<div v-if="buttonVisibility(field)" class="buttons">
				<button v-for="(btn, index) in field.buttons"
					@click="buttonClickHandler(btn, field, $event)"
					:class="btn.classes"
					:key="index"
					v-text="btn.label"
					:type="getButtonType(btn)"></button>
			</div>
		</div>

		<div v-show="field.hint" class="hint" v-html="fieldHint(field)"></div>

		<div class="errors help-block" role="alert">
			<span v-for="(error, index) in fieldErrors(field)" :key="index" v-html="error"></span>
		</div>
	</div>
</template>
<script>
import { get as objGet, isNil, isFunction } from "lodash";
import { slugifyFormID } from "./utils/schema";
import formMixin from "./formMixin.js";
import fieldComponents from "./utils/fieldsLoader.js";

export default {
	name: "form-group",
	components: fieldComponents,
	mixins: [formMixin],
	props: {
		vfg: {
			type: Object,
			required: true
		},
		model: Object,
		options: {
			type: Object
		},
		field: {
			type: Object,
			required: true
		},
		errors: {
			type: Array,
			default() {
				return [];
			}
		}
	},
	methods: {
		// Should field type have a label?
		fieldTypeHasLabel(field) {
			// if there is no label or it's a radio or checkbox
			if (isNil(field.label) || field.type === "dtlcheckbox" || field.type === "dtlradio") return false;

			let relevantType = "";
			if (field.type === "input") {
				relevantType = field.inputType;
			} else {
				relevantType = field.type;
			}

			switch (relevantType) {
				case "button":
				case "submit":
				case "reset":
					return false;
				default:
					return true;
			}
		},
		getFieldID(schema) {
			const idPrefix = objGet(this.options, "fieldIdPrefix", "");
			return slugifyFormID(schema, idPrefix);
		},
		// Get type of field 'field-xxx'. It'll be the name of HTML element
		getFieldType(fieldSchema) {
			return "field-" + fieldSchema.type;
		},
		// Get type of button, default to 'button'
		getButtonType(btn) {
			return objGet(btn, "type", "button");
		},
		// Child field executed validation
		onFieldValidated(res, errors, field) {
			this.$emit("validated", res, errors, field);
		},
		buttonVisibility(field) {
			return field.buttons && field.buttons.length > 0;
		},
		buttonClickHandler(btn, field, event) {
			return btn.onclick.call(this, this.model, field, event, this);
		},
		// Get current hint.
		fieldHint(field) {
			if (isFunction(field.hint)) return field.hint.call(this, this.model, field, this);

			return field.hint;
		},
		fieldErrors(field) {
			return this.errors.filter((e) => e.field === field).map((item) => item.error);
		},
		onModelUpdated(newVal, schema) {
			this.$emit("model-updated", newVal, schema);
		},
		validate(calledParent) {
			return this.$refs.child.validate(calledParent);
		},
		clearValidationErrors() {
			if (this.$refs.child) {
				return this.$refs.child.clearValidationErrors();
			}
		}
	}
};
</script>
<style lang="scss">
$errorColor: #f00;
.form-group:not([class*=" col-"]) {
	width: 100%;
}
.form-group {
	display: inline-block;
	vertical-align: top;
	// width: 100%;
	// margin: 0.5rem 0.26rem;
	margin-bottom: 1rem;

	label {
		font-weight: 400;
		& > :first-child {
			display: inline-block;
		}
	}

	&.featured {
		> label {
			font-weight: bold;
		}
	}

	&.required {
		> label:after {
			content: "*";
			font-weight: normal;
			color: Red;
			// position: absolute;
			padding-left: 0.2em;
			font-size: 1em;
		}
	}

	&.disabled {
		> label {
			color: #666;
			font-style: italic;
		}
	}

	&.error {
		input:not([type="checkbox"]),
		textarea,
		select {
			border: 1px solid $errorColor;
			background-color: rgba($errorColor, 0.15);
		}

		.errors {
			color: $errorColor;
			font-size: 0.8em;
			span {
				display: block;
				background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAAiklEQVR4Xt2TMQoCQQxF3xdhu72MpZU3GU/meBFLOztPYrVWsQmEWSaMsIXgK8P8RyYkMjO2sAN+K9gTIAmDAlzoUzE7p4IFytvDCQWJKSStYB2efcAvqZFM0BcstMx5naSDYFzfLhh/4SmRM+6Agw/xIX0tKEDFufeDNRUc4XqLRz3qabVIf3BMHwl6Ktexn3nmAAAAAElFTkSuQmCC");
				background-repeat: no-repeat;
				padding-left: 17px;
				padding-top: 0px;
				margin-top: 0.2em;
				font-weight: 600;
			}
		}
	}
}
</style>

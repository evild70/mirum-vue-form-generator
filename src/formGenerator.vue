<template>
	<div class="vue-form-generator" v-if="schema != null">
		<fieldset v-if="schema.fields" :is="tag">
			<template v-for="(field, index) in fields">
				<form-group v-if="fieldVisible(field)"
					:vfg="vfg"
					:field="field"
					:errors="errors"
					:model="model"
					:options="options"
					@validated="onFieldValidated"
					@model-updated="onModelUpdated"
					:key="index">
				</form-group>
			</template>
		</fieldset>
		<!-- Groups -->
		<template v-for="(group, groupidx) in groups">
			<fieldset :is="tag" :class="getFieldRowClasses(group)" :key="groupidx">
				<legend v-if="group.legend">{{ group.legend }}</legend>
				<template v-for="(field, index) in group.fields">
					<form-group v-if="fieldVisible(field)"
						:vfg="vfg"
						:field="field"
						:errors="errors"
						:model="model"
						:options="options"
						@validated="onFieldValidated"
						@model-updated="onModelUpdated"
						:key="index">
					</form-group>
				</template>
			</fieldset>
		</template>
		<!-- Steps -->
		<template v-if="steps">
			<ul class="c-tab c-tab--step">
				<li
					v-for="(step, stepindex) in steps"
					:key="stepindex"
					class="c-tab__item"
				>
					<a
						href="#"
						:class="getStepTabClasses(step, stepindex)"
						role="tab"
						:aria-description="getStepTabDescription(step, stepindex)"
						@click.prevent="setActiveStep(stepindex)"
						ref="stepTab"
					>
						{{ step.title }}
					</a>
				</li>
			</ul>
			<div class="u-p-t">
				<fieldset
					v-for="(step, stepindex) in steps"
					v-show="activeStep === stepindex"
					:is="tag"
					:class="getFieldRowClasses(step)"
					:key="stepindex"
					:aria-hidden="activeStep !== stepindex"
					role="tabpanel"
				>
					<template v-for="(field, index) in step.fields">
						<form-group v-if="fieldVisible(field)"
							:vfg="vfg"
							:field="field"
							:errors="errors"
							:model="model"
							:options="options"
							@validated="onFieldValidated"
							@model-updated="onModelUpdated"
							:key="index"
							:ref="`step_${stepindex}`">
						</form-group>
					</template>
				</fieldset>
			</div>
		</template>
	</div>
</template>

<script>
import { get as objGet, forEach, isFunction, isNil, isArray } from "lodash";
import formMixin from "./formMixin.js";
import formGroup from "./formGroup.vue";

export default {
	name: "formGenerator",
	components: { formGroup },
	mixins: [formMixin],
	props: {
		schema: Object,

		model: Object,

		options: {
			type: Object,
			default() {
				return {
					validateAfterLoad: false,
					validateAsync: false,
					validateAfterChanged: false,
					validationErrorClass: "error",
					validationSuccessClass: ""
				};
			}
		},

		multiple: {
			type: Boolean,
			default: false
		},

		isNewModel: {
			type: Boolean,
			default: false
		},

		tag: {
			type: String,
			default: "fieldset",
			validator: function(value) {
				return value.length > 0;
			}
		}
	},

	data() {
		return {
			vfg: this,
			activeStep: 0,
			canStepTo: 0,
			prevStep: 0, // For async validation
			errors: [], // Validation errors
			fieldToStepMap: {},
			stepValid: [],
		};
	},

	computed: {
		fields() {
			let res = [];
			if (this.schema && this.schema.fields) {
				forEach(this.schema.fields, field => {
					if (!this.multiple || field.multi === true) res.push(field);
				});
			}

			return res;
		},
		groups() {
			let res = [];
			if (this.schema && this.schema.groups) {
				forEach(this.schema.groups.slice(0), group => {
					res.push(group);
				});
			}

			return res;
		},
		steps() {
			let res = [];
			if (this.schema && this.schema.steps) {
				forEach(this.schema.steps.slice(0), step => {
					res.push(step);
				});
			}

			return res;
		}
	},

	watch: {
		// new model loaded
		model: function(newModel, oldModel) {
			if (oldModel === newModel)
				// model property changed, skip
				return;

			if (newModel != null) {
				this.$nextTick(() => {
					// Model changed!
					if (this.options.validateAfterLoad === true && this.isNewModel !== true) {
						this.validate();
					} else {
						this.clearValidationErrors();
					}
				});
			}
		}
	},

	created() {
		// Set default step validation
		if (this.schema.steps) {
			for (let i = 0; i < this.schema.steps.length; i++) {
				this.stepValid.push(true);
			}
		}
	},

	mounted() {
		this.$nextTick(() => {
			if (this.model) {
				// First load, running validation if neccessary
				if (this.options.validateAfterLoad === true && this.isNewModel !== true) {
					this.validate();
				} else {
					this.clearValidationErrors();
				}
			}
		});
	},

	methods: {
		// Get visible prop of field
		fieldVisible(field) {
			if (isFunction(field.visible)) return field.visible.call(this, this.model, field, this);

			if (isNil(field.visible)) return true;

			return field.visible;
		},

		setFieldToStepMap(fieldName) {
			for (let i = 0; i < this.schema.steps.length; i++) {
				const step = this.schema.steps[i];
				for (let j = 0; j < step.fields.length; j++) {
					if (fieldName === step.fields[j].model) {
						this.fieldToStepMap[fieldName] = i;
						return i;
					}
				}
			}
			return NaN;
		},

		handleStepValidation(field, isValid) {
			// Set this no matter what so we can use it as a "hasBeenModified" check
			let stepIndex = this.fieldToStepMap[field.schema.model];
			if (typeof stepIndex !== "number") {
				stepIndex = this.setFieldToStepMap(field.schema.model);
			}

			// If no errors exist, set all to valid
			if (this.errors.length === 0) {
				for (let i = 0; i < this.stepValid.length; i++) {
					this.stepValid[i] = true;
				}
			} else {
				const isStepValid = this.stepValid[stepIndex];
				if (isStepValid !== isValid) {
					if (isStepValid) {
						// The step is now invalid
						this.stepValid[stepIndex] = false;
					} else {
						// Check if there are any other errors in this step.
						for (let i = 0; i < this.errors.length; i++) {
							const name = this.errors[i].field.model;
							if (typeof this.fieldToStepMap[name] === "number") {
								if (this.fieldToStepMap[name] === stepIndex) {
									// There was another error in this step. Still invalid.
									return;
								}
							}
						}
						// There were no errors in this step.
						this.stepValid[stepIndex] = true;
					}
				}
			}
		},

		// Child field executed validation
		onFieldValidated(res, errors, field) {
			// Remove old errors for this model name
			this.errors = this.errors.filter(e => e.field.model !== field.schema.model);

			if (!res && errors && errors.length > 0) {
				// Add errors with this field
				forEach(errors, err => {
					this.errors.push({
						field: field.schema,
						error: err
					});
				});
			}

			let isValid = this.errors.length === 0;
			this.$emit("validated", isValid, this.errors, this);

			if (this.schema.steps && field.schema.model) {
				this.handleStepValidation(field, errors.length === 0);
			}
		},

		onModelUpdated(newVal, schema) {
			this.$emit("model-updated", newVal, schema);
		},

		// Validating the model properties
		validate(isAsync = null) {
			if (isAsync === null) {
				isAsync = objGet(this.options, "validateAsync", false);
			}
			this.clearValidationErrors();

			let fields = [];
			let results = [];

			forEach(this.$children, child => {
				if (isFunction(child.validate)) {
					fields.push(child.$refs.child); // keep track of validated children
					results.push(child.validate(true));
				}
			});

			let handleErrors = errors => {
				let formErrors = [];
				forEach(errors, (err, i) => {
					if (isArray(err) && err.length > 0) {
						forEach(err, error => {
							formErrors.push({
								field: fields[i].schema,
								error: error
							});
						});
					}
				});
				this.errors = formErrors;
				let isValid = formErrors.length === 0;
				this.$emit("validated", isValid, formErrors, this);
				return isAsync ? formErrors : isValid;
			};

			if (!isAsync) {
				return handleErrors(results);
			}

			return Promise.all(results).then(handleErrors);
		},

		// Clear validation errors
		clearValidationErrors() {
			this.errors.splice(0);

			forEach(this.$children, child => {
				child.clearValidationErrors();
			});
		},

		getStepTabClasses(step, index) {
			const hasErrors = !this.stepValid[index];
			const baseClasses = {
				"c-tab__tab": true,
				[objGet(this.options, "validationErrorClass", "error")]: hasErrors,
				[objGet(this.options, "validationSuccessClass", "valid")]: !hasErrors,
				disabled: index > this.canStepTo,
				active: index === this.activeStep,
			};
			return baseClasses;
		},

		getStepTabDescription(step, index) {
			const description = [];

			if (index > this.canStepTo) {
				description.push(this.schema.stepDisabledText || "You must complete the previous steps before you can access this one.");
			}

			if (!this.stepValid[index]) {
				description.push(this.schema.stepErrorText || "There are errors present on this step.");
			}

			if (description.length > 0) {
				return description.join(" ");
			}
			return false;
		},

		/**
		 * Checks each field in the step for validitiy.
		 * We don't immediately return on the first false validation so that we can
		 * mimic submitting the current step.
		 *
		 * @param {Number} index The step index.
		 *
		 * @returns {Boolean} The validity of the step.
		 */
		validateStep(index) {
			let isValid = true;
			for (let i = 0; i < this.$refs[`step_${index}`].length; i++) {
				const field = this.$refs[`step_${index}`][i];
				if (isFunction(field.validate)) {
					const validation = field.validate();
					if (Array.isArray(validation)) {
						console.log('reg');
						if (validation.length > 0) {
							isValid = false;
						}
					} else {
						const checkValidation = (res) => {
							console.log(res.length > 0);
							if (res.length > 0) {
								isValid = false;
								this.setActiveStep(this.prevStep, false);
								this.canStepTo = this.prevStep;
								this.$emit("steperror");
								return;
							}
						};

						validation.then(checkValidation);
					}
				}
			}

			if (!isValid) {
				this.$emit("steperror");
			}
			return isValid;
		},

		setActiveStep(index, isUserAction = true) {
			if (isUserAction) {
				this.prevStep = this.activeStep;
			}
			this.activeStep = index;
			this.$emit("gotostep", index);
			this.$refs.stepTab[index].focus();
		},

		goToStep(index) {
			if (index > this.activeStep) {
				console.log(this.validateStep(this.activeStep));
				console.log('---');
				if (this.validateStep(this.activeStep)) {
					if (index === this.canStepTo + 1) {
						this.canStepTo = index;
					}
					this.setActiveStep(index);
				}
			} else if (index <= this.canStepTo && index !== this.activeStep) {
				this.setActiveStep(index);
			}
		},
	}
};
</script>

<style lang="scss">
.vue-form-generator {
	* {
		box-sizing: border-box;
	}

	.form-control {
		// Default Bootstrap .form-control style
		display: block;
		&:not([class*=" col-"]) {
			width: 100%;
		}
		padding: 6px 12px;
		font-size: 14px;
		line-height: 1.42857143;
		color: #555;
		background-color: #fff;
		background-image: none;
		border: 1px solid #ccc;
		border-radius: 4px;
		box-shadow: inset 0 1px 1px rgba(0, 0, 0, 0.075);
		transition: border-color ease-in-out 0.15s, box-shadow ease-in-out 0.15s;
	} // .form-control

	span.help {
		margin-left: 0.3em;
		position: relative;

		.icon {
			display: inline-block;
			width: 16px;
			height: 14px;
			background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAYAAAAf8/9hAAAABmJLR0QA/wD/AP+gvaeTAAAA+UlEQVQ4ja3TS0oDQRAG4C8+lq7ceICICoLGK7iXuNBbeAMJuPVOIm7cqmDiIncIggg+cMZFaqCnZyYKWtB0df31V1VXdfNH6S2wD9CP8xT3KH8T9BiTcE7XBMOfyBcogvCFO9ziLWwFRosyV+QxthNsA9dJkEYlvazsQdi3sBv6Ol6TBLX+HWT3fcQZ3vGM5fBLk+ynAU41m1biCXvhs4OPBDuBpa6GxF0P8YAj3GA1d1qJfdoS4DOIcIm1DK9x8iaWeDF/SP3QU6zRROpjLDFLsFlibx1jJaMkSIGrWKntvItcyTBKzCcybsvc9ZmYz3kz9Ooz/b98A8yvW13B3ch6AAAAAElFTkSuQmCC");
			background-repeat: no-repeat;
			background-position: center center;
		} // .icon

		.helpText {
			background-color: #444;
			bottom: 30px;
			color: #fff;
			display: block;
			left: 0px;
			//margin-bottom: 15px;
			opacity: 0;
			padding: 20px;
			pointer-events: none;
			position: absolute;
			text-align: justify;
			width: 300px;
			//transform: translateY(10%);
			transition: all 0.25s ease-out;
			box-shadow: 2px 2px 6px rgba(0, 0, 0, 0.5);
			border-radius: 6px;

			a {
				font-weight: bold;
				text-decoration: underline;
			} // a
		} // .helpText

		/* This bridges the gap so you can mouse into the tooltip without it disappearing */
		.helpText:before {
			bottom: -20px;
			content: " ";
			display: block;
			height: 20px;
			left: 0;
			position: absolute;
			width: 100%;
		}

		&:hover .helpText {
			opacity: 1;
			pointer-events: auto;
			transform: translateY(0px);
		}
	} // span.help

	.field-wrap {
		display: flex;

		.buttons {
			white-space: nowrap;
			margin-left: 4px;
		}

		button,
		input[type="submit"] {
			// Default Bootstrap button style
			display: inline-block;
			padding: 6px 12px;
			margin: 0px;
			font-size: 14px;
			font-weight: normal;
			line-height: 1.42857143;
			text-align: center;
			white-space: nowrap;
			vertical-align: middle;
			touch-action: manipulation;
			cursor: pointer;
			user-select: none;
			color: #333;
			background-color: #fff;
			border: 1px solid #ccc;
			border-radius: 4px;

			&:not(:last-child) {
				margin-right: 4px;
			}

			&:hover {
				color: #333;
				background-color: #e6e6e6;
				border-color: #adadad;
			}

			&:active {
				color: #333;
				background-color: #d4d4d4;
				border-color: #8c8c8c;
				outline: 0;
				box-shadow: inset 0 3px 5px rgba(0, 0, 0, 0.125);
			}

			&:disabled {
				opacity: 0.6;
				cursor: not-allowed;
			}
		} // button, input[submit]
	} // .field-wrap

	.hint {
		font-style: italic;
		font-size: 0.8em;
	} // .hint
} // fieldset
</style>

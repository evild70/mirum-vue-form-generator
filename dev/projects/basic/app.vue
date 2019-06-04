<template>
	<div class="container">
		<p>&nbsp;</p>
		<h1>Basic</h1>
		<div class="row">
			<div class="col-sm-12">
				<vue-form-generator :schema="schema" :model="model" :options="formOptions" ref="form" :is-new-model="isNewModel" @model-updated="modelUpdated" @validated="onValidated"></vue-form-generator>
			</div>
		</div>
		<div class="row">
			<div class="col-sm-12">
				<pre v-if="model" v-html="prettyModel"></pre>
			</div>
		</div>
	</div>
</template>

<script>
import mixinUtils from "../../mixins/utils.js";

export default {
	mixins: [mixinUtils],

	data() {
		return {
			isNewModel: false,

			selected: [],

			model: {
				first_name: "",
				last_name: "",
				status: false
			},

			schema: {
				fields: [
					{
						type: "input",
						inputType: "text",
						label: "First Name",
						model: "first_name",
						required: true,
						validator: "string",
						help: "This is help text."
					},
					{
						type: "input",
						inputType: "text",
						label: "Last Name",
						model: "last_name",
						required: true,
						validator: "string",
						tooltip: {
							header: "Tooltip title top",
							body: "Tooltip description will go right here, but keep it reasonable."
						},
						help: "This is help text."
					},
					{
						type: "checkbox",
						label: "Active",
						model: "status",

					},
					{
						type: "input",
						inputType: "text",
						label: "Color",
						model: "color",
						visible: function() {
							return this.model && this.model.status;
						}
					},
					{
						type: "submit",
						buttonText: "Change Previous Type",
						onSubmit: () => {

						}
					}
				]
			},

			formOptions: {
				validateAfterLoad: false,
				validateAfterChanged: false,
				validateBeforeSave: true
			}
		};
	},

	methods: {
		showWarning() {
			if (this.$refs.form && this.$refs.form.errors) {
				return this.$refs.form.errors.length > 0;
			}
		},

		onValidated(res, errors) {
			console.log("VFG validated:", res, errors);
		},

		modelUpdated(newVal, schema) {
			console.log("main model has updated", newVal, schema);
		},
	},

	mounted() {
		this.$nextTick(function() {
			window.app = this;
		});
	}
};
</script>

<style lang="scss">
@import "../../style.scss";
@import "../../unity.scss";
</style>

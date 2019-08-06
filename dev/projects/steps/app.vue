<template>
	<div class="container">
		<h1>Steps</h1>
		<div class="row">
			<div class="col-sm-12">
				<vue-form-generator :schema="schema" :model="model" :options="formOptions" tag="section"></vue-form-generator>
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
			model: {
				name: "Brian Blessed",
				email: "brian@hawkman.mongo",
				others: {
					more: "More",
					things: "Things"
				},
				single: "blah"
			},

			schema: {
				steps: [
					{
						title: "Step 1",
						fields: [
							{
								type: "input",
								inputType: "text",
								label: "Name",
								model: "name",
								required: true,
								validator: "string",
							},
							{
								type: "input",
								inputType: "email",
								label: "Email",
								model: "email",
								required: true,
								validator: "string",
							}
						]
					},
					{
						title: "Step 2",
						fields: [
							{
								type: "input",
								inputType: "text",
								label: "More",
								model: "others.more",
								required: true,
								validator: "string",
							},
							{
								type: "input",
								inputType: "text",
								label: "Things",
								model: "others.things"
							}
						]
					}
				],
			},

			formOptions: {
				fieldIdPrefix: "frm1-"
			}
		};
	},

	created() {
		window.app = this;
	}
};
</script>

<style lang="scss">
@import "../../style.scss";
</style>

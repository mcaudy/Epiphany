<template lang="pug">
	form(@submit="submitForm($event)")
		.my-search
			input#search-bar(v-model="input_data", type="text", :placeholder="placeholder", @keypress="checkKey($event)", @keydown.enter.exact.prevent="submitForm($event)", ref="input_data")
			i.coon-x-circle(v-show="input_data", @click="clear_input")
		.my-buttons
			.my-button(v-for="button in buttons")
				button(type="buttonType(button)", @click="buttonClick($event, button)")
					| {{ button.label }}
</template>

<script>
	export default {
		name: 'inputText',
		data() {
			return {
				input_data: ""
			};
		},
		props: {
			'required'        : Boolean,
			'alphanumericOnly': Boolean,
			'placeholder'     : String,
			'buttons'         : Array,
			'defaultData'     : String
		},
		mounted() {
			this.input_data = this.defaultData;
		},
		computed: {
			formData() {
				return {
					input_data: this.input_data
				};
			},
			submitButton() {
				var submits = this.buttons.filter((obj) => {
					return obj.type == "submit";
				});
				if (submits.length > 0) {
					return submits[0];
				}
				return null;
			}
		},
		methods: {
			clear_input() {
				this.input_data = "";
			},
			buttonType(button_obj) {
				if (button_obj.type == "submit") {
					return "submit";
				}
				return "button";
			},
			buttonClick(event, button_obj) {
				switch(button_obj.type) {
					case "submit":
						this.submitForm(event);
						break;
					default:
						if (button_obj && typeof button_obj.callback == "function") {
							button_obj.callback(this.formData);
						}
						this.$root.closingWindow();
						break;
				}
				return false;
			},
			errorFlash(event, element) {
				if (event) event.preventDefault();
				this.$refs[element].classList.add("error")
				return;
			},
			checkKey(event) {
				if (this.alphanumericOnly) {
					var ew = event.which;
					if (ew == 32)
						return true;
					if (48 <= ew && ew <= 57)
						return true;
					if (65 <= ew && ew <= 90)
						return true;
					if (97 <= ew && ew <= 122)
						return true;

					event.preventDefault();
					return false;
				}
				return true;
			},
			submitForm(event) {
				if (this.required && this.input_data.length == 0) {
					return this.errorFlash(event, "input_data");
				}

				if (this.submitButton && typeof this.submitButton.validate == "function") {
					var result = this.submitButton.validate(this.formData);
					if (result) {
						return this.errorFlash(event, result);
					}
				}

				if (this.submitButton && typeof this.submitButton.callback == "function") {
					this.submitButton.callback(this.formData);
				}

				this.$root.inputSubmit(this.formData);
			},
		},
		watch: {
			input_data() {
				this.$refs["input_data"].classList.remove("error");
			}
		}
	}
</script>
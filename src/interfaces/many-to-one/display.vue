<template>
	<div class="no-wrap">{{ displayValue }}</div>
</template>

<script>
import mixin from '@directus/extension-toolkit/mixins/interface';
import { findIndex } from 'lodash';

export default {
	mixins: [mixin],
	data() {
		return {
			loading: false,
			data: null // when the primary key is passed in, we'll fetch the related item so we can
			// display the value like normal
		};
	},
	computed: {
		systemLanguage() {
			return this.$i18n.locale.split('-')[0];
		},
		displayValue() {
			let value = this.value;
			let template = this.options.template;
			if (template.includes('.')) {
				let translationKeys = [];
				let parsedTemplate = template;
				Object.keys(value).forEach(key => {
					// a translation is an array - so I check for it and I replace it with the index of the language
					// for instance: data.name becomes data.[index-of-lang].name
					if (Array.isArray(value[key])) {
						const langIndex = findIndex(value[key], o => {
							return o.lang == this.systemLanguage;
						});
						parsedTemplate = parsedTemplate.replace(
							new RegExp(key, 'g'),
							`${key}.${langIndex}`
						);
					}
				});
				template = parsedTemplate;
			}
			if (this.isPrimaryKey && this.data && this.loading === false) {
				value = this.data;
			}

			if (value) {
				return this.$helpers.micromustache.render(template, value);
			}

			return '--';
		},
		isPrimaryKey() {
			return typeof this.value !== 'object';
		}
	},
	watch: {
		value() {
			if (this.isPrimaryKey) {
				this.fetchRelationalData();
			}
		}
	},
	methods: {
		async fetchRelationalData() {
			if (this.relation?.collection_one?.collection) {
				this.loading = true;

				const res = await this.$api.getItem(
					this.relation.collection_one.collection,
					this.value
				);

				this.data = res.data;

				this.loading = false;
			}
		}
	}
};
</script>

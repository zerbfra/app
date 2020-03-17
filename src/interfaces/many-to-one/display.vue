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
				const relationField = template.split('.')[0].replace('{{', '');
				const langIndex = findIndex(
					value[relationField],
					o => o.lang == this.systemLanguage
				);
				const templateWithIndex = template.split('.').join(`.${langIndex}.`);
				template = templateWithIndex;
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

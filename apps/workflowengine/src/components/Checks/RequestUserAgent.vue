<!--
  - @copyright Copyright (c) 2019 Julius Härtl <jus@bitgrid.net>
  -
  - @author Julius Härtl <jus@bitgrid.net>
  -
  - @license GNU AGPL version 3 or any later version
  -
  - This program is free software: you can redistribute it and/or modify
  - it under the terms of the GNU Affero General Public License as
  - published by the Free Software Foundation, either version 3 of the
  - License, or (at your option) any later version.
  -
  - This program is distributed in the hope that it will be useful,
  - but WITHOUT ANY WARRANTY; without even the implied warranty of
  - MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
  - GNU Affero General Public License for more details.
  -
  - You should have received a copy of the GNU Affero General Public License
  - along with this program. If not, see <http://www.gnu.org/licenses/>.
  -
  -->

<template>
	<div>
		<Multiselect
			:value="currentValue"
			:placeholder="t('workflowengine', 'Select a user agent')"
			label="label"
			track-by="pattern"
			group-values="children"
			group-label="label"
			:options="options"
			:multiple="false"
			:tagging="false"
			@input="setValue">
			<template slot="singleLabel" slot-scope="props">
				<span class="option__icon" :class="props.option.icon" />
				<span class="option__title option__title_single">{{ props.option.label }}</span>
			</template>
			<template slot="option" slot-scope="props">
				<span class="option__icon" :class="props.option.icon" />
				<span class="option__title">{{ props.option.label }} {{ props.option.$groupLabel }}</span>
			</template>
		</Multiselect>
		<input v-if="!isPredefined"
			type="text"
			:value="currentValue.pattern"
			@input="updateCustom">
	</div>
</template>

<script>
import { Multiselect } from 'nextcloud-vue/dist/Components/Multiselect'
import valueMixin from '../../mixins/valueMixin'

export default {
	name: 'RequestUserAgent',
	components: {
		Multiselect
	},
	mixins: [
		valueMixin
	],
	data() {
		return {
			newValue: '',
			predefinedTypes: [
				{
					label: t('workflowengine', 'Sync clients'),
					children: [
						{ pattern: 'android', label: t('workflowengine', 'Android client'), icon: 'icon-phone' },
						{ pattern: 'ios', label: t('workflowengine', 'iOS client'), icon: 'icon-phone' },
						{ pattern: 'desktop', label: t('workflowengine', 'Desktop client'), icon: 'icon-desktop' },
						{ pattern: 'mail', label: t('workflowengine', 'Thunderbird & Outlook addons'), icon: 'icon-mail' }
					]
				}
			]
		}
	},
	computed: {
		options() {
			return [...this.predefinedTypes, this.customValue]
		},
		matchingPredefined() {
			return this.predefinedTypes
				.map(groups => groups.children)
				.flat()
				.find((type) => this.newValue === type.pattern)
		},
		isPredefined() {
			return !!this.matchingPredefined
		},
		customValue() {
			return {
				label: t('workflowengine', 'Others'),
				children: [
					{
						icon: 'icon-settings-dark',
						label: t('workflowengine', 'Custom user agent'),
						pattern: ''
					}
				]
			}
		},
		currentValue() {
			if (this.matchingPredefined) {
				return this.matchingPredefined
			}
			return {
				icon: 'icon-settings-dark',
				label: t('workflowengine', 'Custom user agent'),
				pattern: this.newValue
			}
		}
	},
	methods: {
		validateRegex(string) {
			var regexRegex = /^\/(.*)\/([gui]{0,3})$/
			var result = regexRegex.exec(string)
			return result !== null
		},
		setValue(value) {
			// TODO: check if value requires a regex and set the check operator according to that
			if (value !== null) {
				this.newValue = value.pattern
				this.$emit('input', this.newValue)
			}
		},
		updateCustom(event) {
			this.newValue = event.target.value
			this.$emit('input', this.newValue)
		}
	}
}
</script>

<style scoped src="./../../css/multiselect.css"></style>

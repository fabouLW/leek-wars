<template lang="html">
	<div v-if="constant" class="doc-constant" :class="{item: is_weapon || is_chip, deprecated: constant.deprecated}">
		<h2 v-if="!is_chip && !is_weapon">
			{{ constant.name }}
		</h2>

		<div v-if="constant.deprecated" v-dochash class="deprecated-message">
			{{ $t('doc.deprecated_constant') }}
			<span v-if="constant.replacement">{{ $t('doc.replaced_by', ['#' + CONSTANT_BY_ID[constant.replacement].name]) }}</span>
		</div>

		<router-link class="encyclo" :to="'/encyclopedia/' + $i18n.locale + '/' + constant.name" :title="'Encyclopédie > ' + constant.name + '()'">
			<v-icon class="book">mdi-book-open-page-variant</v-icon>
		</router-link>

		<item-preview v-if="is_chip" :item="LeekWars.items[constant.value]" />
		<item-preview v-else-if="is_weapon" :item="LeekWars.items[constant.value]" />

		<div v-if="new_constant">
			<markdown v-if="LeekWars.encyclopedia[$i18n.locale] && Object.keys(LeekWars.encyclopedia[$i18n.locale]).length" :content="new_constant.description" :pages="{}" mode="encyclopedia" />

			<div v-for="(section, s) in new_constant.primary" :key="s">
				<h4>{{ s }}</h4>
				<markdown :content="section" :pages="{}" mode="encyclopedia" />
			</div>
			<div v-if="Object.values(new_constant.secondary).length" class="expand" @click.stop="expanded = !expanded">Détails ({{ Object.values(new_constant.secondary).length }})<v-icon v-if="expanded">mdi-chevron-up</v-icon><v-icon v-else>mdi-chevron-down</v-icon></div>
			<div v-if="expanded" class="secondary">
				<div v-for="(section, s) in new_constant.secondary" :key="s">
					<h4>{{ s }}</h4>
					<markdown :content="section" :pages="{}" mode="encyclopedia" />
				</div>
			</div>
		</div>
		<div v-else>
			<div v-if="$te('doc.const_' + constant.name)" v-dochash v-code class="content" v-html="$t('doc.const_' + constant.name)"></div>
		</div>
		<h4>{{ $t('doc.value') }}</h4>
		<ul>
			<li>{{ constant.name }}
			<span class="argument">{{ $t('doc.arg_type_' + constant.type) }}</span> = {{ constant.value }}</li>
		</ul>
		<h4 v-if="chips.length + weapons.length">{{ $t('doc.items') }} ({{ chips.length + weapons.length }})</h4>
		<router-link v-for="chip of chips" :key="chip.id" :to="'/help/documentation/CHIP_' + chip.name.toUpperCase()">
			<rich-tooltip-item v-slot="{ on }" :item="LeekWars.items[chip.id]" :bottom="true" :instant="true" @input="$emit('input', $event)">
				<img :src="'/image/chip/' + chip.name + '.png'" class="item" v-on="on">
			</rich-tooltip-item>
		</router-link>
		<router-link v-for="weapon of weapons" :key="'w' + weapon.id" :to="'/help/documentation/WEAPON_' + weapon.name.toUpperCase()">
			<rich-tooltip-item v-slot="{ on }" :item="LeekWars.items[weapon.item]" :bottom="true" :instant="true" @input="$emit('input', $event)">
				<img :src="'/image/weapon/' + weapon.name + '.png'" class="item weapon" v-on="on">
			</rich-tooltip-item>
		</router-link>
	</div>
</template>

<script lang="ts">
	import ItemPreview from '@/component/market/item-preview.vue'
	import { Constant } from '@/model/constant'
	import { LeekWars } from '@/model/leekwars'
	import { Component, Prop, Vue, Watch } from 'vue-property-decorator'
	import RichTooltipItem from '@/component/rich-tooltip/rich-tooltip-item.vue'
	import { CHIPS } from '@/model/chips'
	import { CONSTANT_BY_ID } from '@/model/constant_by_id'
	import { locale } from '@/locale'
	import Markdown from '../encyclopedia/markdown.vue'

	@Component({ name: 'documentation-constant', components: { ItemPreview, RichTooltipItem, Markdown }})
	export default class DocumentationConstant extends Vue {
		@Prop() constant!: Constant
		CONSTANT_BY_ID = CONSTANT_BY_ID
		expanded: boolean = false
		new_constant: any = null

		@Watch('constant', {immediate: true})
		updateFun() {
			LeekWars.documentation(locale).then((functions) => {
				this.new_constant = functions[this.constant.name]
			})
		}

		get value_int() {
			return parseInt(this.constant.value, 10)
		}
		get is_weapon() {
			return this.constant.name.startsWith('WEAPON_')
		}
		get is_chip() {
			return this.constant.name.startsWith('CHIP_')
		}
		get chips() {
			const items = [] as any
			if (this.constant.deprecated) { return items }
			if (this.constant.name.startsWith("EFFECT_MODIFIER_")) {
				for (const i in CHIPS) {
					if (CHIPS[i].effects.some((e) => e.modifiers & this.value_int)) {
						items.push(CHIPS[i])
					}
				}
			} else if (this.constant.name.startsWith("EFFECT_") && !this.constant.name.startsWith("EFFECT_TARGET_")) {
				for (const i in CHIPS) {
					if (CHIPS[i].effects.some((e) => e.id === this.value_int)) {
						items.push(CHIPS[i])
					}
				}
			} else if (this.constant.name.startsWith("AREA_")) {
				for (const i in CHIPS) {
					if (CHIPS[i].area === this.value_int) {
						items.push(CHIPS[i])
					}
				}
			} else if (this.constant.name.startsWith("LAUNCH_TYPE_")) {
				for (const i in CHIPS) {
					if (CHIPS[i].launch_type === this.value_int) {
						items.push(CHIPS[i])
					}
				}
			}
			return items
		}
		get weapons() {
			const items = [] as any
			if (this.constant.name.startsWith("EFFECT_MODIFIER_")) {
				for (const i in LeekWars.weapons) {
					if (LeekWars.weapons[i].effects.some((e) => e.modifiers & this.value_int)) {
						items.push(LeekWars.weapons[i])
					}
				}
			} else if (this.constant.name.startsWith("EFFECT_") && !this.constant.name.startsWith("EFFECT_TARGET_")) {
				for (const i in LeekWars.weapons) {
					if (LeekWars.weapons[i].effects.some((e) => e.id === this.value_int) || LeekWars.weapons[i].passive_effects.some((e) => e.id === this.value_int)) {
						items.push(LeekWars.weapons[i])
					}
				}
			} else if (this.constant.name.startsWith("AREA_")) {
				for (const i in LeekWars.weapons) {
					if (LeekWars.weapons[i].area === this.value_int) {
						items.push(LeekWars.weapons[i])
					}
				}
			} else if (this.constant.name.startsWith("LAUNCH_TYPE_")) {
				for (const i in LeekWars.weapons) {
					if (LeekWars.weapons[i].launch_type === this.value_int) {
						items.push(LeekWars.weapons[i])
					}
				}
			}
			return items
		}
	}
</script>

<style lang="scss" scoped>
	h2 {
		margin-bottom: 10px;
		font-size: 17px;
		color: #333;
	}
	h4 {
		font-weight: 500;
		margin: 0;
		color: #666;
		margin-top: 10px;
		margin-bottom: 8px;
		font-size: 15px;
	}
	.doc-constant .item {
		height: 50px;
		margin: 0 2px;
	}
	.item.weapon {
		object-fit: contain;
		max-width: 120px;
		max-height: 50px;
	}
	.argument {
		color: #0000D0;
		font-weight: bold;
	}
	::v-deep a {
		color: #5fad1b;
		font-weight: 500;
		&:hover {
			text-decoration: underline;
		}
	}
	.doc-constant .md {
		padding: 0 !important;
		::v-deep pre code {
			margin-bottom: 0;
		}
		::v-deep p {
			font-size: 15px;
		}
	}
	.encyclo {
		position: absolute;
		right: 12px;
		top: 15px;
		color: #aaa;
		i {
			font-size: 18px;
		}
		&:hover {
			text-decoration: none;
			color: #000;
		}
	}
</style>
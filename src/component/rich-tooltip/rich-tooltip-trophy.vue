<template>
	<v-menu v-model="value" :close-on-content-click="false" :width="280" offset-overflow :nudge-top="0" :open-delay="_open_delay" :close-delay="_close_delay" :top="!bottom" transition="none" :bottom="bottom" :open-on-hover="!locked" offset-y @input="$emit('input', $event)">
		<template v-slot:activator="{ on }">
			<slot :on="on"></slot>
		</template>
		<div class="card" @mouseenter="mouse = true" @mouseleave="mouse = false">
			<trophy ref="preview" :trophy="trophy" @input="setParent" />
		</div>
	</v-menu>
</template>

<script lang="ts">
	import Trophy from '@/component/trophies/trophy.vue'
	import { Component, Prop, Vue } from 'vue-property-decorator'

	@Component({ components: { Trophy } })
	export default class RichTooltipTrophy extends Vue {
		@Prop({required: true}) trophy!: any
		@Prop() bottom!: boolean
		@Prop() instant!: boolean
		locked: boolean = false
		mouse: boolean = false
		value: boolean = false

		get _open_delay() {
			return this.instant ? 0 : 500
		}
		get _close_delay() {
			return this.instant ? 0 : 0
		}

		setParent(event: boolean) {
			this.locked = event
			if (!event && !this.mouse) {
				this.value = false
				this.$emit('input', false)
			}
		}
	}
</script>

<style lang="scss" scoped>
	.card {
		width: 280px;
		background: white;
	}
	.trophy {
		display: block;
	}
</style>
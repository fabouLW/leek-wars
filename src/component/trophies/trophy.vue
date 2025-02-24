<template>
	<router-link v-ripple :class="{unlocked: trophy.unlocked, locked: !trophy.unlocked, card: trophy.unlocked}" :to="'/trophy/' + trophy.code" class="trophy">
		<div class="flex">
			<img :src="'/image/trophy/' + trophy.code + '.svg'" class="image">
			<div class="info">
				<div class="header">
					<div class="name">{{ $t('trophy.' + trophy.code) }}</div>
					<div v-if="trophy.points" class="points">{{ trophy.points }}</div>
				</div>
				<div class="description">{{ trophy.description }}</div>
				<div v-if="trophy.habs" class="habs">{{ trophy.habs | number }} <span class="hab"></span></div>
				<tooltip v-if="trophy.progression != null">
					<template v-slot:activator="{ on }">
						<div class="trophy-bar" :class="{full: trophy.unlocked}" v-on="on">
							<div :style="{width: Math.floor(100 * Math.min(trophy.threshold, trophy.progression) / trophy.threshold) + '%'}" class="bar striked"></div>
						</div>
					</template>
					{{ trophy.progression | number }} / {{ trophy.threshold | number }}
				</tooltip>
			</div>
		</div>
		<div class="unlock">
			<img v-if="trophy.in_fight" class="fight-icon" src="/image/trophy/winner.svg" :title="$t('trophy.unlockable_fight')">
			<template v-if="trophy.unlocked">
				<i18n v-if="trophy.fight" tag="span" class="date" path="main.unlocked_the">
					<router-link slot="date" :to="'/fight/' + trophy.fight" class="fight">{{ trophy.date | date }}</router-link>
				</i18n>
				<i18n v-else tag="span" class="date" path="main.unlocked_the">
					<span slot="date">{{ trophy.date | date }}</span>
				</i18n>
			</template>
			<span class="rarity"><span v-if="trophy.unlocked"> • </span>{{ trophy.total }} • {{ (trophy.rarity * 100).toPrecision(2) }}%</span>
		</div>
	</router-link>
</template>

<script lang="ts">
	import { mixins } from '@/model/i18n'
	import { LeekWars } from '@/model/leekwars'
	import { Component, Prop, Vue, Watch } from 'vue-property-decorator'

	@Component({ name: 'trophy' })
	export default class Trophy extends Vue {
		@Prop({ required: true }) trophy: any
	}
</script>

<style lang="scss" scoped>
	.content:not(.first) {
		padding: 8px;
	}
	.v-input--switch {
		margin-left: 8px;
		margin-right: -12px;
	}
	.global-percent {
		font-size: 40px;
	}
	.global-count {
		font-size: 30px;
		color: #888;
		float: right;
		line-height: 55px;
	}
	.global-bar {
		height: 14px;
		position: relative;
		background: white;
		border-radius: 6px;
		margin: 5px 0;
		border: 1px solid #ddd;
		.bar {
			height: 12px;
			width: 0;
			background: #008fbb;
			position: absolute;
			border-radius: 6px;
		}
	}
	.panel ::v-deep .actions {
		flex: 1;
	}
	.category-bar-wrapper {
		text-align: right;
		white-space: nowrap;
		display: flex;
		width: 100%;
		.stats {
			display: inline-block;
			color: white;
			font-size: 16px;
			margin: 9px 10px;
		}
	}
	.category-bar {
		height: 12px;
		position: relative;
		background: white;
		border-radius: 6px;
		flex: 1;
		margin-top: 12px;
		.bar {
			height: 12px;
			width: 0;
			background: #30bb00;
			position: absolute;
			border-radius: 6px;
		}
	}
	.bar {
		transition: all ease 0.3s;
	}
	.trophies {
		display: grid;
		grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
		grid-gap: 8px;
		padding: 8px;
	}
	#app.app .trophies {
		grid-template-columns: repeat(auto-fill, minmax(165px, 1fr));
	}
	.trophy {
		padding: 6px;
		.image {
			width: 50px;
			height: 50px;
			float: left;
			margin-right: 7px;
			margin-bottom: 2px;
		}
		.info {
			flex: 1;
		}
		.header {
			display: flex;
			justify-content: space-between;
			align-items: center;
			margin-bottom: 5px;
		}
		.points {
			border: 1px solid #aaa;
			padding: 1px 4px;
			border-radius: 4px;
			margin-left: 5px;
			font-weight: 500;
		}
		.name {
			font-size: 16px;
		}
		.description {
			color: #555;
			font-size: 13px;
		}
		.habs {
			color: #555;
			font-size: 13px;
			padding-top: 5px;
			.hab {
				width: 12px;
				height: 12px;
				margin-bottom: 1px;
				background-size: cover;
			}
		}
		.fight-icon {
			width: 16px;
			height: 16px;
			opacity: 0.7;
			margin-right: 2px;
		}
		.trophy-bar {
			height: 10px;
			position: relative;
			background: white;
			border-radius: 6px;
			margin-top: 6px;
			border: 1px solid #ddd;
			.bar {
				height: 8px;
				border-radius: 6px;
				position: absolute;
				background: #30bb00;
			}
			&.full .bar {
				background: #ddd;
			}
		}
		.unlock {
			display: flex-wrap;
			align-items: center;
			margin-top: 4px;
		}
		.date, .rarity {
			color: #888;
			font-size: 13px;
			font-style: italic;
			.fight {
				color: black;
			}
		}
	}
	#app.app .trophy {
		.image {
			width: 38px;
			height: 38px;
		}
		.trophy-bar {
			margin-left: 0;
			width: 100%;
		}
	}
	.trophy.locked {
		.image {
			opacity: 0.8;
		}
	}
	.list-icon {
		margin-right: 10px;
	}
</style>
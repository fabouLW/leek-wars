<template lang="html">
	<div :class="{generating: fight.status == 0, win: fight.result == 'win', defeat: fight.result == 'defeat', draw: fight.result == 'draw'}" class="fight">
		<div v-if="fight.type == FightType.BATTLE_ROYALE" class="fighters">
			<div class="fighter left"><div>Battle</div></div>
			<router-link :to="'/fight/' + fight.id" class="center">
				<v-icon v-if="fight.status == 0" class="timersand">mdi-timer-sand-empty</v-icon>
				<v-icon v-else>mdi-sword-cross</v-icon>
			</router-link>
			<div class="fighter right"><div>Royale</div></div>
		</div>
		<div v-else class="fighters">
			<router-link v-if="fight.type == FightType.SOLO && fight.leeks1[0]" :to="'/leek/' + fight.leeks1[0].id" class="fighter">
				<rich-tooltip-leek :id="fight.leeks1[0].id" v-slot="{ on }">
					<div v-on="on">{{ fight.leeks1[0].name }}</div>
				</rich-tooltip-leek>
			</router-link>
			<router-link v-else-if="fight.type == FightType.FARMER" :to="'/farmer/' + fight.farmer1" class="fighter">
				<rich-tooltip-farmer :id="fight.farmer1" v-slot="{ on }">
					<div v-on="on">({{ fight.farmer1_name }})</div>
				</rich-tooltip-farmer>
			</router-link>
			<router-link v-else-if="fight.type == FightType.TEAM" :to="'/team/' + fight.team1" class="fighter">
				<rich-tooltip-composition :id="fight.composition1" v-slot="{ on }">
					<div v-on="on">[{{ fight.team1_name }}]</div>
				</rich-tooltip-composition>
			</router-link>
			<router-link :to="'/fight/' + fight.id" class="center">
				<v-icon v-if="fight.status == 0" class="timersand">mdi-timer-sand-empty</v-icon>
				<v-icon v-else-if="fight.context == FightContext.CHALLENGE">mdi-flag-outline</v-icon>
				<v-icon v-else-if="fight.context == FightContext.TOURNAMENT">mdi-trophy-outline</v-icon>
				<img v-else src="/image/icon/black/garden.png">
			</router-link>
			<router-link v-if="fight.type == FightType.SOLO && fight.leeks2[0]" :to="'/leek/' + fight.leeks2[0].id" class="fighter">
				<rich-tooltip-leek :id="fight.leeks2[0].id" v-slot="{ on }">
					<div v-on="on">{{ fight.leeks2[0].name }}</div>
				</rich-tooltip-leek>
			</router-link>
			<router-link v-else-if="fight.type == FightType.FARMER" :to="'/farmer/' + fight.farmer2" class="fighter">
				<rich-tooltip-farmer :id="fight.farmer2" v-slot="{ on }">
					<div v-on="on">({{ fight.farmer2_name }})</div>
				</rich-tooltip-farmer>
			</router-link>
			<router-link v-else-if="fight.type == FightType.TEAM" :to="'/team/' + fight.team2" class="fighter">
				<rich-tooltip-composition :id="fight.composition2" v-slot="{ on }">
					<div v-on="on">[{{ fight.team2_name }}]</div>
				</rich-tooltip-composition>
			</router-link>
		</div>
		<div class="time">{{ LeekWars.formatDuration(fight.date) }}</div>
	</div>
</template>

<script lang="ts">
	import { Fight, FightContext, FightType } from '@/model/fight'
	import { Component, Prop, Vue } from 'vue-property-decorator'
	import RichTooltipFarmer from '@/component/rich-tooltip/rich-tooltip-farmer.vue'
	import RichTooltipLeek from '@/component/rich-tooltip/rich-tooltip-leek.vue'
	import RichTooltipComposition from '@/component/rich-tooltip/rich-tooltip-composition.vue'

	@Component({ name: 'fight-history', components: { RichTooltipFarmer, RichTooltipLeek, RichTooltipComposition } })
	export default class FightHistory extends Vue {
		@Prop() fight!: Fight
		FightType = FightType
		FightContext = FightContext
	}
</script>

<style lang="scss" scoped>
	.fight {
		margin: 5px;
		color: #333;
		text-align: center;
		border-radius: 3px;
		font-size: 15px;
		height: 42px;
		white-space: nowrap;
		background: white;
		position: relative;
		.center {
			background: rgba(255, 255, 255, 0.3);
			&:hover {
				background: rgba(255, 255, 255, 0.75);
			}
			flex: 42px 0 0;
			height: 42px;
			img {
				width: 22px;
				height: 22px;
				margin: 10px 6px;
				opacity: 0.8;
			}
			i {
				color: #333;
				line-height: 42px;
				font-size: 26px;
				&.timersand {
					animation: rotate 2s linear infinite;
				}
			}
		}
		.fighters {
			height: 42px;
			display: flex;
		}
		.fighter {
			padding: 4px 0;
			height: 30px;
			line-height: 30px;
			flex: 1;
			overflow: hidden;
			&.left {
				text-align: right;
			}
			&.right {
				text-align: left;
			}
			div {
				padding: 0 6px;
				overflow: hidden;
				text-overflow: ellipsis;
			}
		}
		.time {
			position: absolute;
			bottom: 2px;
			right: 8px;
			color: #777;
			font-size: 11px;
			text-align: right;
		}
	}
	.win {
		background-color: #b6f182;
	}
	.draw {
		background-color: #dcdcdc;
	}
	.defeat {
		background-color: #ffb3ae;
	}
	.generating {
		background-color: white;
	}
	@keyframes rotate {
		0% { transform: rotate(0); }
		20% { transform: rotate(180deg); }
		100% { transform: rotate(180deg); }
	}
</style>
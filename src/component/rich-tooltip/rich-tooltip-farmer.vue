<template>
	<v-menu ref="menu" v-model="value" :close-on-content-click="false" offset-overflow :disabled="disabled || id <= 0" :nudge-width="expand_leeks ? 500 : 200" :nudge-top="0" :open-delay="_open_delay" :close-delay="_close_delay" :top="!bottom" :bottom="bottom" :transition="instant ? 'none' : 'my-transition'" :open-on-hover="!locked" offset-y @input="open($event)">
		<template v-slot:activator="{ on }">
			<slot :on="on"></slot>
		</template>
		<div class="card" @mouseenter="mouse = true" @mouseleave="mouse = false">
			<loader v-if="!farmer" :size="30" />
			<template v-else>
				<div class="flex">
					<router-link :to="'/farmer/' + farmer.id">
						<avatar :farmer="farmer" />
					</router-link>
					<div class="info">
						<span class="name">
							<img :src="farmer.connected ? '/image/connected.png' : '/image/disconnected.png'">
							<router-link :to="'/farmer/' + farmer.id" :class="farmer.color" class="text">{{ farmer.name }}</router-link>
							<router-link v-if="farmer.team" :to="'/team/' + farmer.team.id">
								<emblem :team="farmer.team" :title="farmer.team.name" />
							</router-link>
							<flag v-if="farmer.country" :code="farmer.country" class="country" />
							<lw-title v-if="farmer.title.length" :title="farmer.title" />
							<div class="spacer"></div>
							<v-btn v-if="!$store.state.farmer || id != $store.state.farmer.id" icon small @click="sendMessage()">
								<v-icon>mdi-chat</v-icon>
							</v-btn>
						</span>
						<div>
							<router-link :to="'/trophies/' + farmer.id" class="stat">
								<img class="icon" src="/image/icon/grey/trophy.png">{{ farmer.points | number }}
							</router-link>
							<router-link v-if="farmer.forum_messages" :to="'/search?farmer=' + farmer.name + '&order=date'" class="stat">
								<img class="icon" src="/image/forum.png">{{ $t('main.n_messages', [farmer.forum_messages]) }}
							</router-link>
						</div>
					</div>
				</div>
				<talent :id="farmer.id" :talent="farmer.talent" category="farmer" />
				<span class="talent-more">({{ farmer.talent_more >= 0 ? '+' + farmer.talent_more : farmer.talent_more }})</span>
				<ranking-badge v-if="farmer && farmer.ranking && farmer.ranking <= 1000 && farmer.in_garden" :id="farmer.id" :ranking="farmer.ranking" category="farmer" />
				<span class="level">• {{ $t('main.level_n', [farmer.total_level]) }}</span>
				<v-btn class="expand" icon small @click="expand_leeks = !expand_leeks">
					<v-icon v-if="expand_leeks">mdi-chevron-up</v-icon>
					<v-icon v-else>mdi-chevron-down</v-icon>
				</v-btn>
				<table v-if="expand_leeks" class="leeks">
					<tr>
						<th>{{ $t('main.name') }}</th>
						<th>{{ $t('main.level') }}</th>
						<th><img src="/image/talent.png"></th>
						<th v-for="c in LeekWars.characteristics" :key="c" class="c"><img :src="'/image/charac/small/' + c + '.png'" :class="{zero: sums[c] === 0}"></th>
					</tr>
					<tr v-for="leek in farmer.leeks" :key="leek.id">
						<td class="leek-name">
							<rich-tooltip-leek :id="leek.id" v-slot="{ on }" :bottom="true" @input="setParent">
								<router-link :to="'/leek/' + leek.id">
									<span v-on="on">{{ leek.name }}</span>
								</router-link>
							</rich-tooltip-leek>
						</td>
						<td>{{ leek.level }}</td>
						<td><b>{{ leek.talent }}</b></td>
						<td v-for="c in LeekWars.characteristics" :key="c" :class="['color-' + c, leek[c] === 0 ? 'zero' : '']" class="c">{{ leek[c] }}</td>
					</tr>
				</table>
			</template>
		</div>
	</v-menu>
</template>

<script lang="ts">
	import { Farmer } from '@/model/farmer'
	import { LeekWars } from '@/model/leekwars'
	import { store } from '@/model/store'
	import { Component, Prop, Vue, Watch } from 'vue-property-decorator'
	import RichTooltipLeek from '@/component/rich-tooltip/rich-tooltip-leek.vue'
	const LWTitle = () => import('@/component/title/title.vue')

	@Component({ components: { RichTooltipLeek, 'lw-title': LWTitle } })
	export default class RichTooltipFarmer extends Vue {
		@Prop({required: true}) id!: number
		@Prop() disabled!: boolean
		@Prop() bottom!: boolean
		@Prop() instant!: boolean
		content_created: boolean = false
		farmer: Farmer | null = null
		expand_leeks: boolean = false
		sums: {[key: string]: number} = {}
		locked: boolean = false
		mouse: boolean = false
		value: boolean = false

		get _open_delay() {
			return this.instant ? 0 : 500
		}
		get _close_delay() {
			return this.instant ? 0 : 0
		}
		@Watch('id')
		update() {
			this.farmer = null
			this.content_created = false
		}
		open(v: boolean) {
			this.$emit('input', v)
			this.expand_leeks = localStorage.getItem('richtooltipfarmer/expanded') === 'true'
			if (this.content_created) { return }
			this.content_created = true
			if (this.id > 0 && !this.farmer) {
				LeekWars.get<Farmer>('farmer/rich-tooltip/' + this.id).then(farmer => {
					this.farmer = farmer
					for (const c of LeekWars.characteristics) {
						Vue.set(this.sums, c, Object.values(this.farmer.leeks).reduce((sum: number, leek: any) => sum + leek[c], 0))
					}
					if (this.expand_leeks) {
						(this.$refs.menu as any).onResize()
					}
				})
			}
		}

		sendMessage() {
			if (!this.farmer) { return }
			LeekWars.get('message/find-conversation/' + this.farmer.id).then(conversation => {
				store.commit('new-conversation', conversation)
				this.$router.push('/messages/conversation/' + conversation.id)
			}).error(() => {
				if (!this.farmer) { return }
				this.$router.push('/messages/new/' + this.farmer.id + '/' + this.farmer.name + '/' + this.farmer.avatar_changed)
			})
		}

		@Watch('expand_leeks')
		updateExpand() {
			localStorage.setItem('richtooltipfarmer/expanded', this.expand_leeks ? 'true' : 'false')
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
		padding: 8px;
	}
	.avatar {
		width: 50px;
		height: 50px;
		flex-grow: 0;
		flex-basis: 50px;
	}
	.spacer {
		flex: 1;
	}
	.info {
		flex: 1;
		padding-left: 10px;
		.icon {
			width: 15px;
			margin-right: 4px;
			vertical-align: middle;
			padding-bottom: 2px;
		}
		.stat {
			padding-right: 4px;
			font-size: 13px;
			img {
				opacity: 0.5;
			}
		}
		.title {
			font-size: 14px;
		}
	}
	.name {
		display: flex;
		align-items: center;
		font-size: 16px;
		height: 25px;
		margin-right: -4px;
		margin-bottom: 6px;
		img, .country {
			height: 17px;
			margin-right: 3px;
		}
		i {
			font-size: 18px;
		}
		.emblem {
			margin-left: 5px;
			margin-top: 3px;
		}
		.country {
			margin-left: 5px;
			margin-top: 1px;
		}
	}
	.talent-more {
		font-size: 15px;
		margin-left: 5px;
		color: #888;
		display: inline-block;
		vertical-align: top;
		margin-top: 10px;
	}
	.badge {
		margin-bottom: 2px;
		vertical-align: bottom;
		margin-right: 0;
	}
	.level {
		display: inline-block;
		font-size: 15px;
		font-weight: 500;
		margin-left: 5px;
		vertical-align: top;
		margin-top: 10px;
		color: #555;
	}
	.expand {
		vertical-align: top;
		margin-top: 5px;
		margin-left: 10px;
	}
	.leeks {
		text-align: left;
		width: calc(100% + 16px);
		margin: 0 -8px;
		tr {
			border-bottom: 1px solid #ddd;
		}
		tr:nth-child(2n) {
			background: #f7f7f7;
		}
		td, th {
			padding: 3px 4px;
		}
		td:first-child, th:first-child {
			padding-left: 10px;
			padding-right: 10px;
		}
		img {
			width: 18px;
		}
		.c {
			width: 30px;
			font-weight: 500;
		}
		.zero {
			filter: saturate(0);
			opacity: 0.3;
		}
		.leek-name {
			max-width: 120px;
			text-overflow: ellipsis;
			overflow: hidden;
		}
	}
</style>
<template>
	<div :class="{subtitle: LeekWars.subtitle, full: !LeekWars.lightBar || LeekWars.menuExpanded}" class="app-bar">
		<div v-ripple :class="{back: LeekWars.splitBack}" class="menu-button" @click="mainButton">
			<div>
				<div class="bar"></div>
				<div class="bar"></div>
				<div class="bar"></div>
			</div>
		</div>
		<div v-if="!LeekWars.lightBar || LeekWars.menuExpanded" class="title-wrapper" @click="LeekWars.toggleMenu">
			<div class="title">{{ LeekWars.title }}</div>
			<div v-show="LeekWars.subtitle" class="subtitle">{{ LeekWars.subtitle }}</div>
		</div>
		<div v-if="!LeekWars.lightBar || LeekWars.menuExpanded" class="actions-wrapper">
			<div class="static-actions">
				<div v-show="LeekWars.menuExpanded || $store.state.unreadMessages > 0" v-ripple class="action header-button mobile messages-button" @click="$router.push('/messages'); LeekWars.closeMenu()">
					<v-icon>mdi-message-outline</v-icon>
					<span v-show="$store.state.unreadMessages > 0" class="counter messages-counter">{{ $store.state.unreadMessages }}</span>
				</div>
				<div v-show="LeekWars.menuExpanded || $store.state.unreadNotifications > 0" v-ripple class="action header-button mobile notifications-button">
					<v-menu :nudge-bottom="0" :max-width="400" :max-height="434" bottom offset-y @input="readNotifications">
						<template v-slot:activator="{ on }">
							<div class="header-button notifications-button" v-on="on">
								<v-icon>mdi-information-outline</v-icon>
								<span v-show="$store.state.unreadNotifications > 0" class="counter notifications-counter">{{ $store.state.unreadNotifications }}</span>
							</div>
						</template>
						<div class="dialog">
							<div class="dialog-items">
								<notification v-for="notification in $store.state.notifications" :key="notification.id" :notification="notification" @click.native="readNotification(notification)" />
							</div>
							<router-link to="/notifications" class="see-all" @click.native="LeekWars.closeMenu()">{{ $t('main.all_notifications') }}</router-link>
						</div>
					</v-menu>
				</div>
				<router-link v-show="LeekWars.menuExpanded" v-ripple to="/settings" class="action header-button mobile settings" @click.native="closeMenu">
					<v-icon>mdi-settings-outline</v-icon>
				</router-link>
			</div>
			<div v-show="!LeekWars.menuExpanded" class="actions">
				<div v-for="(action, a) in LeekWars.actions" :key="a" v-ripple class="tab action" @click="action.click($event)">
					<v-icon v-if="action.icon" class="action">{{ action.icon }}</v-icon>
					<img v-else :src="'/image/' + action.image" class="action">
				</div>
			</div>
		</div>
		<div class="dark" :class="{visible: dark}"></div>
	</div>
</template>

<script lang="ts">
	import { LeekWars } from '@/model/leekwars'
	import { Component, Vue } from 'vue-property-decorator'

	@Component({ name: 'lw-bar' })
	export default class Bar extends Vue {
		dark: boolean = false
		mainButton() {
			if (LeekWars.menuExpanded || !LeekWars.splitBack) {
				LeekWars.toggleMenu()
			} else {
				this.$root.$emit('back')
			}
		}
		closeMenu() {
			LeekWars.menuExpanded = false
			LeekWars.dark = 0
		}
		readNotifications(e: any) {
			if (e === false && this.$store.state.unreadNotifications) {
				LeekWars.post('notification/read-all')
				this.$store.commit('read-notifications')
			}
			this.dark = e
		}
		readNotification(notification: any) {
			LeekWars.post('notification/read', {notification_id: notification.id})
		}
	}
</script>

<style lang="scss" scoped>
	.app-bar {
		position: fixed;
		top: 0;
		left: 0;
		height: 56px;
		z-index: 6;
		background: #4b9e06;
		color: white;
		line-height: 55px;
		font-size: 18px;
		overflow: hidden;
		text-overflow: ellipsis;
		word-break: break-all;
		white-space: nowrap;
		display: flex;
		box-shadow: 0 2px 2px 0 rgba(0,0,0,.07);
		&.full {
			right: 0;
		}
	}
	#app:not(.connected) .app-bar {
		display: none;
	}
	.app-bar .menu-button {
		width: 61px;
		padding: 14px 20px;
		padding-right: 14px;
	}
	.app-bar .menu-button .bar {
		width: 20px;
		height: 2px;
		border-radius: 2px;
		margin: 5px 0;
		background: white;
		transition: all ease 400ms;
	}
	.app-bar .menu-button.back .bar:first-child {
		transform: translateY(2px) rotate(-38deg);
	}
	.app-bar .menu-button.back .bar:nth-child(2) {
		opacity: 0;
	}
	.app-bar .menu-button.back .bar:last-child {
		transform: rotate(38deg);
	}
	#app.app.menu-expanded .app-bar .menu-button .bar:first-child {
		transform: translateY(7px) rotate(45deg);
	}
	#app.app.menu-expanded .app-bar .menu-button .bar:nth-child(2) {
		opacity: 0;
	}
	#app.app.menu-expanded .app-bar .menu-button .bar:last-child {
		transform: translateY(-7px) rotate(-45deg);
	}
	.app-bar .title-wrapper {
		flex: 1;
		text-overflow: ellipsis;
		overflow-x: hidden;
		padding-left: 4px;
	}
	.app-bar .title {
		text-overflow: ellipsis;
		overflow-x: hidden;
	}
	.app-bar.subtitle .title {
		line-height: 25px;
		padding-top: 7px;
		font-weight: bold;
	}
	.app-bar .subtitle {
		text-overflow: ellipsis;
		overflow-x: hidden;
		line-height: 15px;
		font-size: 12px;
		display: none;
	}
	.app-bar.subtitle .subtitle {
		display: block;
	}
	.app-bar .actions-wrapper {
		padding-right: 4px;
	}
	.app-bar .actions, .app-bar .static-actions {
		display: inline-block;
	}
	#app.app.menu-expanded .app-bar .static-actions .action {
		display: inline-block;
	}
	.action {
		display: inline-block;
		line-height: normal;
		vertical-align: top;
		position: relative;
	}
	.action i {
		font-size: 26px;
		padding: 15px 12px;
		color: white;
	}
	.action img {
		width: 54px;
		height: 54px;
		opacity: 1;
		padding: 15px 14px;
	}
	.app-bar.content .action.list:not(.content),
	.app-bar.list .action.content:not(.list),
	.app-bar .action.hidden {
		display: none;
	}
	.app-bar .action.visible {
		display: inline-block;
	}
	.counter {
		position: absolute;
		top: 7px;
		right: 5px;
		padding: 4px 3px;
		background: #ff6f00;
		color: #fff;
		border-radius: 5px;
		height: 20px;
		line-height: 12px;
	}
	.dark {
		position: fixed;
		top: 56px;
		left: 0;
		right: 0;
		height: 0;
		background: #0000;
		transition: background 200ms ease;
		&.visible {
			height: 100vh;
			background: #0007;
		}
	}
	.dialog {
		background: #f2f2f2;
		min-width: min(400px, 100vw);
	}
	.dialog-items {
		overflow-y: auto;
		max-height: 400px;
	}
	.see-all {
		padding: 8px;
		display: block;
		text-align: center;
		color: #777;
	}
	.see-all:hover {
		background: white;
	}
</style>
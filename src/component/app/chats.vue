<template>
	<div v-if="$store.state.farmer" class="chats">
		<div v-for="(window, i) in LeekWars.chatWindows" :key="window.name" :class="{expanded: window.expanded, unread: $refs.chats && $refs.chats[i] && !$refs.chats[i].read}" class="window">
			<div v-if="$store.state.chat[window.id]" class="header">
				<router-link v-if="window.type === ChatType.PM" v-ripple :to="'/farmer/' + getFarmer(window).id">
					<avatar :farmer="getFarmer(window)" class="image" />
				</router-link>
				<router-link v-else-if="window.type === ChatType.TEAM" v-ripple :to="'/team/' + $store.state.farmer.team.id">
					<emblem :team="$store.state.farmer.team" class="image" />
				</router-link>
				<div v-ripple class="title" @click="toggleExpanded(window, i)">{{ window.title }}</div>
				<v-icon class="close" @click="LeekWars.removeChat(i)">mdi-close</v-icon>
			</div>
			<chat :id="window.id" ref="chats" class="chat" @send="sendMessage($event, window.name)" />
		</div>
	</div>
</template>

<script lang="ts">
	const ChatElement = () => import(/* webpackChunkName: "chat" */ `@/component/chat/chat.vue`)
	import { ChatType, ChatWindow } from '@/model/chat'
	import { LeekWars } from '@/model/leekwars'
	import { store } from '@/model/store'
	import { Component, Vue, Watch } from 'vue-property-decorator'

	@Component({
		components: { chat: ChatElement }
	})
	export default class Chats extends Vue {
		ChatType = ChatType

		@Watch('LeekWars.chatWindows', {deep: true})
		update() {
			localStorage.setItem('chats', JSON.stringify(LeekWars.chatWindows))
		}

		toggleExpanded(window: ChatWindow, index: number) {
			window.expanded = !window.expanded
			setTimeout(() => ((this.$refs.chats as Vue[])[index] as any).updateScroll())
		}

		getFarmer(window: ChatWindow) {
			const chat = store.state.chat[window.id]
			if (chat) {
				const f = chat.farmers.find(f => f.id !== store.state.farmer!.id)
				if (f) { return f }
			}
			return {id: -1}
		}

		sendMessage(message: string, id: number) {
			LeekWars.post('message/send-message', {conversation_id: id, message})
		}
	}
</script>

<style lang="scss" scoped>
	.chats {
		position: fixed;
		bottom: 0;
		right: 0;
		z-index: 1000;
		display: inline-flex;
		flex-direction: row-reverse;
		align-items: flex-end;
		pointer-events: none;
	}
	.window {
		width: 300px;
		background: #f2f2f2;
		margin-right: 15px;
		border-top-left-radius: 7px;
		border-top-right-radius: 7px;
		box-shadow: 0px 3px 5px -1px rgba(0,0,0,0.2), 0px 5px 8px 0px rgba(0,0,0,0.14), 0px 1px 14px 0px rgba(0,0,0,0.12);
		pointer-events: all;
		.header {
			background: #2a2a2a;
			color: white;
			border-top-left-radius: 6px;
			border-top-right-radius: 6px;
			display: flex;
			user-select: none;
			cursor: pointer;
			.image {
				height: 34px;
				margin: 4px;
				vertical-align: bottom;
			}
			.title {
				padding: 10px;
				padding-left: 5px;
				font-size: 18px;
				flex: 1;
				overflow: hidden;
				text-overflow: ellipsis;
			}
			.title:first-child {
				padding-left: 10px;
			}
			.v-icon {
				padding: 8px;
			}
		}
		.chat {
			height: calc(370px - 42px);
		}
		&:not(.expanded) .chat {
			display: none;
		}
	}
	.window.unread .header {
		animation: unread 2.5s infinite;
	}
	@keyframes unread {
		0% { background:#5fad1b; }
		50% { background:#2a2a2a; }
		100% { background:#5fad1b; }
	}
</style>
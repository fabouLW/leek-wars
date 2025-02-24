<template lang="html">
	<v-list v-if="filterOptions === null" dense>
		<v-list-item v-for="(command, c) of commands" :key="command.name" v-ripple class="command" :class="{selected: index === c}" @click="$emit('command', command.name)">
			<v-list-item-content>
				<v-list-item-title>/{{ command.name }}</v-list-item-title>
				<v-list-item-subtitle>{{ command.description }}</v-list-item-subtitle>
			</v-list-item-content>
		</v-list-item>
	</v-list>
	<v-list v-else-if="options.length" dense>
		<v-list-item v-for="option of options" :key="option.name" v-ripple class="command" @click="$emit('command', commands[0].name + ':' + option.name)">
			<v-list-item-content>
				<v-list-item-title>/{{ commands[0].name }}:{{ option.name }}</v-list-item-title>
				<v-list-item-subtitle>{{ option.description }}</v-list-item-subtitle>
			</v-list-item-content>
		</v-list-item>
	</v-list>
</template>

<script lang="ts">
	import { Command, Commands } from '@/model/commands'
	import { Component, Prop, Vue, Watch } from 'vue-property-decorator'

	@Component({ name: 'chat-commands' })
	export default class ChatCommands extends Vue {

		@Prop() filter!: string
		commands = Commands.commands
		options: any[] = []
		filterOptions: string | null = null
		index: number = 0

		mounted() {
			Commands.init()
		}

		@Watch('filter', {immediate: true})
		update() {
			const parts = this.filter.toLowerCase().split(':')
			const filterCommand = parts[0]
			this.filterOptions = parts.length > 1 ? parts[1] : null
			this.options = []
			this.commands = Commands.commands.filter(command =>
				(parts.length === 1 && command.name.indexOf(filterCommand) === 0) || command.name === filterCommand
			)
			if (this.commands.length === 1) {
				const command = this.commands[0]
				if (command.options) {
					if (this.filterOptions === null) {
						this.options = command.options
					} else {
						this.options = command.options.filter(option => option.nameLower.indexOf(this.filterOptions || '') === 0)
					}
				}
			}
		}
		getSelected(): Command {
			return this.commands[this.index]
		}
		getSelectedOption() {
			return this.commands[this.index].options ? this.options[this.index] : null
		}
		selectFirst() {
			let command = this.commands[this.index].name
			if (this.options.length) {
				command += ':' + this.options[0].name
			}
			this.$emit('command', command)
		}
		up() {
			this.index--
			if (this.index < 0) this.index = this.commands.length - 1
		}
		down() {
			this.index = (this.index + 1) % this.commands.length
		}
	}
</script>

<style lang="scss" scoped>
	.command.selected {
		background: #eee;
	}
</style>
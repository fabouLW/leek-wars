<template>
	<div class="page">
		<div class="page-header page-bar">
			<h1><router-link to="/admin">Administration</router-link> > Serveurs</h1>
		</div>
		<panel class="first last">
			<loader v-if="loading" />
			<div v-else>
				<div v-if="LeekWars.objectSize(nodes) === 0" class="empty">Aucun serveur</div>
				<div class="servers">
					<div v-for="(node, n) in nodes" :key="n" class="server card">
						<div class="load">
							<div :style="{'margin-top': ((1 - node.load) * 127) + 'px'}"></div>
						</div>
						<img src="/image/admin/server.png">
						<br>
						<div class="name">
							{{ node.name }}<img class="status" src="/image/connected.png">
						</div>
						<div class="total-wrapper">Total : {{ node.generated | number }}</div>
						<div class="threads">
							<div v-for="(runner, r) in node.runners" :key="r" class="thread">
								<div class="th-name">
									<img class="status" src="/image/connected.png">&nbsp;<b>{{ runner.name }}</b>
								</div>
								<span class="green">✔ <span class="generated">{{ runner.generated | number }}</span></span>&nbsp;&nbsp;
								<span v-if="runner.errors > 0" class="red">✘ <span class="error">{{ runner.errors | number }}</span></span>
								<br>
								<div class="task">
									<span v-if="runner.task && runner.task.type === 1">
										<router-link :to="'/fight/' + runner.task.fight">► Combat {{ runner.task.fight }}</router-link>
										<router-link :to="'/farmer/' + runner.task.farmer.id">
											<avatar :farmer="runner.task.farmer" />
										</router-link>
										<router-link :to="'/farmer/' + runner.task.farmer.id">
											{{ runner.task.farmer.name }}
										</router-link>
									</span>
									<span v-else-if="runner.task && runner.task.type === 2">
										<router-link :to="'/tournament/' + runner.task.tournament">► Tournoi {{ runner.task.tournament }}</router-link>
									</span>
								</div>
							</div>
						</div>
					</div>
				</div>
				<div class="queue">
					<h4>➤ Queue ({{ queue.length }})</h4>
					<div class="farmers">
						<div v-for="(task, t) in queue" :key="t" class="card farmer">
							<router-link :to="'/farmer/' + task[1].id">
								<avatar :farmer="task[1]" /> {{ task[1].name }}
							</router-link>
							<div class="fight">
								<router-link v-if="task[0] === 1" :to="'/fight/' + task[2]">{{ task[2] }}</router-link>
								<router-link v-else-if="task[0] === 2" :to="'/tournament/' + task[2]">{{ task[2] }}</router-link>
							</div>
						</div>
					</div>
				</div>
			</div>
		</panel>
	</div>
</template>

<script lang="ts">
	import { LeekWars } from '@/model/leekwars'
	import { Component, Vue } from 'vue-property-decorator'

	const REMOVE_RUNNER = 14
	const LISTEN_DATA = 15
	const UPDATE_RUNNER = 16
	const ADMIN_QUEUE = 36

	class Node {
		public name!: string
		public generated!: number
		public runners!: Runner[]
		public load!: number
	}

	class Runner {
		public id!: number
		public node!: string
		public generated!: number
		public errors!: number
		public name!: string
		public task!: string
	}

	@Component({})
	export default class AdminServers extends Vue {

		runners: {[key: number]: Runner} = {}
		queue: any = []
		loading: boolean = true

		get nodes() {
			const nodes: {[key: string]: Node} = {}
			for (const runner of Object.values(this.runners)) {
				if (!(runner.node in nodes)) {
					Vue.set(nodes, runner.node, { name: runner.node, generated: 0, runners: [], load: 0 } as Node)
				}
				const node = nodes[runner.node]
				node.generated += runner.generated
				node.runners.push(runner)
				node.load = ((node.runners.length - 1) * node.load + (runner.task ? 1 : 0)) / node.runners.length
			}
			return Object.values(nodes).sort((a, b) => a.name.localeCompare(b.name))
		}

		created() {
			LeekWars.socket.send([LISTEN_DATA])
			LeekWars.setTitle("Admin serveurs")
			this.$root.$on('wsmessage', this.update)
		}

		beforeDestroy() {
			this.$root.$off('wsmessage', this.update)
		}

		update(message: any) {
			const type = message.type
			const data = message.data
			if (type === UPDATE_RUNNER) {
				this.loading = false
				for (const th of data) {
					this.updateRunner(th.node, th.id, th.name, th.connected, th.task, th.task_start, th.generated, th.errors)
				}
			}
			if (type === REMOVE_RUNNER) {
				for (const th of data) {
					this.removeRunner(th.id)
				}
			}
			if (type === ADMIN_QUEUE) {
				this.queue = data
			}
		}

		updateRunner(nodeName: string, runnerID: any, runnerName: string, connected: any, task: any, task_start: any, generated: number, errors: number) {
			const runner = this.runners[runnerID]
			if (!runner) {
				const runner = { id: runnerID, name: runnerName, node: nodeName, generated, errors, task } as Runner
				Vue.set(this.runners, runnerID, runner)
			} else {
				runner.name = runnerName
				runner.node = nodeName
				runner.generated = generated
				runner.errors = errors
				runner.task = task
			}
		}

		removeRunner(id: number) {
			if (id in this.runners) {
				Vue.delete(this.runners, id)
			}
		}
	}
</script>

<style lang="scss" scoped>
	.servers {
		display: flex;
		flex-wrap: wrap;
		gap: 15px;
	}
	.server {
		position: relative;
		background: white;
		padding: 10px;
		text-align: center;
		vertical-align: top;
		flex: 1;
	}
	.server .load {
		position: absolute;
		top: 10px;
		left: 10px;
		width: 15px;
		height: 127px;
		background: #eee;
		overflow: hidden;
		border: 2px solid #eee;
	}
	.server .load div {
		height: 100%;
		width: 13px;
		background: #5fad1b;
		transition: margin-top 0.4s ease;
	}
	.servers .name {
		font-size: 22px;
		font-weight: 300;
		margin: 5px;
	}
	.servers .name .status {
		vertical-align: middle;
		margin-left: 8px;
		width: 16px;
	}
	.server .total-wrapper {
		color: #777;
	}
	.threads {
		text-align: left;
		color: #555;
		border-top: 3px solid #f2f2f2;
		padding-top: 8px;
		margin-top: 8px;
	}
	.server .thread {
		color: #aaa;
		font-size: 14px;
		padding-bottom: 10px;
		padding-left: 5px;
		padding-right: 5px;
		display: inline-block;
		width: 50%;
		vertical-align: top;
	}
	.server .thread .th-name {
		color: #666;
		font-size: 14px;
		margin-bottom: 4px;
		text-overflow: ellipsis;
		overflow: hidden;
		white-space: nowrap;
	}
	.server .thread .status {
		width: 16px;
		vertical-align: bottom;
	}
	.server .red {
		color: red;
		font-weight: 500;
	}
	.server .green {
		color: green;
		font-weight: 500;
	}
	h4 {
		margin: 12px 0;
	}
	.queue .farmer {
		display: inline-block;
		width: 80px;
		text-align: center;
		padding: 4px 4px;
		margin-right: 6px;
		margin-bottom: 6px;
		.avatar {
			width: 50px;
		}
		.fight {
			font-size: 13px;
			margin-top: 2px;
			a {
				color: #777;
			}
		}
	}
	.queue .farmers {
		min-height: 60px;
	}
	.empty {
		padding: 20px;
		text-align: center;
		color: #777;
	}
	.task {
		display: flex;
		align-items: center;
		margin: 2px 0;
		height: 20px;
		color: black;
		gap: 5px;
		flex-wrap: nowrap;
		span {
			display: flex;
			align-items: center;
			gap: 5px;
			flex-wrap: nowrap;
			white-space: nowrap;
			text-overflow: ellipsis;
			overflow: hidden;
		}
		.avatar {
			width: 25px;
			flex-basis: 25px 0 0;
			height: 25px;
		}
	}
</style>
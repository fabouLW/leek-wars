<template lang="html">
	<div class="explorer">
		<editor-folder :folder="fileSystem.rootFolder" :level="0" />

		<editor-folder :folder="fileSystem.bin" :level="0" />

		<v-menu offset-y absolute :position-x="x" :position-y="y" :value="aiMenu">
			<div v-if="ai" class="title">{{ ai.name }}</div>
			<v-list class="menu" :dense="true">
				<v-list-item v-ripple @click="open()">
					<v-icon>mdi-card-plus-outline</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('open') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-if="ai && ai.folder !== -1" v-ripple @click="$emit('test')">
					<v-icon>mdi-play</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('test') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-if="ai && ai.folder !== -1" v-ripple @click="renameStart">
					<v-icon>mdi-pencil</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('rename') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-if="ai && ai.folder !== -1" v-ripple @click="deleteDialog = true">
					<v-icon>mdi-delete</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('delete') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-if="ai && ai.folder === -1" v-ripple @click="destroyDialog = true">
					<v-icon>mdi-delete-forever</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('destroy') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-if="ai && ai.folder === -1" v-ripple @click="restoreAI">
					<v-icon>mdi-file-restore</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('restore') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<template v-if="ai && ai.includes && ai.includes.length">
					<v-menu offset-x open-on-hover>
						<template v-slot:activator="{ on, attrs }">
							<v-list-item v-ripple v-bind="attrs" v-on="on">
								<v-icon>mdi-download</v-icon>
								<v-list-item-content>
									<v-list-item-title>{{ $t('download') }}</v-list-item-title>
								</v-list-item-content>
								<v-list-item-icon>
									<v-icon>mdi-menu-right</v-icon>
								</v-list-item-icon>
							</v-list-item>
						</template>
						<v-list class="menu" :dense="true">
							<v-list-item v-ripple @click="downloadSimple()">
								<v-icon>mdi-file-outline</v-icon>
								<v-list-item-content>
									<v-list-item-title>{{ $t('download_simple') }}</v-list-item-title>
								</v-list-item-content>
							</v-list-item>
							<v-list-item v-ripple @click="downloadIncludes()">
								<v-icon>mdi-file-multiple-outline</v-icon>
								<v-list-item-content>
									<v-list-item-title>{{ $t('download_includes') }}</v-list-item-title>
								</v-list-item-content>
							</v-list-item>
						</v-list>
					</v-menu>
				</template>
				<v-list-item v-else v-ripple @click="downloadSimple()">
					<v-icon>mdi-download</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('download') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
			</v-list>
		</v-menu>

		<v-menu offset-y absolute :position-x="x" :position-y="y" :value="folderMenu">
			<div v-if="folder && folder.id !== 0" class="title">{{ folder.name }}</div>
			<v-list class="menu" :dense="true">
				<v-list-item v-ripple @click="newAIStart()">
					<v-icon>mdi-file-plus-outline</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('new_ai') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-ripple @click="newFolderStart()">
					<v-icon>mdi-folder-plus-outline</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('new_folder') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-ripple @click="renameStart">
					<v-icon>mdi-pencil</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('rename') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
				<v-list-item v-ripple @click="deleteDialog = true">
					<v-icon>mdi-delete</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('delete') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
			</v-list>
		</v-menu>

		<v-menu offset-y absolute :position-x="x" :position-y="y" :value="binMenu">
			<div v-if="folder" class="title">{{ $t(folder.name) }}</div>
			<v-list class="menu" :dense="true">
				<v-list-item v-ripple @click="emptyDialog = true">
					<v-icon>mdi-delete-forever</v-icon>
					<v-list-item-content>
						<v-list-item-title>{{ $t('empty_bin') }}</v-list-item-title>
					</v-list-item-content>
				</v-list-item>
			</v-list>
		</v-menu>

		<popup v-model="renameDialog" :width="500">
			<v-icon slot="icon">mdi-pencil</v-icon>
			<span slot="title">{{ $t('rename') }}</span>
			<div class="padding">
				<input ref="nameInput" v-model="newName" type="text" class="input dialog-input" @keyup.stop @keyup.enter="rename()">
			</div>
			<div slot="actions">
				<div v-ripple @click="renameDialog = false">{{ $t('main.cancel') }}</div>
				<div v-ripple class="green" @click="rename()">{{ $t('rename') }}</div>
			</div>
		</popup>

		<popup v-model="deleteDialog" :width="500">
			<v-icon slot="icon">mdi-delete</v-icon>
			<span v-if="ai" slot="title">{{ $t('delete_ai', [ai.name]) }}</span>
			<span v-else-if="folder" slot="title">{{ $t('delete_folder', [folder.name]) }}</span>
			{{ $t('delete_warning') }}
			<div slot="actions">
				<div v-ripple @click="deleteDialog = false">{{ $t('delete_cancel') }}</div>
				<div v-ripple class="red" @click="deleteItem">{{ $t('delete_validate') }}</div>
			</div>
		</popup>

		<popup v-model="destroyDialog" :width="500">
			<v-icon slot="icon">mdi-delete-forever</v-icon>
			<span v-if="ai" slot="title">{{ $t('destroy_ai', [ai.name]) }}</span>
			{{ $t('destroy_warning') }}
			<div slot="actions">
				<div v-ripple @click="destroyDialog = false">{{ $t('delete_cancel') }}</div>
				<div v-ripple class="red" @click="destroyAI">{{ $t('destroy_validate') }}</div>
			</div>
		</popup>

		<popup v-model="emptyDialog" :width="500">
			<v-icon slot="icon">mdi-delete</v-icon>
			<span slot="title">{{ $t('empty_bin') }}</span>
			{{ $t('empty_warning') }}
			<div slot="actions">
				<div v-ripple @click="emptyDialog = false">{{ $t('delete_cancel') }}</div>
				<div v-ripple class="red" @click="emptyBin">{{ $t('empty_bin') }}</div>
			</div>
		</popup>

		<popup v-model="newAIDialog" :width="500">
			<v-icon slot="icon">mdi-plus-circle-outline</v-icon>
			<span slot="title">{{ $t('new_desc') }}</span>
			<div class="padding">
				<input ref="newAIInput" v-model="newAIName" :placeholder="$t('ai_name')" type="text" class="input dialog-input" @keyup.stop @keyup.enter="newAI(false, newAIName)">
			</div>
			<div slot="actions">
				<div v-ripple @click="newAIDialog = false">{{ $t('main.cancel') }}</div>
				<div v-ripple class="green" @click="newAI(false, newAIName)">{{ $t('main.create') }}</div>
			</div>
		</popup>

		<popup v-model="newFolderDialog" :width="500">
			<v-icon slot="icon">mdi-folder-plus</v-icon>
			<span slot="title">{{ $t('new_folder') }}</span>
			<div class="padding">
				<input ref="newFolderInput" v-model="newFolderName" :placeholder="$t('folder_name')" type="text" class="input dialog-input" @keyup.stop @keyup.enter="newFolder(newFolderName)">
			</div>
			<div slot="actions">
				<div v-ripple @click="newFolderDialog = false">{{ $t('main.cancel') }}</div>
				<div v-ripple class="green" @click="newFolder(newFolderName)">{{ $t('main.create') }}</div>
			</div>
		</popup>
	</div>
</template>

<script lang="ts">
	import { AI } from '@/model/ai'
	import { fileSystem } from '@/model/filesystem'
	import { i18n, mixins } from '@/model/i18n'
	import { LeekWars } from '@/model/leekwars'
	import { Component, Prop, Vue, Watch } from 'vue-property-decorator'
	import EditorFolder from './editor-folder.vue'
	import { Folder } from './editor-item'

	@Component({ name: 'editor-explorer', i18n: {}, mixins: [...mixins], components: { 'editor-folder': EditorFolder } })
	export default class Explorer extends Vue {
		@Prop({required: true}) currentAi!: AI
		@Prop({required: true}) selectedFolder!: Folder
		fileSystem = fileSystem
		aiMenu: boolean = false
		folderMenu: boolean = false
		binMenu: boolean = false
		x: number = 0
		y: number = 0
		ai: AI | null = null
		folder: Folder | null = null
		renameDialog: boolean = false
		newName: string = ''
		deleteDialog: boolean = false
		destroyDialog: boolean = false
		emptyDialog: boolean = false
		newAIName: string = ''
		newAIDialog: boolean = false
		newFolderDialog: boolean = false
		newFolderName: string = ''

		created() {
			this.$root.$on('editor-menu', this.openMenu)
			this.$root.$on('keyup', this.keyup)
		}
		destroyed() {
			this.$root.$off('editor-menu', this.openMenu)
			this.$root.$off('keyup', this.keyup)
		}

		open() {
			this.$router.push('/editor/' + this.ai!.id)
		}

		openMenu(item: AI | Folder, ai: boolean, e: any) {
			e.preventDefault()
			this.aiMenu = this.folderMenu = this.binMenu = false
			this.x = e.clientX
			this.y = e.clientY
			if (ai) {
				this.ai = item as AI
				this.folder = null
				this.$nextTick(() => this.aiMenu = true)
			} else {
				this.folder = item as Folder
				this.ai = null
				// console.log("folder", this.folder)
				if (this.folder.id === -1) {
					this.$nextTick(() => this.binMenu = true)
				} else {
					this.$nextTick(() => this.folderMenu = true)
				}
			}
		}

		renameStart() {
			if (this.ai) {
				this.newName = this.ai!.name
			} else {
				this.newName = this.folder!.name
			}
			this.renameDialog = true
			setTimeout(() => (this.$refs.nameInput as HTMLElement).focus(), 50)
		}

		rename() {
			if (this.ai) {
				if (this.newName !== this.ai.name) {
					LeekWars.post('ai/rename', {ai_id: this.ai.id, new_name: this.newName}).then(data => {
						LeekWars.toast(i18n.t('leekscript.ai_renamed', [this.newName]) as string)
						fileSystem.renameAI(this.ai!, this.newName)
					}).error(error => {
						LeekWars.toast(error.error)
					})
				}
			} else if (this.folder) {
				if (this.newName !== this.folder.name) {
					LeekWars.post('ai-folder/rename', {folder_id: this.folder.id, new_name: this.newName}).then(data => {
						LeekWars.toast(i18n.t('leekscript.folder_renamed', [this.newName]) as string)
						this.folder!.name = this.newName
					}).error(error => {
						LeekWars.toast(error.error)
					})
				}
			}
			this.renameDialog = false
		}

		deleteItem() {
			if (this.ai) {
				fileSystem.deleteAI(this.ai)
				this.$emit('delete-ai', this.ai)
				this.deleteDialog = false
			} else if (this.folder) {
				fileSystem.deleteFolder(this.folder)
				this.$emit('delete-folder', this.folder)
				this.deleteDialog = false
			}
		}
		deleteAI(ai: AI) {
			this.ai = ai
			this.deleteDialog = true
		}

		destroyAI() {
			if (this.ai) {
				fileSystem.destroyAI(this.ai)
				// this.$emit('delete-ai', this.ai)
				this.destroyDialog = false
			}
		}

		emptyBin() {
			fileSystem.emptyBin()
			this.emptyDialog = false
		}

		restoreAI() {
			if (this.ai) {
				fileSystem.restore(this.ai)
			}
		}

		openNewAI(folder: Folder) {
			this.folder = folder
			this.newAIStart()
		}
		newAIStart() {
			this.newAIDialog = true
			setTimeout(() => (this.$refs.newAIInput as HTMLElement).focus(), 50)
		}
		newAI(v2: boolean, name: string) {
			if (!this.folder) { return }
			LeekWars.post('ai/new-name', {folder_id: this.folder.id, version: LeekWars.LATEST_LEEKSCRIPT_VERSION, name}).then(data => {
				const ai = new AI(data.ai)
				ai.valid = true
				ai.version = LeekWars.LATEST_LEEKSCRIPT_VERSION
				ai.total_chars = ai.code.length
				ai.total_lines = ai.code.split("\n").length
				fileSystem.add_ai(ai, this.folder!)
				this.folder!.expanded = true
				this.$router.push('/editor/' + ai.id)
				this.newAIDialog = false
				this.newAIName = ''
			})
		}
		openNewFolder(folder: Folder) {
			this.folder = folder
			this.newFolderStart()
		}
		newFolderStart() {
			this.newFolderDialog = true
			setTimeout(() => (this.$refs.newFolderInput as HTMLElement).focus(), 50)
		}
		newFolder(name: string) {
			if (!this.folder) { return }
			LeekWars.post('ai-folder/new-name', {folder_id: this.folder.id, name}).then(data => {
				if (this.folder) {
					const folder = new Folder(data.id, name, this.folder.id)
					folder.items = []
					fileSystem.add_folder(folder, this.folder)
					this.folder!.expanded = true
					this.$router.push('/editor/' + folder.id)
					this.newFolderDialog = false
					this.newFolderName = ''
				}
			})
		}

		keyup(e: KeyboardEvent) {
			if (e.which === 46) { // Suppr
				if (this.currentAi) {
					this.ai = this.currentAi
					this.deleteDialog = true
				} else if (this.selectedFolder) {
					this.folder = this.selectedFolder
					this.deleteDialog = true
				}
			}
		}

		downloadSimple() {
			this.download(this.ai!.name, "/** " + this.ai!.path + " **/\n\n" + this.ai!.code)
		}

		downloadIncludes() {
			if (!this.ai) { return }

			const regex = /include\s*\(\s*["'](.*?)["']\s*\)\s*;?/gm
			const included_ais = new Set<AI>()
			const fun = (ai: AI): string => "/** " + ai.path + " **/\n\n" + (ai.code ? ai.code.replace(regex, (a, path) => {
				const included = fileSystem.find(path, ai.folder)
				if (included && !included_ais.has(included)) {
					included_ais.add(included)
					return fun(included)
				} else {
					return ""
				}
			}) : '')
			const code = fun(this.ai!)

			this.download(this.ai!.name, code)
		}

		download(filename: string, text: string) {
			const data = "/** Exporté le " + new Date().toLocaleString() + " **/\n\n" + text
			if (!filename.includes(".")) {
				filename += ".leek"
			}
			const element = document.createElement('a')
			element.setAttribute('href', 'data:text/plain;charset=utf-8,' + encodeURIComponent(data))
			element.setAttribute('download', filename)
			element.style.display = 'none'
			document.body.appendChild(element)
			element.click()
			document.body.removeChild(element)
		}
	}
</script>

<style lang="scss" scoped>
.explorer {
	height: 100%;
	display: flex;
	flex-direction: column;
}
.title {
	padding: 5px 10px;
	padding-top: 10px;
	color: #777;
	font-size: 13px;
}
.v-icon {
	margin-right: 8px;
}
.dialog-input {
	width: 100%;
	padding: 10px;
}
</style>
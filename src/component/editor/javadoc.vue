<template lang="html">
	<div>
		<h2>
			<span v-if="keyword.type === 'user-static-method' || keyword.type === 'user-static-field'">{{ keyword.clazz.name }}.</span>{{ javadoc.name }} <span v-if="return_ && return_.name" class="arrow">→</span> <span v-if="return_ && return_.name">{{ return_.name }}</span>
		</h2>
		<div class="description" v-html="javadoc.description"></div>

		<template v-if="args.length > 0">
			<h4>{{ $t('doc.parameters') }}</h4>
			<ul>
				<li v-for="(arg, i) in args" :key="i">{{ arg.name }} <span v-if="arg.name && arg.text">:</span> <span v-if="arg.text" v-dochash v-code v-html="arg.text"></span></li>
			</ul>
		</template>

		<h4 v-if="return_">{{ $t('doc.return') }}</h4>
		<ul v-if="return_">
			<li>{{ return_.name }} <span v-if="return_.name">:</span> <span v-dochash v-code v-html="return_.text"></span>
			</li>
		</ul>

		<div v-for="(item, i) in other" :key="i">
			<b>{{ item.type }}</b> {{ item.name }} <span v-if="item.name">:</span> <span v-dochash v-code v-html="item.text"></span>
		</div>
	</div>
</template>

<script lang="ts">
	import { Component, Prop, Vue } from 'vue-property-decorator'

	@Component({ name: "javadoc" })
	export default class Javadoc extends Vue {
		@Prop() javadoc!: any
		@Prop() keyword!: any

		get args() {
			return this.javadoc.items.filter((i: any) => i.type === 'param')
		}
		get return_() {
			const ret = this.javadoc.items.filter((i: any) => i.type === 'return')
			if (ret.length) { return ret[0] }
			return null
		}
		get other() {
			return this.javadoc.items.filter((i: any) => i.type !== 'param' && i.type !== 'return')
		}
	}
</script>

<style lang="scss" scoped>
	.description {
		white-space: pre-wrap;
		color: #444;
	}
	h2 {
		margin-bottom: 12px;
		font-size: 17px;
		color: #111;
		font-family: monospace;
	}
	h4 {
		font-weight: 500;
		margin: 0;
		color: #111;
		margin-top: 10px;
		margin-bottom: 8px;
		font-size: 15px;
	}
	::v-deep a {
		color: #5fad1b;
		font-weight: 500;
		&:hover {
			text-decoration: underline;
		}
	}
	::v-deep ul {
		margin: 5px 0;
	}
	.arrow {
		font-size: 30px;
		line-height: 17px;
		vertical-align: top;
	}
</style>
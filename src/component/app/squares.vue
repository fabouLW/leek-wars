<template>
	<div v-show="LeekWars.squares.squares.length" class="squares">
		<router-link v-for="square in LeekWars.squares.squares" :key="square.id" v-ripple :to="square.link" class="square card" :class="{[square.clazz]: square.clazz}" @click.native="LeekWars.closeMenu()">
			<v-icon v-if="square.icon" :class="{padding: square.padding}" class="image">{{ square.image }}</v-icon>
			<img v-else :src="square.image" :class="{padding: square.padding}" class="image">
			<div class="wrapper">
				<div class="title" v-html="square.title"></div>
				<div v-emojis class="message" v-html="square.message"></div>
			</div>
			<span v-if="square.resultIcon && LeekWars.notifsResults" class="result">
				<v-icon :class="square.resultIcon">{{ square.resultIcon }}</v-icon>
			</span>
		</router-link>
	</div>
</template>

<script lang="ts">
	import { Component, Vue } from 'vue-property-decorator'
	import '@/model/emojis'

	@Component({})
	export default class Squares extends Vue {}
</script>

<style lang="scss" scoped>
	.squares {
		position: fixed;
		bottom: 0;
		right: 0;
		z-index: 10000;
		padding: 20px;
	}
	.square {
		display: flex;
		margin-top: 10px;
		width: 300px;
		border-radius: 2px;
		box-shadow: 0px 11px 15px -7px rgba(0,0,0,0.2), 0px 24px 38px 3px rgba(0,0,0,0.14), 0px 9px 46px 8px rgba(0,0,0,0.12);
		overflow: hidden;
		animation: in-and-out 5s;
		img.padding {
			opacity: 0.7;
		}
		i {
			font-size: 32px;
			color: #444;
		}
	}
	@keyframes in-and-out {
		0% { transform: translate(150%, 0); }
		10% { transform: translate(0, 0); }
		90% { transform: translate(0, 0); }
		100% { transform: translate(150%, 0); }
	}
	.image {
		width: 60px;
		height: 60px;
		flex: 0 0 60px;
		margin-right: 8px;
	}
	.image.padding {
		width: 60px;
		height: 60px;
		flex: 0 0 60px;
		margin-right: 0;
		padding: 12px;
	}
	.wrapper {
		padding: 10px 0;
	}
	.title {
		font-size: 15px;
		margin-bottom: 4px;
	}
	.message {
		font-size: 14px;
	}
	.result {
		position: absolute;
		background: white;
		height: 24px;
		width: 24px;
		border-radius: 50%;
		text-align: center;
		border-bottom: 2px solid #ccc;
		border-right: 2px solid #ccc;
		left: 2px;
		top: 2px;
		i {
			font-size: 20px;
			padding-top: 2px;
			padding-left: 2px;
			font-weight: bold;
		}
	}
	.result .mdi-check {
		color: green;
	}
	.result .mdi-close {
		color: red;
	}
</style>
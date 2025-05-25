<script setup>
import { computed, ref } from 'vue'

const inputValue = ref('0')
const isResult = ref(false)
const previousAct = ref('')
const lastOperation = ref('')
const numbers = ['1', '2', '3', '4', '5', '6', '7', '8', '9', 'f', '0', '.']

const deleteIcon = computed(
	() => isResult.value || inputValue.value < 3 || inputValue.value == 'Error'
)

function AppendToValue(value) {
	const isOperator = /[+\-*/]/.test(value)
	const lastChar = inputValue.value.slice(-1)

	if (value === 'f') {
		isResult.value = true
		alert('This function will be added later')
		return
	}

	if (isResult.value) {
		if (/\d/.test(value)) {
			inputValue.value = value === '.' ? '0.' : value
		} else {
			inputValue.value += value
		}
		isResult.value = false
	} else {
		if (
			(inputValue.value === '0' && value !== '.') ||
			inputValue.value == 'Error'
		) {
			inputValue.value = value
		} else if (isOperator && /[+\-*/]/.test(lastChar)) {
			inputValue.value = inputValue.value.slice(0, -1) + value
		} else {
			inputValue.value += value
		}
	}
}

function clearInput() {
	if (isResult.value || inputValue.value < 3 || inputValue.value == 'Error') {
		inputValue.value = '0'
		previousAct.value = ''
		lastOperation.value = ''
	} else {
		inputValue.value = inputValue.value.slice(0, -1)
	}
	isResult.value = false
}

function toggleSign() {
	if (!inputValue.value) return
	const num = eval(inputValue.value)
		.toFixed(4)
		.replace(/\.?0+$/, '')
	inputValue.value = (-num).toString()
	isResult.value = true
}

function percent() {
	let num = parseFloat(inputValue.value)
	if (!isNaN(num)) {
		inputValue.value = (num / 100).toString()
		isResult.value = true
	}
}

function calculateResult() {
	try {
		const isPureNumber = /^\d+(\.\d+)?$/.test(inputValue.value)
		if (isPureNumber) {
			previousAct.value = ''
			return
		}
		if (isResult.value && lastOperation.value) {
			inputValue.value += lastOperation.value
			previousAct.value = inputValue.value + '='
		} else {
			const match = inputValue.value.match(/([+\-*/])(\d+\.?\d*)$/)
			lastOperation.value = match ? match[0] : ''
			previousAct.value = inputValue.value + '='
		}
		const result = eval(inputValue.value)
		inputValue.value = Number(result)
			.toFixed(4)
			.replace(/\.?0+$/, '')
		isResult.value = true
	} catch {
		inputValue.value = 'Error'
		isResult.value = false
		lastOperation.value = ''
		previousAct.value = ''
	}
}
</script>

<template>
	<div class="calculator">
		<div class="container">
			<div class="input-container">
				<div class="memory">
					<svg
						viewBox="0 0 24 24"
						fill="none"
						width="40"
						height="40"
						xmlns="http://www.w3.org/2000/svg"
						stroke="#000000"
					>
						<g id="SVGRepo_bgCarrier" stroke-width="0"></g>
						<g
							id="SVGRepo_tracerCarrier"
							stroke-linecap="round"
							stroke-linejoin="round"
						></g>
						<g id="SVGRepo_iconCarrier">
							<path
								d="M4 18L20 18"
								stroke="#ff9f0a"
								stroke-width="2"
								stroke-linecap="round"
							></path>
							<path
								d="M4 12L20 12"
								stroke="#ff9f0a"
								stroke-width="2"
								stroke-linecap="round"
							></path>
							<path
								d="M4 6L20 6"
								stroke="#ff9f0a"
								stroke-width="2"
								stroke-linecap="round"
							></path>
						</g>
					</svg>
				</div>
				<div class="previousAct">{{ previousAct }}</div>
				<input type="text" class="input" :value="inputValue" readonly />
			</div>
			<div class="buttons">
				<div class="left-but">
					<div class="gray-buttons">
						<button class="gray" @click="clearInput">
							<span v-if="deleteIcon">AC</span>
							<span v-else>←</span>
						</button>
						<button class="gray" @click="toggleSign">+/-</button>
						<button class="gray" @click="percent">%</button>
					</div>
					<div class="num-buttons">
						<button
							class="number"
							v-for="btn in numbers"
							:key="btn"
							@click="AppendToValue(btn)"
						>
							{{ btn }}
						</button>
					</div>
				</div>

				<div class="orange-buttons">
					<button class="orange" @click="AppendToValue('/')">/</button>
					<button class="orange" @click="AppendToValue('*')">*</button>
					<button class="orange" @click="AppendToValue('-')">-</button>
					<button class="orange" @click="AppendToValue('+')">+</button>
					<button class="orange" @click="calculateResult">=</button>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
@import url(./home.css);
</style>

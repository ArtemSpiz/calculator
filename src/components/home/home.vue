<script setup>
import { computed, ref, watch, onMounted } from 'vue'

const inputValue = ref('0')
const evalExpression = ref('0')
const isResult = ref(false)
const previousAct = ref('')
const lastOperation = ref('')
const showComplexButtons = ref(false)
const waitingForPower = ref(false)
const waitingForLog = ref(false)
const tempValue = ref('')
const tempEval = ref('')

const numbers = [
	{ type: 'text', value: '1' },
	{ type: 'text', value: '2' },
	{ type: 'text', value: '3' },
	{ type: 'text', value: '4' },
	{ type: 'text', value: '5' },
	{ type: 'text', value: '6' },
	{ type: 'text', value: '7' },
	{ type: 'text', value: '8' },
	{ type: 'text', value: '9' },
	{
		type: 'svg',
		value: `<svg xmlns="http://www.w3.org/2000/svg" width="35" height="35" viewBox="0 0 50 50">
<path d="M 14 4 C 8.4886661 4 4 8.4886661 4 14 L 4 36 C 4 41.511334 8.4886661 46 14 46 L 36 46 C 41.511334 46 46 41.511334 46 36 L 46 14 C 46 8.4886661 41.511334 4 36 4 L 14 4 z M 14 6 L 36 6 C 40.430666 6 44 9.5693339 44 14 L 44 36 C 44 40.430666 40.430666 44 36 44 L 14 44 C 9.5693339 44 6 40.430666 6 36 L 6 14 C 6 9.5693339 9.5693339 6 14 6 z M 18 9 C 15.250484 9 13 11.250484 13 14 L 13 36 C 13 38.749516 15.250484 41 18 41 L 32 41 C 34.749516 41 37 38.749516 37 36 L 37 14 C 37 11.250484 34.749516 9 32 9 L 18 9 z M 18 11 L 32 11 C 33.668484 11 35 12.331516 35 14 L 35 36 C 35 37.668484 33.668484 39 32 39 L 18 39 C 16.331516 39 15 37.668484 15 36 L 15 14 C 15 12.331516 16.331516 11 18 11 z M 18 13 C 17.448 13 17 13.448 17 14 L 17 18 C 17 18.552 17.448 19 18 19 L 32 19 C 32.552 19 33 18.552 33 18 L 33 14 C 33 13.448 32.552 13 32 13 L 18 13 z M 19 21 A 2 2 0 0 0 19 25 A 2 2 0 0 0 19 21 z M 25 21 A 2 2 0 0 0 25 25 A 2 2 0 0 0 25 21 z M 31 21 A 2 2 0 0 0 31 25 A 2 2 0 0 0 31 21 z M 19 27 A 2 2 0 0 0 19 31 A 2 2 0 0 0 19 27 z M 25 27 A 2 2 0 0 0 25 31 A 2 2 0 0 0 25 27 z M 31 27 A 2 2 0 0 0 31 31 A 2 2 0 0 0 31 27 z M 19 33 C 17.895 33 17 33.895 17 35 C 17 36.105 17.895 37 19 37 L 25 37 C 26.105 37 27 36.105 27 35 C 27 33.895 26.105 33 25 33 L 19 33 z M 31 33 A 2 2 0 0 0 31 37 A 2 2 0 0 0 31 33 z"></path>
</svg>`,
	},
	{ type: 'text', value: '0' },
	{ type: 'text', value: '.' },
]
const complexAct = [
	'(',
	')',
	'**2',
	'**3',
	'**n',
	'√',
	'!',
	'log',
	'sin',
	'cos',
	'tan',
	'π',
]
const historyInput = ref([])
const historyResult = ref([])
const showHistory = ref(false)
const stateMap = {
	inputValue,
	previousAct,
	lastOperation,
	isResult,
	historyInput,
	historyResult,
	showHistory,
	showComplexButtons,
}

const deleteIcon = computed(
	() =>
		isResult.value ||
		inputValue.value.length <= 3 ||
		inputValue.value == 'Error'
)

function closeHistory() {
	showHistory.value = false
}

function factorial(n) {
	if (n < 0) return Error
	if (n == 0 || n == 1) return 1
	let result = 1
	for (let i = 2; i <= n; i++) result *= i
	return result
}

function AppendToValue(value) {
	const isOperator = /[+\-*/]/.test(value)
	const lastChar = inputValue.value.slice(-1)

	if (
		inputValue.value === 'Error' &&
		[
			'sin',
			'cos',
			'tan',
			'√',
			'log',
			'**2',
			'**3',
			'**n',
			'(',
			')',
			'π',
			'!',
		].includes(value)
	) {
		return
	}

	if (value === 'f') {
		showComplexButtons.value = !showComplexButtons.value
		return
	}

	if (value === '**2') {
		inputValue.value = `(${inputValue.value})²`
		evalExpression.value = `(${evalExpression.value})**2`
		return
	}

	if (value === '**3') {
		inputValue.value = `(${inputValue.value})³`
		evalExpression.value = `(${evalExpression.value})**3`
		return
	}

	if (value === '**n') {
		tempValue.value = inputValue.value
		tempEval.value = evalExpression.value
		waitingForPower.value = true
		evalExpression.value = ''
		return
	}
	if (waitingForPower.value) {
		inputValue.value = `(${tempValue.value})^(${value})`
		evalExpression.value = `(${tempEval.value})**(${value})`
		waitingForPower.value = false
		return
	}

	if (value === '√') {
		inputValue.value = `√(${inputValue.value})`
		evalExpression.value = `Math.sqrt(${evalExpression.value})`
		return
	}

	if (value === '!') {
		inputValue.value = `(${inputValue.value})!`
		evalExpression.value = `factorial(${evalExpression.value})`
		return
	}

	if (value === 'log') {
		tempValue.value = inputValue.value
		tempEval.value = evalExpression.value
		waitingForLog.value = true
		evalExpression.value = ''
		return
	}

	if (waitingForLog.value && value !== 'log') {
		inputValue.value = `log₍${tempValue.value}₎(${value})`
		evalExpression.value = `(Math.log(${value}) / Math.log(${tempEval.value}))`
		waitingForLog.value = false
		return
	}

	if (value === 'sin') {
		inputValue.value = `sin(${inputValue.value})`
		evalExpression.value = `Math.sin(${evalExpression.value})`

		return
	}
	if (value === 'cos') {
		inputValue.value = `cos(${inputValue.value})`
		evalExpression.value = `Math.cos(${evalExpression.value})`
		return
	}
	if (value === 'tan') {
		inputValue.value = `tan(${inputValue.value})`
		evalExpression.value = `Math.tan(${evalExpression.value})`
		return
	}

	if (value === 'π') {
		if (inputValue.value === '0' || inputValue.value === 'Error') {
			inputValue.value = 'π'
			evalExpression.value = 'Math.PI'
		} else {
			inputValue.value += 'π'
			evalExpression.value += 'Math.PI'
		}
		return
	}

	if (isResult.value) {
		if (/\d/.test(value)) {
			inputValue.value = value === '.' ? '0.' : value
			evalExpression.value = value === '.' ? '0.' : value
			previousAct.value = ''
		} else {
			inputValue.value += value
			evalExpression.value += value
		}
		isResult.value = false
	} else {
		if (
			(inputValue.value === '0' && value !== '.') ||
			inputValue.value == 'Error'
		) {
			inputValue.value = value
			evalExpression.value = value
		} else if (isOperator && /[+\-*/]/.test(lastChar)) {
			inputValue.value = inputValue.value.slice(0, -1) + value
			evalExpression.value = evalExpression.value.slice(0, -1) + value
		} else {
			inputValue.value += value
			evalExpression.value += value
		}
	}
}

function clearHistory() {
	historyInput.value = []
	historyResult.value = []
	localStorage.removeItem('calc_historyInput')
	localStorage.removeItem('calc_historyResult')
}

function deleteEntry(index) {
	historyInput.value.splice(index, 1)
	historyResult.value.splice(index, 1)
}

function clearInput() {
	if (
		isResult.value ||
		inputValue.value.length <= 3 ||
		inputValue.value == 'Error'
	) {
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
	const lastChar = evalExpression.value.trim().slice(-1)
	if (/[+\-*/]/.test(lastChar)) {
		return
	}

	try {
		if (isResult.value && lastOperation.value) {
			evalExpression.value += lastOperation.value
			previousAct.value = inputValue.value + '='
		} else if (
			!/([+\-*/()√^logsincosπ])/.test(evalExpression.value) &&
			evalExpression.value.trim() !== 'Math.PI'
		) {
			return
		} else {
			const match = inputValue.value.match(/([+\-*/])(\d+\.?\d*)$/)
			lastOperation.value = match ? match[0] : ''
			previousAct.value = inputValue.value + '='
		}

		const result = eval(evalExpression.value)
		const formattedResult = Number(result)
			.toFixed(4)
			.replace(/\.?0+$/, '')

		inputValue.value = formattedResult
		evalExpression.value = formattedResult
		isResult.value = true

		historyInput.value.push(`${previousAct.value}`)
		historyResult.value.push(`${formattedResult}`)
	} catch {
		inputValue.value = 'Error'
		evalExpression.value = ''
		isResult.value = false
		lastOperation.value = ''
	}
}

for (const [key, refVar] of Object.entries(stateMap)) {
	watch(
		refVar,
		newVal => {
			localStorage.setItem(`calc_${key}`, JSON.stringify(newVal))
		},
		{ deep: true }
	)
}

onMounted(() => {
	for (const [key, refVar] of Object.entries(stateMap)) {
		const saved = localStorage.getItem(`calc_${key}`)
		if (saved !== null) {
			refVar.value = JSON.parse(saved)
		}
	}
})
</script>

<template>
	<div class="calculator" :class="{ 'show-mode': showComplexButtons }">
		<div class="container">
			<div class="input-container">
				<div class="memory" @click="showHistory = !showHistory">
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

			<div :class="{ show: showHistory }" class="history-panel">
				<div class="history-scroll">
					<div class="history-buttons">
						<button class="clear-history" @click="clearHistory">
							Очистити
						</button>
						<button class="close-button" @click="closeHistory">Закрити</button>
					</div>
					<div
						v-for="(input, index) in historyInput"
						:key="index"
						class="history-entry"
					>
						<div class="history-entry-text">
							<div class="history-input">{{ input }}</div>
							<div class="history-result">{{ historyResult[index] }}</div>
						</div>

						<div class="history-entry-delete" @click="deleteEntry(index)">
							<svg
								viewBox="0 0 24 24"
								fill="none"
								xmlns="http://www.w3.org/2000/svg"
								width="24"
								height="24"
							>
								<g id="SVGRepo_bgCarrier" stroke-width="0"></g>
								<g
									id="SVGRepo_tracerCarrier"
									stroke-linecap="round"
									stroke-linejoin="round"
								></g>
								<g id="SVGRepo_iconCarrier">
									<path
										d="M16 8L8 16M8 8L16 16"
										stroke="#ff0000"
										stroke-width="2"
										stroke-linecap="round"
									></path>
								</g>
							</svg>
						</div>
					</div>
				</div>
			</div>
			<div class="buttons">
				<div class="darkGray-buttons" v-if="showComplexButtons">
					<button
						class="complexAct"
						v-for="btn in complexAct"
						:key="btn"
						@click="AppendToValue(btn)"
					>
						{{ btn }}
					</button>
				</div>
				<div class="activeComplexButtons">
					<div class="left-but">
						<div
							class="gray-buttons"
							:class="{ 'active-ComBut': showComplexButtons }"
						>
							<button class="gray" @click="clearInput">
								<span v-if="deleteIcon">AC</span>
								<span v-else>←</span>
							</button>
							<button class="gray" @click="toggleSign">+/-</button>
							<button class="gray" @click="percent">%</button>
						</div>
						<div
							:class="{ ' active-ComBut': showComplexButtons }"
							class="num-buttons"
						>
							<button
								class="number"
								v-for="btn in numbers"
								:key="btn.value"
								@click="AppendToValue(btn.type === 'text' ? btn.value : 'f')"
								v-html="btn.type === 'svg' ? btn.value : btn.value"
							></button>
						</div>
					</div>

					<div
						class="orange-buttons"
						:class="{ ' orangeActive-ComBut': showComplexButtons }"
					>
						<button class="orange" @click="AppendToValue('/')">/</button>
						<button class="orange" @click="AppendToValue('*')">*</button>
						<button class="orange" @click="AppendToValue('-')">-</button>
						<button class="orange" @click="AppendToValue('+')">+</button>
						<button class="orange" @click="calculateResult">=</button>
					</div>
				</div>
			</div>
		</div>
	</div>
</template>

<style>
@import url(./home.css);
</style>

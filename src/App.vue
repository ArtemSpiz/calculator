<script setup>
import Home from '@/pages/Home.vue'
import { onMounted } from 'vue'

onMounted(() => {
	document.addEventListener(
		'touchstart',
		event => {
			if (event.touches.length > 1) {
				event.preventDefault()
			}
		},
		{ passive: false }
	)

	let lastTouchEnd = 0
	document.addEventListener(
		'touchend',
		event => {
			const now = Date.now()
			if (now - lastTouchEnd <= 300) {
				event.preventDefault()
			}
			lastTouchEnd = now
		},
		{ passive: false }
	)
})
</script>

<template>
	<Home></Home>
</template>

<style scoped></style>

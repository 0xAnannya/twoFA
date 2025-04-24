<script lang="ts">
	// Bindable props from parent
	let {
		error = $bindable<boolean>(false),
		isSuccess = $bindable<boolean>(false),
		buttonMsg = $bindable()
	} = $props();

</script>

<button
	class="text-md relative mt-4 h-13 w-full rounded-xl bg-gray-200 py-3 font-normal text-gray-500"
	class:animate-shake={error}
	class:success={isSuccess}
>
	{buttonMsg}
</button>

<style lang="postcss">
	@keyframes shake {
		0%,
		100% {
			transform: translateX(0);
		}
		25% {
			transform: translateX(-10px);
		}
		75% {
			transform: translateX(10px);
		}
	}
	.animate-shake {
		animation: shake 0.2s ease-in-out both;
		background-color: #fd4b63;
		color: white;
	}
	.success {
		background-color: #4193f7;
		transform-origin: right; /* ← anchor scaling at the right edge */
		transform: scaleX(0); /* start “squished” to zero width */
		animation: expand-rtl 0.3s linear forwards;
		color: white;
	}

	@keyframes expand-rtl {
		to {
			transform: scaleX(1);
		} /* grow back out to full width to the left */
	}
</style>

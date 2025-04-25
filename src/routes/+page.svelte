<script lang="ts">
	import OtPbtn from '../components/OTPbtn.svelte';
	import { correctCode, VALID_KEYWORDS } from '../utils/constants';
	import defaultLock from '../icons/defaultLock.svg';
	import errorLock from '../icons/errorLock.svg';
	import successLock from '../icons/successLock.svg';

	const inputs = $state<string[]>(['', '', '', '', '', '']);
	let error = $state<boolean>(false);
	let isSuccess = $state<boolean>(false);
	let buttonMsg = $state(`${6} digits left`);
	let selStart = $state<number | null>(null);
	let selEnd = $state<number | null>(null);

	let focusedIdx = $state<number | null>(null);

	const inputRefs: Array<HTMLInputElement | null> = Array(6).fill(null);

	function handleInput(e: Event, idx: number) {
		const target = e.currentTarget as HTMLInputElement;
		const val = target.value.replace(/\D/, '').slice(-1);
		error = false;
		isSuccess = false;
		inputs[idx] = val;

		// Move focus to next input
		if (val && idx < inputRefs.length - 1) {
			inputRefs[idx + 1]?.focus();
		}

		// Auto-submit on complete
		if (inputs.every((ch) => ch !== '')) {
			submitCode();
		}
	}

	function handleKeydown(e: KeyboardEvent, idx: number) {
		// Expand selection to the left
		if (e.shiftKey && e.key === 'ArrowLeft') {
			e.preventDefault();
			if (selStart === null) {
				selStart = idx;
				selEnd = idx;
			}
			if (selEnd !== null) {
				selEnd = Math.max(0, selEnd - 1);
				inputRefs[selEnd]?.focus();
			}
			return;
		}

		// Expand selection to the right
		if (e.shiftKey && e.key === 'ArrowRight') {
			e.preventDefault();
			if (selStart === null) {
				selStart = idx;
				selEnd = idx;
			}
			if (selEnd !== null) {
				selEnd = Math.min(5, selEnd + 1);
				inputRefs[selEnd]?.focus();
			}
			return;
		}

		// Cut selected range with Ctrl+X
		if (e.ctrlKey && e.key.toLowerCase() === 'x' && selStart !== null && selEnd !== null) {
			e.preventDefault();
			const start = Math.min(selStart, selEnd);
			const end = Math.max(selStart, selEnd);
			for (let i = start; i <= end; i++) {
				inputs[i] = '';
			}
			inputRefs[start]?.focus();
			selStart = null;
			selEnd = null;
			return;
		}

		// Backspace behavior (with or without selection)
		if (e.key === 'Backspace') {
			if (selStart !== null && selEnd !== null) {
				e.preventDefault();
				const start = Math.min(selStart, selEnd);
				const end = Math.max(selStart, selEnd);

				// Clear selected inputs
				for (let i = start; i <= end; i++) {
					inputs[i] = '';
				}

				// Reset selection + refocus

				selStart = null;
				selEnd = null;
				focusedIdx = start;
				inputRefs[start]?.focus();
				inputRefs[start]?.setSelectionRange?.(0, 0);

				return;
			} else {
				// Normal backspace behavior
				if (inputs[idx]) {
					inputs[idx] = '';
				} else if (idx > 0) {
					inputRefs[idx - 1]?.focus();
					inputs[idx - 1] = '';
					focusedIdx = idx - 1;
				}
			}
			return; // Prevent further nav after backspace
		}

		// Clear selection on normal nav
		if (!e.shiftKey && ['ArrowLeft', 'ArrowRight'].includes(e.key)) {
			selStart = null;
			selEnd = null;
		}

		if (e.key === 'ArrowLeft' && idx > 0) {
			inputRefs[idx - 1]?.focus();
		} else if (e.key === 'ArrowRight' && idx < inputRefs.length - 1) {
			inputRefs[idx + 1]?.focus();
		} else if (e.key === 'Enter') {
			submitCode();
		} else if (e.key.toLowerCase() === 'a' && e.ctrlKey) {
			(e.currentTarget as HTMLInputElement).select();
		}
	}

	function submitCode() {
		const code = inputs.join('');
		const ok = code === correctCode;

		if (code.length === 6) {
			error = !ok;
			isSuccess = ok;
		}
	}

	function handlePaste(e: ClipboardEvent) {
		e.preventDefault();
		const paste = (e.clipboardData?.getData('text') || '').replace(/\D/g, '').slice(0, 6);
		const chars = paste
			.padEnd(6, ' ')
			.split('')
			.map((ch) => (ch === ' ' ? '' : ch));
		console.log(paste);
		for (let i = 0; i < 6; i++) {
			inputs[i] = chars[i];
		}

		// Move focus to the first empty box
		const firstEmpty = chars.findIndex((ch) => ch === '');
		inputRefs[firstEmpty === -1 ? 5 : firstEmpty]?.focus();

		if (chars.every((ch) => ch !== '')) {
			submitCode();
		}
	}

	$effect(() => {
		inputRefs[0]?.focus();
	});

	$effect(() => {
		if (inputs.some((ch) => ch === '')) {
			error = false;
			isSuccess = false;
		}
	});

	function register(node: HTMLInputElement, idx: number) {
		inputRefs[idx] = node;
		return {
			destroy() {
				inputRefs[idx] = null;
			}
		};
	}

	$effect(() => {
		if (error) {
			buttonMsg = 'Wrong code!';
		} else if (!error && isSuccess) {
			buttonMsg = `Let’s Go!`;
		} else {
			buttonMsg = `${6 - inputs.filter(Boolean).length} digits left`;
		}
	});
</script>

<div class="flex min-h-screen items-center justify-center bg-blue-50">
	<div
		class="flex w-full max-w-md flex-col items-center space-y-6 rounded-3xl bg-white px-10 py-8 shadow-md"
	>
		<div
			class="flex h-20 w-20 items-center justify-center rounded-full transition-all duration-500"
		>
			{console.log(error, isSuccess)}
			{#if isSuccess}
				<img src={successLock} alt="Lock Status" />
			{:else if error}
				<img src={errorLock} alt="Lock Status" />
			{:else}
				<img src={defaultLock} alt="Lock Status" />
			{/if}
		</div>

		<h2 class="text-3xl font-medium text-gray-800">Easy peasy</h2>
		<p class="text-center text-xs text-gray-500">
			Enter 6-digit code from your two factor authenticator APP.
		</p>
		<div class="flex flex-col">
			<div class="flex space-y-5 space-x-3" onpaste={handlePaste}>
				{#each inputs as input, idx}
					<div class="relative h-14 w-11">
						<!-- Visible Box -->
						<div
							class="h-full w-full rounded-lg border-1 border-gray-300 text-center"
							class:border-gray-300={!error && input === '' && focusedIdx !== idx}
							class:border-blue-400={focusedIdx === idx || input !== ''}
							class:border-red-400={error}
							class:border-[#63acf8]={!error && isSuccess}
						></div>
						<!-- Actual Input (invisible, but interactive) -->
						<input
							use:register={idx}
							maxlength="1"
							class="absolute top-0 left-0 h-full w-full bg-transparent text-center text-2xl font-light text-transparent caret-transparent outline-none"
							class:caret-red-400={error}
							onfocus={() => (focusedIdx = idx)}
							onblur={() => (focusedIdx = null)}
							value={input}
							oninput={(e: Event) => handleInput(e, idx)}
							onkeydown={(e: KeyboardEvent) => handleKeydown(e, idx)}
							onkeypress={(e: KeyboardEvent) => {
								if (!VALID_KEYWORDS.has(e.key)) e.preventDefault();
							}}
						/>

						<!-- Horizontal Custom Cursor -->
						{#if  ((focusedIdx === idx) && ( !isSuccess && !error)) }
							<span
								class="animate-slide-in absolute bottom-0 left-1/2 h-0.5 w-[70%] -translate-y-1/2 rounded-sm bg-blue-400"
								class:bg-red-400={error}
								
							></span>
						{/if}

						<!-- Animated Digit Drop -->
						{#if input !== ''}
							<span
								class="drop-digit absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-2xl font-light text-blue-400"
								class:selected={selStart !== null &&
									selEnd !== null &&
									idx >= Math.min(selStart, selEnd) &&
									idx <= Math.max(selStart, selEnd)}
								class:error
							>
								{input}
							</span>
						{/if}
					</div>
				{/each}
			</div>

			<OtPbtn bind:error bind:isSuccess bind:buttonMsg />
		</div>
	</div>
</div>

<style>
	@keyframes drop-digit {
		0% {
			opacity: 0;
			transform: translateY(-100%) scale(1.2);
		}
		60% {
			opacity: 1;
			transform: translateY(10%) scale(1);
		}
		100% {
			transform: translateY(0) scale(1);
		}
	}

	.drop-digit {
		animation: drop-digit 0.6s ease-out;
	}

	.error {
		background-color: #fef8f8;
		color: #f44336;
		border-color: #f87171;
	}

	.selected {
		background-color: rgba(99, 172, 248, 0.2);
		border-radius: 4px;
		padding: 2px 4px;
	}

	@keyframes slide-in {
		from {
			transform: translate(-50%, 0);
			opacity: 1;
		}
		to {
			transform: translate(-50%, -10px);
			opacity: 1;
		}
	}

	.animate-slide-in {
		animation: slide-in 0.5s forwards;
	}
</style>

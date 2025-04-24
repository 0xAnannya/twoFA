<script lang="ts">
	import { fly } from 'svelte/transition';

	const inputs = $state<string[]>(['', '', '', '', '', '']);
	let error = $state<boolean>(false);
	let isSuccess = $state<boolean>(false);

	let selStart = $state<number | null>(null);
	let selEnd = $state<number | null>(null);
	const correctCode = '123456';
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
		if (e.key === 'Backspace' && !inputs[idx] && idx > 0) {
			inputRefs[idx - 1]?.focus();
		} else if (e.key === 'ArrowLeft' && idx > 0) {
			inputRefs[idx - 1]?.focus();
		} else if (e.key === 'ArrowRight' && idx < inputRefs.length - 1) {
			inputRefs[idx + 1]?.focus();
		} else if (e.key === 'Enter') {
			submitCode();
		} else if (e.key.toLowerCase() === 'a' && e.ctrlKey) {
			(e.currentTarget as HTMLInputElement).select();
		}
	}

	// function handleKeydown(e: KeyboardEvent, idx: number) {
	//     // Shift+Left: expand selection start->left
	//     if (e.shiftKey && e.key === 'ArrowLeft') {
	//       e.preventDefault();
	//       if (selStart == null) selStart = idx;
	//       selEnd = Math.max(0, idx - 1);
	//       return;
	//     }
	//     // Shift+Right: expand selection start->right
	//     if (e.shiftKey && e.key === 'ArrowRight') {
	//       e.preventDefault();
	//       if (selStart == null) selStart = idx;
	//       selEnd = Math.min(5, idx + 1);
	//       return;
	//     }
	//     // Ctrl+X: cut selected range
	//     if (e.ctrlKey && e.key.toLowerCase() === 'x' && selStart != null && selEnd != null) {
	//       e.preventDefault();
	//       const start = Math.min(selStart, selEnd);
	//       const end = Math.max(selStart, selEnd);
	//       for (let i = start; i <= end; i++) inputs[i] = '';
	//       inputRefs[start]?.focus();
	//       inputRefs[start]?.setSelectionRange(0, 0);
	//       selStart = null;
	//       selEnd = null;
	//       return;
	//     }
	//     // Enter to submit
	//     if (e.key === 'Enter') { submitCode(); return; }
	//     // Arrow navigation without shift
	//     if (!e.shiftKey && e.key === 'ArrowLeft' && idx > 0) {
	//       inputRefs[idx - 1]?.focus();
	//       inputRefs[idx - 1]?.setSelectionRange(0, 0);
	//       selStart = null;
	//       selEnd = null;
	//       return;
	//     }
	//     if (!e.shiftKey && e.key === 'ArrowRight' && idx < 5) {
	//       inputRefs[idx + 1]?.focus();
	//       inputRefs[idx + 1]?.setSelectionRange(0, 0);
	//       selStart = null;
	//       selEnd = null;
	//       return;
	//     }
	//     // Backspace navigation
	//     if (e.key === 'Backspace' && !inputs[idx] && idx > 0) {
	//       inputRefs[idx - 1]?.focus();
	//       inputRefs[idx - 1]?.setSelectionRange(0, 0);
	//       selStart = null;
	//       selEnd = null;
	//       return;
	//     }
	//   }

	function submitCode() {
		const code = inputs.join('');
		const ok = code === correctCode;

		if (code.length === 6) {
			error = !ok;
			isSuccess = ok;
		}
	}

	function handlePaste(e: ClipboardEvent) {
		const paste = (e.clipboardData?.getData('text') || '').replace(/\D/g, '').slice(0, 6);
		// Populate inputs by mutation rather than reassigning const
		console.log(paste);
		const chars = paste
			.padEnd(6, ' ')
			.split('')
			.map((ch) => (ch === ' ' ? '' : ch));
		for (let i = 0; i < 6; i++) {
			inputs[i] = chars[i];
		}
		console.log(inputs, chars);
		console.log(paste.length);

		inputRefs[Math.min(paste.length - 1, 5)]?.focus();

		// Auto-submit after pasting full code
		if (inputs.every((ch) => ch !== '')) {
			submitCode();
		}
	}

	$effect(() => {
		inputRefs[0]?.focus();
	});

	function register(node: HTMLInputElement, idx: number) {
		inputRefs[idx] = node;
		return {
			destroy() {
				inputRefs[idx] = null;
			}
		};
	}
</script>

<div class="flex min-h-screen items-center justify-center bg-blue-50">
	<div
		class="flex w-full max-w-md flex-col items-center space-y-6 rounded-3xl bg-white px-10 py-8 shadow-md"
	>
		<div
			class="flex h-20 w-20 items-center justify-center rounded-full transition-all duration-500"
		>
			{#if !error && isSuccess}
				<!-- Unlocked Icon -->
				<svg
					class="h-10 w-10 rotate-[20deg] text-green-500 transition-transform duration-500"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 11c-1.104 0-2 .895-2 2v4h4v-4c0-1.105-.896-2-2-2zm4-4V7a4 4 0 00-8 0v2h2V7a2 2 0 014 0v2h2z"
					/>
				</svg>
			{:else if error && !isSuccess}
				<!-- Locked Icon (Red) -->
				<svg
					class="h-10 w-10 scale-110 text-red-400 transition-all duration-500"
					fill="none"
					viewBox="0 0 24 24"
					stroke="currentColor"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 11c-1.104 0-2 .895-2 2v4h4v-4c0-1.105-.896-2-2-2zm0-6a4 4 0 00-4 4v2h8V9a4 4 0 00-4-4z"
					/>
				</svg>
			{:else}
				<!-- Locked Icon (Default) -->

				<svg
					fill="#87Ceeb"
					width="118px"
					height="118px"
					viewBox="-11.52 -11.52 47.04 47.04"
					xmlns="http://www.w3.org/2000/svg"
					stroke="#87Ceeb"
					stroke-width="0.00024000000000000003"
					><g id="SVGRepo_bgCarrier" stroke-width="0"
						><rect x="-11.52" y="-11.52" width="47.04" height="47.04" rx="23.52" fill="#EAF4FB"
						></rect></g
					><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g
						id="SVGRepo_iconCarrier"
						><path
							d="M12,1A5.006,5.006,0,0,0,7,6V8.765a8,8,0,1,0,10,0V6A5.006,5.006,0,0,0,12,1ZM9,6a3,3,0,0,1,6,0V7.589a7.935,7.935,0,0,0-6,0Zm9,9a6,6,0,1,1-6-6A6.006,6.006,0,0,1,18,15Zm-4-1a1.994,1.994,0,0,1-1,1.723V18a1,1,0,0,1-2,0V15.723A2,2,0,1,1,14,14Z"
						></path></g
					></svg
				>
			{/if}
		</div>

		<h2 class="text-3xl font-medium text-gray-800">Easy peasy</h2>
		<p class="text-center text-xs text-gray-500">
			Enter 6-digit code from your two factor authenticator APP.
		</p>

		<div class="flex space-x-3" onpaste={handlePaste}>
			{#each inputs as input, idx}
				<input
					use:register={idx}
					maxlength="1"
					class="font-lg h-13 w-11 rounded-lg border-2 text-center text-xl transition-colors duration-200 outline-none"
					class:border-blue-400={!error}
					class:border-red-400={error}
					class:text-blue-500={!error}
					class:text-red-500={error}
					class:selected={selStart != null &&
						selEnd != null &&
						idx >= Math.min(selStart, selEnd) &&
						idx <= Math.max(selStart, selEnd)}
					value={input}
					oninput={(e: Event) => handleInput(e, idx)}
					onkeydown={(e: KeyboardEvent) => handleKeydown(e, idx)}
					onkeypress={(e: KeyboardEvent) => {
						if (
							!/^[0-9]$/.test(e.key) &&
							e.key !== 'Backspace' &&
							e.key !== 'ArrowLeft' &&
							e.key !== 'ArrowRight'
						) {
							e.preventDefault();
						}
					}}
					onsubmit={submitCode}
				/>
			{/each}
		</div>
		{#if error}
			<button
				class="mt-4 w-full rounded-xl bg-red-400 py-3 text-lg font-semibold text-white shadow-lg"
			>
				Wrong code!
			</button>
		{:else if !error && isSuccess}
			<div in:fly={{ x: 400, duration: 200, opacity: 0, delay: 300 }} class="w-full">
				<button
					class="mt-4 w-full rounded-xl bg-blue-600 py-3 text-lg font-semibold text-white shadow-lg"
					>Let's Go</button
				>
			</div>
		{:else}
			<button
				class="mt-4 w-full rounded-xl bg-gray-200 py-3 text-lg font-semibold text-gray-600 shadow-inner"
				onclick={submitCode}
			>
				{6 - inputs.filter(Boolean).length} digits left
			</button>
		{/if}
	</div>
</div>

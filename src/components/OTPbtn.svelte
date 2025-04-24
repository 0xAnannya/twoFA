<script lang="ts" >
  // Bindable props from parent
  let {
    enteredOtp = $bindable<string[]>(),
    buttonMsg = $bindable<string>(),
  } = $props();

  const CORRECT_OTP='123456'
</script>

<button
  disabled={enteredOtp.join('').length < 6}
  class="w-full py-3 mt-4 rounded-xl font-semibold text-lg relative overflow-hidden"
  class:animate-shake={enteredOtp.join('').length === 6 && enteredOtp.join('') !== CORRECT_OTP}
  style={
    enteredOtp.join('').length < 6
      ? 'background-position: left bottom;'
      : enteredOtp.join('') !== CORRECT_OTP
      ? 'background: #FD4B63; color: white;'
      : 'color: white; background-position: right bottom;'
  }
>
  {buttonMsg}
</button>

<style lang="postcss">
  @tailwind utilities;
  @layer components {
    button {
      @apply cursor-pointer text-center;
      background: linear-gradient(to left, #0794ff 50%, #edeef1 50%);
      background-size: 200% 100%;
      background-repeat: no-repeat;
      transition: background-position 0.5s ease;
    }
  }
  @layer utilities {
    @keyframes shake {
      0% { transform: translateX(0); }
      25% { transform: translateX(-5px); }
      50% { transform: translateX(5px); }
      75% { transform: translateX(-5px); }
      100% { transform: translateX(0); }
    }
    .animate-shake {
      animation: shake 0.3s ease-in-out both;
    }
  }
</style>
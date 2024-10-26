<script>
  import { SquarePlus  } from 'lucide-svelte';
</script>

<!-- Progress bar-->

<div class="mt-32 text-2xl">
<div class="text-center w-screen flex flex-col justify-center content-center">
<div class="w-[40%] bg-gray-200 rounded-full h-2.5 dark:bg-gray-700">
  <div class="bg-blue-600 h-2.5 rounded-full w-[45%]"></div>
</div>

<div class="">
<p class="text-left">0/500 ML calls used</p>
</div>
</div>
<div class="mt-16">
<div class="flex flex-row justify-around min-w-screen">
<h1 class="color-white text-4xl">Your models</h1>
<button class="bg-cyan-600 inline rounded p-2"><SquarePlus class="inline"/> Create New</button>
</div>
</div>
</div>

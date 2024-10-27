<script>
  import { SquarePlus  } from 'lucide-svelte';
  import Navbar from "../../components/common/navbar.svelte"
</script>

<!-- Progress bar-->
<Navbar />
<div class="mt-24 text-2xl flex flex-col">
<div class=" bg-indigo-200 w-[50%] text-center flex flex-col justify-center self-center items-center p-3 rounded-full">
<div class="w-[70%] bg-gray-200 rounded-full h-5 dark:bg-gray-700">
  <div class="bg-blue-600 h-full rounded-full w-[0%]"></div>
</div>

<div class="mt-2 rounded">
<p class="text-center">0/500 ML calls used</p>
</div>
</div>
<div class="mt-24">
<div class="flex flex-row justify-around min-w-screen">
<h1 class="color-white text-4xl">Your models</h1>
<button class="bg-cyan-600 inline rounded-xl p-2"><SquarePlus class="inline"/> Create New</button>
</div>
</div>
</div>

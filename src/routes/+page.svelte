<script>
  import "../app.css";
  import { sound, save, uuidv4 } from "$lib/utils.js";
  import settings from "$lib/settings.json";
  import { md5 } from "js-md5";


  let state = 0;
  let time = 0;
  let last = 0;
  let iter = 0;
  let t;
  let visual = true;
  let stop = false;

  let data = {
    visual: [],
    auditory: []
  }

  const next = e => {
    if (e.keyCode !== 32) return;

    if (stop) state = 6;

    if ([0, 1, 4, 5].indexOf(state) != -1) {
      let pause = settings.minWaitTime + Math.random() * (settings.maxWaitTime - settings.minWaitTime);

      if (visual) {
        t = setTimeout(() => {
          state = 3;
          time = new Date(); 
        }, pause);
      } else {
        t = setTimeout(() => {
          sound(settings.soundFrequency, settings.soundDuration);
          time = new Date();
        }, pause)
      }

      if (state != 5) iter += 1;
      state = 2;
    } else if (state == 2) {
      let gap = new Date() - time;

      if (!visual && gap > 0) {
        data.auditory.push(gap);
        last = gap;
        state = 4;
      } else {
        clearTimeout(t);
        state = 5;
      }
    } else if (state == 3) {
      let gap = new Date() - time;

      data.visual.push(gap);
      last = gap;
      state = 4;
    }

    if (data.visual.length == settings.samples && visual) {
      visual = false;
      iter = 0;
      state = 1;
    } if (data.auditory.length == settings.samples) {
      stop = true;
    }
  }

  const download = () => {
    let birth = document.querySelector("#birth");
    let sex = document.querySelector("#sex");

    if (!birth.value) return;

    data.birth = birth.value;
    data.sex = sex.value;
    data.timestamp = new Date();
    data.exercise = false;

    let string = JSON.stringify(data);

    string = string + " - " + md5(string);
    save(uuidv4()+".json", string);
  }
</script>

<svelte:window on:keydown={next} />

<header class="h-12 w-full shadow-lg absolute bg-white z-10">
  <div class="h-full w-[970px] mx-auto flex items-center" data-sveltekit-reload>
    <div class="h-full flex items-center px-6">
      Reaction Time
    </div>
    <a class="h-full flex items-center px-6 hover:bg-gray-100" href="/exercise">
      Exercise
    </a>
    <div class="grow"/>
    <a class="h-full flex items-center px-6 hover:bg-gray-100" href="/">
      Restart
    </a>
  </div>
</header>

<div class="h-screen w-screen relative">
  {#if iter}
  <div class="absolute top-14 right-4 text-white text-[24px]">
    {iter}/{settings.samples}
  </div>
  {/if}
  {#if state == 0}
  <div class="h-screen bg-[#2b87d1] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-bolt-lightning text-[120px]" />
    <h1 class="text-[80px]">Visual Reaction Time</h1>
    <h2 class="text-[24px]">When the red box turns green, press <i>{"<Space>"}</i> as quickly as you can.</h2>
    <h2 class="text-[24px]">Press <i>{"<Space>"}</i> to start.</h2>
  </div>
  {:else if state == 1}
  <div class="h-screen bg-[#2b87d1] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-headphones text-[120px]" />
    <h1 class="text-[80px]">Auditory Reaction Time</h1>
    <h2 class="text-[24px]">When you hear a beep sound, press <i>{"<Space>"}</i> as quickly as you can.</h2>
    <h2 class="text-[24px]">Press <i>{"<Space>"}</i> to start.</h2>
  </div>
  {:else if state == 2}
  <div class="h-screen bg-[#ce2636] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-bolt-lightning text-[120px] italic opacity-0" />
    <h1 class="text-[80px]">Wait...</h1>
    <h2 class="text-[24px] opacity-0"> </h2>
    <h2 class="text-[24px] opacity-0"> </h2>
  </div>
  {:else if state == 3}
  <div class="h-screen bg-[#4bdb6a] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-bolt-lightning text-[120px] italic opacity-0" />
    <h1 class="text-[80px]">Go!</h1>
    <h2 class="text-[24px] opacity-0"> </h2>
    <h2 class="text-[24px] opacity-0"> </h2>
  </div>
  {:else if state == 4}
  <div class="h-screen bg-[#2b87d1] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-stopwatch text-[120px]" />
    <h1 class="text-[80px]">{last} ms</h1>
    <h2 class="text-[24px]">Press <i>{"<Space>"}</i> to keep going.</h2>
    <h2 class="text-[24px] opacity-0"> </h2>
  </div>
  {:else if state == 5}
  <div class="h-screen bg-[#2b87d1] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-triangle-exclamation text-[120px]" />
    <h1 class="text-[80px]">You pressed too early.</h1>
    <h2 class="text-[24px]">Press <i>{"<Space>"}</i> to try again.</h2>
    <h2 class="text-[24px] opacity-0"> </h2>
  </div>
  {:else if state == 6}
  <div class="h-screen bg-[#2b87d1] text-white flex flex-col justify-center items-center">
    <i class="fa-solid fa-file-arrow-down text-[120px]" />
    <h1 class="text-[80px] mb-10">Download your results</h1>
    <div class="grid grid-cols-3 text-[24px] border-white border-2 rounded-[1.1rem] px-4 py-4 gap-4">
      <div class="border-white border-2 rounded-lg px-3 py-1 flex justify-between col-span-2">
        <span class="mr-2">Date of birth:</span>
        <input type="date" class="bg-transparent" id="birth">
      </div>
      <div class="border-white border-2 rounded-lg px-3 py-1 flex justify-between">
        <span class="mr-2">Sex:</span>
        <select class="bg-transparent" id="sex">
          <option value="male">Male</option>
          <option value="female">Female</option>
          <option value="intersex">Intersex</option>
        </select>
      </div>
      <button class="border-white border-2 rounded-lg col-span-3" on:click={download}>
        Download
      </button>
    </div>
  </div>
  {/if}
</div>
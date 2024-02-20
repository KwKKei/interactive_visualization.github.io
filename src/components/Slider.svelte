<script>
    import { createEventDispatcher, onDestroy } from 'svelte';
  
    export let selectedYear = 1991;
    export let autoplay = false;
  
    const dispatch = createEventDispatcher();
  
    let intervalId;
  
    const handleChange = (event) => {
      selectedYear = parseInt(event.target.value);
      dispatch('change', selectedYear);
    };
  
    const handleAutoplay = () => {
      autoplay = !autoplay;
      if (autoplay) {
        intervalId = setInterval(() => {
          if (selectedYear < 2019) {
            selectedYear++;
          } else {
            selectedYear = 1991;
          }
          dispatch('change', selectedYear);
        }, 150); // Change the delay as needed
      } else {
        clearInterval(intervalId);
      }
    };
  
    onDestroy(() => {
      clearInterval(intervalId);
    });
  </script>
  
  <label for="year">Select a year:</label>
  <input type="range" id="year" min="1991" max="2019" bind:value={selectedYear} on:input={handleChange}>
  <span>{selectedYear}</span>
  <button on:click={handleAutoplay}>{autoplay ? 'Pause' : 'Play'}</button>
<script>
  function Scrubber(values, {
    format = value => value,
    initial = 0,
    direction = 1,
    delay = null,
    autoplay = true,
    loop = true,
    loopDelay = null,
    alternate = false
  } = {}) {
    values = Array.from(values);

    let frame = null;
    let timer = null;
    let interval = null;

    function start() {
      form.b.textContent = "Pause";
      if (delay === null) frame = requestAnimationFrame(tick);
      else interval = setInterval(tick, delay);
    }

    function stop() {
      form.b.textContent = "Play";
      if (frame !== null) cancelAnimationFrame(frame), frame = null;
      if (timer !== null) clearTimeout(timer), timer = null;
      if (interval !== null) clearInterval(interval), interval = null;
    }

    function running() {
      return frame !== null || timer !== null || interval !== null;
    }

    function tick() {
      if (form.i.valueAsNumber === (direction > 0 ? values.length - 1 : direction < 0 ? 0 : NaN)) {
        if (!loop) return stop();
        if (alternate) direction = -direction;
        if (loopDelay !== null) {
          if (frame !== null) cancelAnimationFrame(frame), frame = null;
          if (interval !== null) clearInterval(interval), interval = null;
          timer = setTimeout(() => (step(), start()), loopDelay);
          return;
        }
      }
      if (delay === null) frame = requestAnimationFrame(tick);
      step();
    }

    function step() {
      form.i.valueAsNumber = (form.i.valueAsNumber + direction + values.length) % values.length;
      form.i.dispatchEvent(new CustomEvent("input", { bubbles: true }));
    }

    function handleInput(event) {
      if (event && event.isTrusted && running()) stop();
      form.value = values[form.i.valueAsNumber];
      form.o.value = format(form.value, form.i.valueAsNumber, values);
    }

    function handleClick() {
      if (running()) return stop();
      direction = alternate && form.i.valueAsNumber === values.length - 1 ? -1 : 1;
      form.i.valueAsNumber = (form.i.valueAsNumber + direction) % values.length;
      form.i.dispatchEvent(new CustomEvent("input", { bubbles: true }));
      start();
    }

    const form = {
      value: values[initial],
      b: { textContent: autoplay ? "Pause" : "Play" },
      i: { valueAsNumber: initial },
      o: { value: format(values[initial], initial, values) },
      start,
      stop,
      running,
      handleInput,
      handleClick
    };

    form.i.oninput = handleInput;
    form.b.onclick = handleClick;

    if (autoplay) start();
    else stop();

    // Ensure cleanup when component is destroyed
    onDestroy(() => {
      stop();
    });

    return form;
  }
</script>

<!-- HTML template -->
<div style="font: 12px var(--sans-serif); font-variant-numeric: tabular-nums; display: flex; height: 33px; align-items: center;">
  <button on:click="{b.onclick}" style="margin-right: 0.4em; width: 5em;">{b.textContent}</button>
  <label style="display: flex; align-items: center;">
    <input type="range" min="0" max={values.length - 1} value="{i.valueAsNumber}" step="1" style="width: 180px;" on:input="{i.oninput}">
    <output style="margin-left: 0.4em;">{o.value}</output>
  </label>
</div>
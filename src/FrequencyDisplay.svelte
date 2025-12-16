<script>
  export let freq = 0;

  let freqHis = [];

  const minFreq = 1;
  const maxFreq = 1000;

  $: {
    if (freq != null) {
      freqHis = [...freqHis, freq].slice(-4); // keep last 4
    }
  }

  // Compute positions for each cursor
  $: positions = freqHis.map(
    (f) => Math.min(Math.max((f - minFreq) / (maxFreq - minFreq), 0), 1) * 100
  );
</script>

<div
  style="width: 100%; height: 30px; background: #eee; border: 1px solid #999; position: relative;"
>
  {#each positions as pos, i (i)}
    <div
      style="
				position: absolute;
				left: {pos}%;
				top: 0;
				width: 4px;
				height: 100%;
				background: rgba(255, 0, 0, {(i + 1) / positions.length});
				transform: translateX(-50%);
			"
    ></div>
  {/each}
</div>

<div style="text-align: center; margin-top: 5px;">
  Frequency: {freq.toFixed(2)} Hz
</div>

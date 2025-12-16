<script>
  import { PitchDetector } from "pitchy";

  let audioContext, source, processor;
  let detector;
  const bufferLength = 2048;

  export let currentNote = "Press start";
  let noteHistory = [];
  let analysis = {};
  let quietCount = 0;
  let minDb = -30;
  let showhistory = false;

  function start() {
    navigator.mediaDevices.getUserMedia({ audio: true }).then((stream) => {
      audioContext = new AudioContext();
      source = audioContext.createMediaStreamSource(stream);

      processor = audioContext.createScriptProcessor(bufferLength, 1, 1);
      source.connect(processor);
      processor.connect(audioContext.destination);

      detector = PitchDetector.forFloat32Array(bufferLength);

      processor.onaudioprocess = (evt) => {
        const input = evt.inputBuffer.getChannelData(0);

        // Compute RMS and dB
        let sum = 0;
        for (let i = 0; i < input.length; i++) sum += input[i] * input[i];
        const rms = Math.sqrt(sum / input.length);
        const db = 20 * Math.log10(rms);

        if (db < minDb) {
          quietCount += 1;
          return;
        }
        quietCount = 0;

        const [freq, clarity] = detector.findPitch(
          input,
          audioContext.sampleRate
        );
        if (!freq || clarity < 0.8) return;

        // Transpose tenor sax +2 semitones
        const transposed = freq * Math.pow(2, 2 / 12);
        const note = freqToNote(transposed);

        // Update history and analysis
        if (noteHistory.length > 10) noteHistory.shift();
        noteHistory.push();
        noteHistory = [...noteHistory, note];

        if (!analysis[note]) analysis[note] = { freq: 0, db: 0, count: 0 };
        analysis[note] = {
          freq: transposed,
          db,
          count: (analysis[note].count || 0) + 1,
        };

        currentNote = sampleAnalysis(noteHistory);
      };
    });
  }

  function stop() {
    if (processor) processor.disconnect();
    if (source) source.disconnect();
    if (audioContext) audioContext.close();
    audioContext = null;
    source = null;
    processor = null;
  }

  function sampleAnalysis(features) {
    const counts = {};
    let maxCount = 0,
      mostCommon = null;
    for (const n of features) {
      counts[n] = (counts[n] || 0) + 1;
      if (counts[n] > maxCount) {
        maxCount = counts[n];
        mostCommon = n;
      }
    }
    return mostCommon;
  }

  function freqToNote(freq) {
    const noteNames = [
      "C",
      "C#",
      "D",
      "D#",
      "E",
      "F",
      "F#",
      "G",
      "G#",
      "A",
      "A#",
      "B",
    ];
    const A4 = 440;
    const semitone = 12 * Math.log2(freq / A4);
    const index = Math.round(semitone) + 57;
    return noteNames[((index % 12) + 12) % 12] + Math.floor(index / 12);
  }
</script>

<div>
  {#if audioContext}
    <button on:click={stop}>🔴 Stop sampling</button>
  {:else}
    <button on:click={start}>Start sampling</button>
  {/if}

  <h2>{currentNote}</h2>
  <label>
    DB cutoff: {minDb}
    <input type="range" bind:value={minDb} min="-100" max="-1" />
  </label>
  <button on:click={() => (showhistory = !showhistory)}>Toggle history</button>

  {#if showhistory}
    <section>
      <h4>History</h4>
      <ul>
        {#each noteHistory as n}
          <li>{n}</li>
        {/each}
      </ul>
    </section>
  {/if}
</div>

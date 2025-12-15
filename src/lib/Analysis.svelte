<script>
  import Meyda from "meyda";
  import { onMount } from "svelte";

  let noteHistory = [];
  let tooQuiet = false;
  let quietCount = 0;
  export let currentNote = "Press start";

  let minDb = -40;
  let bufferSize = 10;

  let stream;
  let audioContext;
  let source;
  let analyzer;
  function start() {
    navigator.mediaDevices.getUserMedia({ audio: true }).then((s) => {
      stream = s;
      audioContext = new AudioContext();
      source = audioContext.createMediaStreamSource(stream);

      analyzer = Meyda.createMeydaAnalyzer({
        audioContext,
        source,
        bufferSize: 1024,
        featureExtractors: ["amplitudeSpectrum", "rms"],
        callback: (features) => {
          if (!features) return;
          if (features.rms) {
            const db = rmsToDb(features.rms);
            if (db < minDb) {
              quietCount += 1;
              return;
            }
          }
          quietCount = 0;

          const spectrum = features.amplitudeSpectrum;
          let maxIndex = 0;
          for (let i = 1; i < spectrum.length; i++) {
            if (spectrum[i] > spectrum[maxIndex]) maxIndex = i;
          }
          const freq = (maxIndex * audioContext.sampleRate) / 1024;
          //   console.log(`${freqToNote(freq)} - ${freq.toFixed(2)} Hz`);
          if (noteHistory.length > bufferSize) {
            noteHistory.shift();
          }
          noteHistory = [...noteHistory, freqToNote(freq)];
          currentNote = sampleAnalysis(noteHistory);
        },
      });

      analyzer.start();
    });
  }
  function stop() {
    if (analyzer) analyzer.stop();
    if (source) source.disconnect();
    if (audioContext) audioContext.close();
    if (stream) {
      stream.getTracks().forEach((track) => track.stop());
    }
    stream = null;
    audioContext = null;
    source = null;
    analyzer = null;
  }

  function sampleAnalysis(features) {
    // Count most common note
    const counts = {};
    let maxCount = 0;
    let mostCommon = null;
    for (const n of noteHistory) {
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
    const noteIndex = Math.round(semitone) + 57;
    const octave = Math.floor(noteIndex / 12);
    const note = noteNames[((noteIndex % 12) + 12) % 12];
    return `${note}${octave}`;
  }
  // Convert RMS to decibels
  function rmsToDb(rms) {
    return 20 * Math.log10(rms);
  }
  onMount(() => {});

  let showhistory = false;
</script>

{#if analyzer}
  <button on:click={stop}>🔴 Stop sampling</button>
{:else}
  <button on:click={start}>Start sampling</button>
{/if}
<div>
  <h2>{currentNote}</h2>
  {#if quietCount > 8}
    <h4>Too quiet!</h4>
  {/if}
  <label for="">
    DB cutoff: {minDb}
    <input type="range" bind:value={minDb} min="-100" max="-1" />
  </label>
  <button on:click={() => (showhistory = !showhistory)}>Toggle history</button>
  {#if showhistory}
    <section>
      <h4>History</h4>
      <ul>
        {#each noteHistory as analysisInc}
          <li>
            {analysisInc}
          </li>
        {/each}
      </ul>
    </section>
  {/if}
</div>

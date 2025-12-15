<script>
  import Meyda from "meyda";
  import { onMount } from "svelte";

  let featureHistory = [];
  let mostCommon = "Press start";
  let bufferSize = 20;

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
        featureExtractors: ["amplitudeSpectrum"],
        callback: (features) => {
          if (!features) return;
          const spectrum = features.amplitudeSpectrum;
          let maxIndex = 0;
          for (let i = 1; i < spectrum.length; i++) {
            if (spectrum[i] > spectrum[maxIndex]) maxIndex = i;
          }
          const freq = (maxIndex * audioContext.sampleRate) / 1024;
          //   console.log(`${freqToNote(freq)} - ${freq.toFixed(2)} Hz`);
          if (featureHistory.length > bufferSize) {
            featureHistory.shift();
          }
          featureHistory = [...featureHistory, freqToNote(freq)];
          mostCommon = sampleAnalysis(featureHistory);
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
    for (const n of featureHistory) {
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

  onMount(() => {});
</script>

{#if analyzer}
  <button on:click={stop}>Stop sampling</button>
{:else}
  <button on:click={start}>Start sampling</button>
{/if}
<div>
  <h2>{mostCommon}</h2>
  <section>
    <h4>History</h4>
    <ul>
      {#each featureHistory as analysisInc}
        <li>
          {analysisInc}
        </li>
      {/each}
    </ul>
  </section>
</div>

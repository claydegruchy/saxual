<script lang="ts">
  import Analysis from "./lib/Analysis.svelte";
  import Dialog from "./lib/Blocks/Dialog.svelte";
  import DisplayNote from "./lib/DisplayNote.svelte";
  
  
  let currentNote;
</script>

<main>
  <!-- <nav class="top centre">
    <section class="flex scroll">toolbar</section>
  </nav> -->
  <nav class="bottom left">
    <Dialog>
      <div slot="button">?</div>
      <div slot="content">
        <h2>What is this?</h2>
        <p>This is a digitisation of parts of the WFRP TOW rulebook.</p>

        <h3>Why did you make this?</h3>
        <p>I find trudging around in pdfs annoying.</p>

        <h3>I think this looks like shit</h3>
        <p>thats just like your opinion, man</p>
      </div>
    </Dialog>
  </nav>
  <DisplayNote {currentNote} />
  <section>
    <Analysis bind:currentNote></Analysis>
  </section>
</main>

<style>
  .scroll {
    display: flex;
    flex-wrap: nowrap; /* keep everything in a single row */
    overflow-x: auto; /* horizontal scroll */
    overflow-y: hidden;
    -webkit-overflow-scrolling: touch;
    width: 100%; /* ensure it fills parent width */
  }

  .scroll button {
    flex: 0 0 auto; /* do not grow or shrink, use content width */
    margin-right: 0.5rem; /* optional spacing between buttons */
  }
</style>

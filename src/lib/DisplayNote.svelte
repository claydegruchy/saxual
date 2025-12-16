<script>
  import { saxyMap } from "../utility";

  export let currentNote;

  function mapNoteToSaxy(note) {
    let result = saxyMap[note] || "?"; // empty graph fallback
    return result;
  }

  $: glyph = mapNoteToSaxy(currentNote);
</script>

<!-- <section> -->

<section>
  {#if glyph == "?"}
    <div class="saxy red">s</div>
  {:else}
    <div class="saxy">{glyph}</div>
  {/if}
</section>

<!-- </section> -->
{#if true}
  <div class="flex">
    {#each Object.entries(saxyMap) as [name, noteGlyph]}
      <section>
        {name}
        <div class="saxy {noteGlyph == glyph ? 'green' : ''}">{noteGlyph}</div>
      </section>
    {/each}
  </div>
{/if}

<style>
  @font-face {
    font-family: "Saxy";
    src: url("/fonts/Saxy.ttf") format("truetype");
  }

  .red {
    color: red;
  }

  .green {
    background-color: lightgreen;
  }
  .saxy {
    font-family: "Saxy";
    font-size: 120px;
  }
</style>

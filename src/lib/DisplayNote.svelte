<script>
  import { saxyMap } from "../utility";

  export let currentNote;

  function mapNoteToSaxy(note) {
    let result = saxyMap[note] || ""; // empty graph fallback
    return "s" + result;
  }

  $: glyph = mapNoteToSaxy(currentNote);
</script>

<div class="saxy">{glyph}</div>

<style>
  @font-face {
    font-family: "Saxy";
    src: url("/fonts/Saxy.ttf") format("truetype");
  }

  .saxy {
    font-family: "Saxy";
    font-size: 120px;
    color: black;
  }
</style>

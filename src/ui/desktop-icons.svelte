<script>
  let {src, name} = $props();
</script>

<main>
  <div class="icon">
    <img class="program-icon" {src} width="48px" alt={src} />
    <p class="label">{name}</p>
  </div>
</main>

<style>
  .icon {
    padding: 7px;
    max-height: 95px;
    padding-bottom: 0px;
    width: 71px;
    margin: 8px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    border-radius: 6px;
    background-color: transparent;
    cursor: pointer;
    user-select: none;
  }

  .icon:hover {
    background-color: #a7a7a760;
  }
  .label {
    font-weight: 500;
    font-size: 12px;
    color: white;
    text-shadow: 2px 2px 4px #000000;
    text-align: center;
    overflow: visible;
  }
</style>
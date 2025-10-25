<template>
  <header class="header">
    <div class="title-wrap">
      <h1 class="title">
        <span class="title-text">Pokémon Card Gallery</span>
        <svg class="poke-icon" viewBox="0 0 24 24" aria-hidden="true">
          <defs>
            <clipPath id="ballCut">
              <circle cx="12" cy="12" r="10"></circle>
            </clipPath>
          </defs>
          <g clip-path="url(#ballCut)">
            <rect x="2" y="2"  width="20" height="10" fill="#e53935"></rect>
            <rect x="2" y="12" width="20" height="10" fill="#ffffff"></rect>
            <rect x="2" y="11" width="20" height="2"  fill="#111111"></rect>
          </g>
          <circle cx="12" cy="12" r="10" fill="none" stroke="#111" stroke-width="2"></circle>
          <circle cx="12" cy="12" r="3"  fill="#fff" stroke="#111" stroke-width="2"></circle>
        </svg>
      </h1>
      <p class="subtitle">powered by PokeAPI</p>
    </div>
  </header>
</template>

<script>
export default {
  name: 'AppHeader',  
}
</script>

<style>

</style>
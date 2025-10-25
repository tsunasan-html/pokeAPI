<template>
  <div class="home">
    <AppHeader />

    <div class="search-row">
      <input v-model="search" class="search" placeholder="Search by name…" />
      <p v-if="!loading && search_pokemons.length > 0" class="meta">
        Results: {{ search_pokemons.length }}
      </p>
    </div>

    <div v-if="loading" class="spinner-wrap">
      <div class="spinner"></div>
    </div>

    <div v-else-if="search_pokemons.length === 0" class="no-result">
      No Pokémon found matching your search.
    </div>

    <div v-else class="grid">
      <article
        v-for="pokemon in search_pokemons"
        :key="pokemon.id"
        class="card"
        :data-type="pokemon.types[0] || 'normal'"
      >
        <div class="card-frame">
          <!-- 名前・HP -->
          <div class="card-header">
            <span class="name">{{ pokemon.name }}</span>
            <span class="hp">HP {{ pokemon.hp }}</span>
          </div>

          <div class="image-wrap">
            <img :src="pokemon.image" :alt="pokemon.name" loading="lazy" />
          </div>

          <div class="types">
            <span v-for="t in pokemon.types" :key="t" class="type" :data-type="t">
              {{ t }}
            </span>
            <span class="id">#{{ String(pokemon.id).padStart(3, '0') }}</span>
          </div>

          <div class="stats">
            <div class="stat">
              <label>ATK</label>
              <div class="bar">
                <span :style="{ width: pokemon.atk + '%' }"></span>
              </div>
            </div>
            <div class="stat">
              <label>DEF</label>
              <div class="bar">
                <span :style="{ width: pokemon.def + '%' }"></span>
              </div>
            </div>
            <div class="stat">
              <label>SPD</label>
              <div class="bar">
                <span :style="{ width: pokemon.spd + '%' }"></span>
              </div>
            </div>
          </div>
        </div>
      </article>
    </div>

    <div class="actions" v-if="!loading && search_pokemons.length > 0">
      <button class="btn" @click="loadMore" :disabled="loadingMore">
        <span v-if="loadingMore">Loading…</span>
        <span v-else>Load more</span>
      </button>
    </div>
  </div>
</template>

<script>
import AppHeader from './components/AppHeader.vue';
export default {
  components: {
    AppHeader
  },
  name: 'App',
  data() {
    return {
      search: '',
      pokemons: [],
      offset: 0,
      limit: 48,
      loading: true,
      loadingMore: false,
    }
  },
  computed: {
    search_pokemons() {
      let searchWord = this.search.trim()
      if (searchWord === '') {
        return this.pokemons
      }
      return this.pokemons.filter(p =>
        p.name.toLowerCase().includes(searchWord.toLowerCase())
      )
    }
  },
  methods: {
    async pokemonApi(offset, limit) {
      try {
        // ここは fetch 専用（データだけ取得）
        const listUrl = `https://pokeapi.co/api/v2/pokemon?offset=${offset}&limit=${limit}`
        const listRes = await fetch(listUrl)
        
        if (listRes.status !== 200) {
          throw new Error(`List HTTP ${listRes.status}`);
        }
        // 一度APIを叩いて、各ポケモンの詳細URL（エンドポイント一覧）を取得
        const listData = await listRes.json()

        // 各ポケモンの詳細データを更に叩く
        const details = await Promise.all(
          listData.results.map(async (item) => {
            const res = await fetch(item.url)

            if (res.status !== 200) {
              throw new Error(`List HTTP ${listRes.status}`);
            }

            const detail = await res.json()

            const stats = Object.fromEntries(
              detail.stats.map(s => [s.stat.name, s.base_stat])
            )
            const types = detail.types.map(t => t.type.name)

            // 取得したAPIレスポンスを、テンプレートで扱いやすいように整形
            return {
              id: detail.id,
              name: detail.name.charAt(0).toUpperCase() + detail.name.slice(1),
              image:
                detail.sprites?.other?.['official-artwork']?.front_default ??
                detail.sprites?.front_default ??
                '',
              types,
              hp:  stats.hp ?? 50,
              atk: stats.attack ?? 50,
              def: stats.defense ?? 50,
              spd: stats.speed ?? 50,
              spAtk: stats['special-attack'] ?? 50,
              spDef: stats['special-defense'] ?? 50,
            }
          })
        )
        return details
      } catch (error) {
        console.log(error);
      }
      this.loading = false
    },
    async loadInitial() {
      this.loading = true
      try {
        const pokemonDatas = await this.pokemonApi(this.offset, this.limit)
        this.pokemons = pokemonDatas
      } catch (error) {
        console.log(error);
      } 
      this.loading = false
    },
    async loadMore() {
      if(this.loading || this.loadingMore) {
        return
      }
      this.loadingMore = true
      try {
        this.offset = this.offset + this.limit
        const morePokemons = await this.pokemonApi(this.offset, this.limit)
         this.pokemons = [...this.pokemons, ...morePokemons]
      } catch (error) {
        console.log(error);
      } 
      this.loadingMore = false
    }
  },
  mounted() {
    this.loadInitial()
  }
}
</script>

<style>
/* ========== Reset ========== */
* { margin: 0; padding: 0; box-sizing: border-box; }

/* ========== Page Background ========== */
body {
  --bg: #0b0c10;
  background: radial-gradient(1200px 600px at 10% 0%, #0e1322 0%, var(--bg) 60%);
  margin: 0;
  padding: 4px;
}

@media (min-width: 768px) {
  body { padding: 2rem; }
}

/* ========== Loading Spinner ========== */
.spinner-wrap {
  display: grid;
  place-items: center;
  min-height: 60vh;
  animation: fadeIn 0.4s ease;
}

.spinner {
  width: 60px;
  height: 60px;
  border: 4px solid rgba(110, 168, 255, 0.15);
  border-top-color: #6ea8ff;
  border-radius: 50%;
  animation: spin 1s linear infinite, glow 1.5s ease-in-out infinite alternate;
  filter: drop-shadow(0 0 10px rgba(110,168,255,0.4));
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

@keyframes glow {
  from { box-shadow: 0 0 5px rgba(110,168,255,0.3); }
  to { box-shadow: 0 0 20px rgba(110,168,255,0.7); }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}


/* ========== Theme Shell ========== */
.home {
  --panel: #12141a;
  --ring: rgba(110,168,255,.35);
  --text: #e6e8ee;
  --muted: #a9b1c3;
  --gold: #eabe5c;

  min-height: 100vh;
  color: var(--text);
  padding: 32px 16px 72px;
  font-family: ui-sans-serif, system-ui, -apple-system, Segoe UI, Roboto, Noto Sans, 'Helvetica Neue', Arial;
}

/* Limit content width to avoid “packed” feel */
.container {
  width: min(1200px, 100%);
  margin-inline: auto;
  padding-inline: 12px;
}

/* ========== Header / Title ========== */
.header {
  display: grid;
  grid-template-columns: 1fr;
  gap: 8px;
  align-items: center;
  margin-bottom: 32px;
}
.title-wrap {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
}

.title {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  line-height: 1.1;
  font-weight: 800;
  letter-spacing: 1px;
  font-size: 30px;
}

.title-text {
  background: linear-gradient(90deg, #7aa2ff, #eabe5c);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 12px rgba(110,168,255,.25);
}

.poke-icon {
  width: 1.2em;
  height: 1.2em;
  flex: 0 0 auto;
  vertical-align: middle;
  transform: translateY(0.06em);
  opacity: 0.9;
  filter: drop-shadow(0 0 6px rgba(110,168,255,0.15));
}

.subtitle {
  font-size: 13px;
  color: var(--muted);
  letter-spacing: .5px;
  margin-top: 6px;
}

/* ========== Search Row ========== */
/* ========== Search Row (Fixed) ========== */
.search-row {
  position: sticky;
  top: 0;
  z-index: 50;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 0.75rem;
  background: rgba(15, 19, 32, 0.85); /* 背景を半透明にしてぼかす */
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(110, 168, 255, 0.15);
  padding: 12px 16px;
  margin-bottom: 24px;
  min-height: 48px;
  transition: box-shadow 0.3s ease;
}

/* スクロールで影をつけたいとき */
body.scrolled .search-row {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.25);
}

/* 検索バー */
.search {
  flex: 1;
  background: #0f1320;
  color: var(--text);
  border: 1px solid #2a3148;
  padding: 8px 12px;
  border-radius: 10px;
  outline: none;
  max-width: 260px;
  transition: box-shadow 0.2s ease, border-color 0.2s ease;
}
.search:focus {
  box-shadow: 0 0 0 4px var(--ring);
  border-color: var(--ring);
}

/* 結果表示（1件以上のときだけ表示） */
.meta {
  color: var(--muted);
  margin: 0;
  text-align: center;
  min-width: 100px;
  font-size: 0.9rem;
}

/* モバイルでは縦並び */
@media (max-width: 640px) {
  .search-row {
    flex-direction: column;
    align-items: stretch;
    gap: 0.75rem;
    padding: 10px 12px;
  }
  .search {
    width: 100%;
    max-width: none;
  }
  .meta {
    order: -1;
    text-align: center;
    margin-top: 4px;
  }
}

.meta.error { color: #ff6b6b; }

/* ========== No Result (中央表示) ========== */
.no-result {
  display: grid;
  place-items: center;         /* 上下左右の中央 */
  min-height: 60vh;            /* 画面中央に見える高さ確保 */
  color: #ff6b6b;
  font-size: 1.25rem;
  font-weight: 600;
  letter-spacing: 0.5px;
  text-align: center;
  text-shadow: 0 0 10px rgba(255,107,107,0.3);
  animation: fadeIn .4s ease;
}
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

/* ========== Grid ========== */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(15rem, 1fr)); /* 15rem ≒ 240px */
  gap: 1.5rem; /* 24px相当（環境依存） */
}

/* ========== Card (Poké Card style) ========== */
.card {
  position: relative;
  border-radius: 18px;
  padding: 1px;
  background:
    linear-gradient(135deg, #3d4670, #171b25) padding-box,
    linear-gradient(135deg, #7aa2ff, #ffcc66) border-box;
  box-shadow:
    0 10px 30px rgba(0,0,0,.4),
    inset 0 0 0 1px rgba(255,255,255,.06);
  transition: transform .15s ease;
}
.card:hover { transform: translateY(-2px); }

.card::after {
  content: "";
  position: absolute; inset: 0;
  border-radius: 18px;
  background:
    radial-gradient(120px 40px at 30% -10%, rgba(255,255,255,.25), transparent 60%),
    radial-gradient(80px 30px at 80% -10%, rgba(255,255,255,.18), transparent 60%);
  pointer-events: none;
  mix-blend-mode: screen;
}

.card-frame {
  background: var(--panel);
  border-radius: 16px;
  padding: 14px;
}

/* Header inside card */
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 700;
  margin-bottom: 8px;
}
.card-header .name { letter-spacing: .3px; }
.card-header .hp { color: var(--gold); }

/* Image area */
.image-wrap {
  display: grid;
  place-items: center;
  background: radial-gradient(300px 140px at 50% 20%, #1a2030, #0f1320);
  border-radius: 12px;
  padding: 10px;
  margin-bottom: 12px;
  box-shadow: inset 0 0 0 1px rgba(255,255,255,.05);
}
.image-wrap img {
  width: 150px;
  height: 150px;
  object-fit: contain;
  filter: drop-shadow(0 8px 10px rgba(0,0,0,.45));
}

/* Types & id row */
.types {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 10px;
}
.type {
  font-size: 12px;
  text-transform: capitalize;
  padding: 4px 8px;
  border-radius: 999px;
  border: 1px solid rgba(255,255,255,.12);
  background: #161a24;
  color: var(--text);
  box-shadow: inset 0 0 12px rgba(255,255,255,.03);
}
.id { color: var(--muted); font-variant-numeric: tabular-nums; }

/* Type tints (rough) */
.type[data-type="fire"]      { background:#281a1a; border-color:#6d2a2a; }
.type[data-type="water"]     { background:#1a2230; border-color:#2c4970; }
.type[data-type="grass"]     { background:#1a2a20; border-color:#2b6d49; }
.type[data-type="electric"]  { background:#2a261a; border-color:#6d5f2a; }
.type[data-type="psychic"]   { background:#271a30; border-color:#5b2a6d; }
.type[data-type="ice"]       { background:#18242e; border-color:#2a5b6d; }
.type[data-type="fighting"]  { background:#2a221c; border-color:#6d4a2a; }
.type[data-type="poison"]    { background:#241a2a; border-color:#6d2a6d; }
.type[data-type="ground"]    { background:#2a241a; border-color:#6d5a2a; }
.type[data-type="flying"]    { background:#1a202a; border-color:#2a4a6d; }
.type[data-type="rock"]      { background:#252421; border-color:#5a564a; }
.type[data-type="ghost"]     { background:#1e1a2a; border-color:#4a2a6d; }
.type[data-type="dragon"]    { background:#1a2230; border-color:#2a5b6d; }
.type[data-type="dark"]      { background:#15161a; border-color:#3a3b44; }
.type[data-type="steel"]     { background:#1b2024; border-color:#3b4754; }
.type[data-type="fairy"]     { background:#2a1f27; border-color:#6d3a5a; }

/* Stats bars */
.stats { display: grid; gap: 8px; }
.stat {
  display: grid;
  grid-template-columns: 36px 1fr;
  align-items: center;
  gap: 8px;
}
.stat label { color: var(--muted); font-size: 11px; }
.bar {
  height: 8px;
  background: #131722;
  border-radius: 999px;
  position: relative;
  box-shadow: inset 0 0 0 1px rgba(255,255,255,.06);
}
.bar span {
  display: block;
  height: 100%;
  border-radius: 999px;
  background: linear-gradient(90deg, #6ea8ff, #eabe5c);
}

/* Buttons (future use) */
.actions { display: grid; place-items: center; margin-top: 16px; }
.btn {
  background: #0f1320;
  color: var(--text);
  border: 1px solid #2a3148;
  padding: 8px 12px;
  border-radius: 10px;
  cursor: pointer;
}
.btn:disabled { opacity:.5; cursor: not-allowed; }
.btn:hover:not(:disabled) { box-shadow: 0 0 0 4px var(--ring); }

/* ========== Responsive tweaks ========== */
@media (max-width: 640px) {
  .title { font-size: 24px; }
  .grid { grid-template-columns: repeat(auto-fill, minmax(210px, 1fr)); gap: 16px; }
  .search-row { gap: 12px; }
}/* ========== Load More Actions / Button ========== */
.actions {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 48px 0 8px;
  gap: 12px;
  /* セクションをうっすら区切る */
  position: relative;
}
.actions::before {
  content: "";
  position: absolute;
  top: -16px;
  left: 50%;
  transform: translateX(-50%);
  width: min(560px, 90%);
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(122,162,255,.35), transparent);
  filter: drop-shadow(0 0 4px rgba(110,168,255,.25));
}

/* ボタン本体 */
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: .5rem;
  margin-top: .5rem;
  padding: .75rem 1rem;
  min-width: 180px;
  border-radius: 12px;
  border: 1px solid #2a3148;
  background: #0f1320;
  color: var(--text);
  font-weight: 700;
  letter-spacing: .2px;
  line-height: 1;
  cursor: pointer;
  transition: transform .15s ease, box-shadow .2s ease, border-color .2s ease, background .2s ease, opacity .2s ease;
  box-shadow:
    0 8px 24px rgba(0,0,0,.35),
    inset 0 0 0 1px rgba(255,255,255,.04);
}

.btn:hover:not(:disabled) {
  transform: translateY(-1px);
  border-color: #3a4f7a;
  box-shadow:
    0 12px 28px rgba(0,0,0,.45),
    0 0 0 4px var(--ring),
    inset 0 0 0 1px rgba(255,255,255,.06);
}

.btn:active:not(:disabled) {
  transform: translateY(0);
  box-shadow:
    0 6px 16px rgba(0,0,0,.4),
    0 0 0 3px var(--ring);
}

.btn:focus-visible {
  outline: none;
  box-shadow: 0 0 0 4px var(--ring);
}

/* ローディング中（disabled中）はクリック不可＆スピナー表示 */
.btn:disabled {
  opacity: .6;
  cursor: not-allowed;
  transform: none;
  box-shadow: 0 6px 16px rgba(0,0,0,.25);
}

/* :disabled中に左側で回る小型スピナーを出す */
.btn:disabled::before {
  content: "";
  width: 1em;
  height: 1em;
  border-radius: 50%;
  border: 2px solid rgba(110,168,255,.25);
  border-top-color: #6ea8ff;
  display: inline-block;
  animation: spin .8s linear infinite;
}

/* ボタン文言の高さを安定させる */
.btn span {
  display: inline-block;
  transform: translateY(1px);
}

/* モバイルでは全幅にして押しやすく */
@media (max-width: 640px) {
  .actions { margin-top: 20px; }
  .btn { width: 100%; min-width: 0; }
}

/* ユーザーがアニメ抑制設定の場合はアニメ軽減 */
@media (prefers-reduced-motion: reduce) {
  .btn,
  .btn:hover:not(:disabled),
  .btn:active:not(:disabled) {
    transition: none;
  }
  .btn:disabled::before {
    animation: none;
  }
}
</style>

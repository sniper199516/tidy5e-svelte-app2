<script>
  import { onMount } from 'svelte';

  let files = [];
  let importedFiles = [];
  let character = null;
  let selectedFile = '';
  let tab = 'attributes';
  let editMode = false;

  // Server-side file list
  async function fetchFiles() {
    const res = await fetch('/characters/files.json');
    files = await res.json();
  }

  // Server-side file fetch
  async function fetchCharacter(file) {
    const res = await fetch(`/characters/${file}`);
    character = await res.json();
    selectedFile = file;
    tab = 'attributes';
    editMode = false;
  }

  // Import JSON (client only)
  function importCharacterFile(event) {
    const file = event.target.files[0];
    if (!file) return;
    const reader = new FileReader();
    reader.onload = (e) => {
      try {
        const data = JSON.parse(e.target.result);
        // Give a synthetic file name if none
        let filename = file.name.endsWith('.json') ? file.name : `${file.name}.json`;
        importedFiles = [...importedFiles, { name: filename, data }];
        character = data;
        selectedFile = filename;
        tab = 'attributes';
        editMode = false;
        // Optionally, add to the dropdown for switching later
        if (!files.includes(filename)) files = [...files, filename];
      } catch (err) {
        alert("Invalid JSON file.");
      }
    };
    reader.readAsText(file);
  }

  function handleFileChange(e) {
    const filename = e.target.value;
    const imported = importedFiles.find(f => f.name === filename);
    if (imported) {
      character = imported.data;
      selectedFile = filename;
      tab = 'attributes';
      editMode = false;
    } else {
      fetchCharacter(filename);
    }
  }

  function abilityMod(score) {
    return score !== undefined ? Math.floor((score - 10) / 2) : '--';
  }

  function saveEdits() {
    editMode = false;
    alert('Changes saved! (In memory only. To save to disk, ask for export functionality.)');
  }

  // For skills display and editing
  const skillsList = [
    { key: 'acr', label: 'Acrobatics', ability: 'dex' },
    { key: 'ani', label: 'Animal Handling', ability: 'wis' },
    { key: 'arc', label: 'Arcana', ability: 'int' },
    { key: 'ath', label: 'Athletics', ability: 'str' },
    { key: 'dec', label: 'Deception', ability: 'cha' },
    { key: 'his', label: 'History', ability: 'int' },
    { key: 'ins', label: 'Insight', ability: 'wis' },
    { key: 'itm', label: 'Intimidation', ability: 'cha' },
    { key: 'inv', label: 'Investigation', ability: 'int' },
    { key: 'med', label: 'Medicine', ability: 'wis' },
    { key: 'nat', label: 'Nature', ability: 'int' },
    { key: 'prc', label: 'Perception', ability: 'wis' },
    { key: 'prf', label: 'Performance', ability: 'cha' },
    { key: 'per', label: 'Persuasion', ability: 'cha' },
    { key: 'rel', label: 'Religion', ability: 'int' },
    { key: 'slt', label: 'Sleight of Hand', ability: 'dex' },
    { key: 'ste', label: 'Stealth', ability: 'dex' },
    { key: 'sur', label: 'Survival', ability: 'wis' }
  ];

  onMount(fetchFiles);
</script>

<h1 style="color:#f9c74f;font-family:serif;">Tidy5e Svelte (Editable, Import Supported)</h1>

<!-- Import Button -->
<div style="margin-bottom:1em;">
  <label style="color:#f9c74f;cursor:pointer;display:inline-block;">
    <input type="file" accept=".json" style="display:none" on:change={importCharacterFile}>
    <span style="background:#232323;padding:0.4em 1.2em;border-radius:0.7em;font-weight:bold;border:2px solid #f9c74f;">
      Import Character JSON
    </span>
  </label>
</div>

<!-- Dropdown (server and imported files) -->
<div style="margin-bottom:2em;">
  <label style="color:#fff;">Select Character: </label>
  <select on:change={handleFileChange}>
    <option value="">--Choose--</option>
    {#each files as file}
      <option value="{file}" selected={file===selectedFile}>{file.replace('.json','')}</option>
    {/each}
    {#each importedFiles as imp}
      {#if !files.includes(imp.name)}
        <option value="{imp.name}" selected={imp.name===selectedFile}>{imp.name.replace('.json','')}</option>
      {/if}
    {/each}
  </select>
</div>

{#if character}
<form autocomplete="off" on:submit|preventDefault={saveEdits}>
  <div class="tidy5e-sheet allow-edit">

    <!-- Header -->
    <header class="tidy5e-header" style="display:flex;gap:2em;align-items:center;">
      <div class="profile" style="flex:0 0 110px;">
        <div class="portrait">
          <img class="player-image" src="{character.img?.startsWith('http') ? character.img : '/avatar.png'}" alt="{character.name}" style="border-radius:1em;width:100px;height:100px;object-fit:cover;">
        </div>
      </div>
      <div class="character-details" style="flex:1;">
        {#if editMode}
          <input class="char-name" style="margin:0;font-size:2em;width:100%;" bind:value={character.name} />
        {:else}
          <h1 class="char-name" style="margin:0;">{character.name}</h1>
        {/if}
        <div style="display:flex;gap:1em;margin-bottom:.7em;">
          <div>Level {character.system?.details?.level ?? '--'}</div>
          <div>{character.system?.details?.race ?? ''}</div>
          <div>{character.system?.details?.background ?? ''}</div>
          <div>{character.system?.details?.alignment ?? ''}</div>
        </div>
        <div>
          <span>XP: {character.system?.details?.xp?.value ?? 0}</span>
          <span> | Proficiency: +{character.system?.attributes?.prof ?? '--'}</span>
        </div>
        <div>
          <span>Speed: {character.system?.attributes?.movement?.walk ?? '--'} ft</span>
          {#if character.system?.attributes?.movement?.fly}
            <span> | Fly: {character.system.attributes.movement.fly} ft</span>
          {/if}
          {#if character.system?.attributes?.movement?.swim}
            <span> | Swim: {character.system.attributes.movement.swim} ft</span>
          {/if}
          {#if character.system?.attributes?.movement?.climb}
            <span> | Climb: {character.system.attributes.movement.climb} ft</span>
          {/if}
        </div>
        <div style="margin:.8em 0;">
          <span>HP: </span>
          {#if editMode}
            <input style="width:40px;text-align:center;" type="number" min="0" bind:value={character.system.attributes.hp.value} />
          {:else}
            <input readonly style="width:40px;text-align:center;" value={character.system?.attributes?.hp?.value ?? ''} />
          {/if}
          <span> / {character.system?.attributes?.hp?.max ?? ''} </span>
          {#if character.system?.attributes?.hp?.temp}
            <span>(Temp: {character.system.attributes.hp.temp})</span>
          {/if}
        </div>
        <div>
          <span>AC: {character.system?.attributes?.ac?.value ?? '--'}</span>
          <span> | Initiative: {character.system?.attributes?.init?.total ?? '--'}</span>
        </div>
      </div>
      <div>
        <button type="button" on:click={() => editMode = !editMode} style="margin-bottom:.7em;">
          {editMode ? 'Cancel' : 'Edit'}
        </button>
        {#if editMode}
          <button type="submit" style="background:#f9c74f;color:#23221d;">Save</button>
        {/if}
      </div>
    </header>

    <!-- Sheet Tabs -->
    <nav class="tidy5e-navigation tabs" data-group="primary" style="margin:1.3em 0;">
      <a class="item {tab==='attributes' ? 'active' : ''}" on:click|preventDefault={()=>tab='attributes'}>Attributes</a>
      <a class="item {tab==='inventory' ? 'active' : ''}" on:click|preventDefault={()=>tab='inventory'}>Inventory</a>
      <a class="item {tab==='spellbook' ? 'active' : ''}" on:click|preventDefault={()=>tab='spellbook'}>Spellbook</a>
      <a class="item {tab==='features' ? 'active' : ''}" on:click|preventDefault={()=>tab='features'}>Features</a>
      <a class="item {tab==='biography' ? 'active' : ''}" on:click|preventDefault={()=>tab='biography'}>Biography</a>
    </nav>

    <!-- Tab Content -->
    <section class="sheet-body">
      {#if tab==='attributes'}
      <div class="tab attributes" style="display:flex;gap:2em;">
        <!-- Skills List (Sidebar) -->
        <div style="flex:1;min-width:180px;">
          <ul class="skills-list" style="list-style:none;padding:0;">
            {#each skillsList as sk}
              <li style="margin-bottom:.4em;">
                <span style="color:#f9c74f;font-weight:bold;">{sk.label}</span>
                <span style="color:#aaa;font-size:.9em;">({sk.ability.toUpperCase()})</span>
                <span style="margin-left:1em;">
                  {#if character.system?.skills?.[sk.key]}
                    {#if editMode}
                      <input type="number" style="width:36px;text-align:center;" bind:value={character.system.skills[sk.key].mod} />
                    {:else}
                      {(character.system.skills[sk.key].mod > 0 ? '+' : '') + character.system.skills[sk.key].mod}
                    {/if}
                  {:else}
                    --
                  {/if}
                </span>
              </li>
            {/each}
          </ul>
        </div>
        <!-- Ability Scores -->
        <div style="flex:2;">
          <ul class="ability-scores" style="display:flex;gap:2em;list-style:none;padding:0;">
            {#each ['str','dex','con','int','wis','cha'] as ab}
              <li class="ability" style="text-align:center;">
                <h4>{ab.toUpperCase()}</h4>
                {#if editMode}
                  <input type="number" style="width:44px;text-align:center;font-size:1.3em;" min="1" max="30" bind:value={character.system.abilities[ab].value} />
                {:else}
                  <div style="font-size:1.7em;font-weight:bold;">{character.system.abilities[ab]?.value ?? '--'}</div>
                {/if}
                <div style="font-size:1em;color:#f9c74f;">
                  {abilityMod(character.system.abilities[ab]?.value) >= 0 ? '+' : ''}{abilityMod(character.system.abilities[ab]?.value)}
                </div>
              </li>
            {/each}
          </ul>
        </div>
      </div>
      {/if}

      {#if tab==='inventory'}
      <div class="tab inventory">
        <h3>Inventory</h3>
        <ul>
          {#each character.items?.filter(i=>i.type==='equipment' || i.type==='weapon' || i.type==='consumable') as item}
            <li>{item.name} <span style="color:#aaa;">({item.type})</span></li>
          {/each}
        </ul>
      </div>
      {/if}

      {#if tab==='spellbook'}
      <div class="tab spellbook">
        <h3>Spellbook</h3>
        {#each character.items?.filter(i=>i.type==='spell') as spell (spell._id)}
          <div class="spell-block" style="margin-bottom:1em;">
            <strong>{spell.name}</strong>
            <div style="color:#f9c74f;">Level: {spell.system.level}</div>
            <div>{@html spell.system.description?.value ?? ''}</div>
          </div>
        {/each}
      </div>
      {/if}

      {#if tab==='features'}
      <div class="tab features">
        <h3>Features</h3>
        <ul>
          {#each character.items?.filter(i=>i.type==='feat') as feat}
            <li>{feat.name}</li>
          {/each}
        </ul>
      </div>
      {/if}

      {#if tab==='biography'}
      <div class="tab biography">
        <h3>Biography</h3>
        {#if editMode}
          <textarea bind:value={character.system.details.biography.value} style="width:100%;min-height:160px;"></textarea>
        {:else}
          <div>{@html character.system?.details?.biography?.value ?? ''}</div>
        {/if}
      </div>
      {/if}
    </section>
  </div>
</form>
{/if}

<style>
  @import "/style.css";
  .active { color: #f9c74f !important; border-bottom: 2px solid #f9c74f;}
  .tidy5e-navigation.tabs a.item {
    padding: 0.5em 1.2em;
    border-radius: 0.5em 0.5em 0 0;
    text-decoration: none;
    cursor: pointer;
    color: #c4ba6a;
    font-weight: bold;
    background: transparent;
    transition: background .14s;
    border: none;
    outline: none;
  }
  .tidy5e-navigation.tabs a.item.active {
    background: #232323;
    color: #f9c74f;
    border-bottom: 2.5px solid #f9c74f;
  }
</style>

<script>
  import { goto } from "$app/navigation";
  import { onMount } from "svelte";
  export let data;
  let id = data.id;
  let melding = "";
  let notat_tittel = "";
  let notat_innhold = "";
  let suksess = "";
  let notater = "";
  let notat_toggle = false;
  let notat_visning = "";
  let ny_eller_oppdatert = "";
  let notat_id = "";

  async function slett_egen_bruker(id) {
    const svar = confirm(
      `Er du sikker på at du vil slette bruker din egen bruker?`,
    );
    if (svar) {
      const respons = await fetch("http://127.0.0.1:5000/slettbruker", {
        method: "POST",
        body: JSON.stringify({ id }),
        headers: { "Content-Type": "application/json" },
      });
      goto(`/`);
    } else {
      return;
    }
  }
  async function endre_passord(id, nytt_passord) {
    const respons = await fetch("http://127.0.0.1:5000/endrepassord", {
      method: "POST",
      body: JSON.stringify({ id, nytt_passord }),
      headers: { "Content-Type": "application/json" },
    });
    const data = await respons.json();
    melding = data.passordendret;
    if (melding == true) {
      string = "Passord endret";
    } else if (melding == false) {
      string = "Endre til et annet passord enn ditt gamle";
      passord = "";
      return;
    }
  }
  async function nytt_notat() {
    const respons = await fetch("http://127.0.0.1:5000/nyttnotat", {
      method: "POST",
      body: JSON.stringify({ notat_tittel, notat_innhold, id }),
      headers: { "Content-Type": "application/json" },
    });
    notat = false;
    notat_tittel = "";
    notat_innhold = "";
    string = "Notat lagret";
    hent_notater();
  }

  async function oppdater_notat() {
    const respons = await fetch("http://127.0.0.1:5000/oppdaternotat", {
      method: "POST",
      body: JSON.stringify({ notat_tittel, notat_innhold, id, notat_id }),
      headers: { "Content-Type": "application/json" },
    });
    notat_toggle = false;
    hent_notater();
    string = "Endringer lagret";
  }

  function sjekk_lengde() {
    string = "";
    if (passord_lengde > max_pa_lengde || passord_lengde < min_pa_lengde) {
      string = `Hold passordlengde innen 8-24 tegn`;
      return;
    }
    if (!string) {
      endre_passord(id, passord);
    }
  }

  function sjekk_notatlengde(ny_eller_oppdatert) {
    string = "";
    string1 = "";
    if (tittel_lengde < 4 || tittel_lengde > 12) {
      string = `Hold tittel innen 4-12 tegn`;
    }
    if (innhold_lengde < 4 || innhold_lengde > 1200) {
      string1 = `Hold notat innen 4-1200 tegn`;
    }
    if (!string && !string1) {
      if (ny_eller_oppdatert === "") {
        nytt_notat();
      }
      if (ny_eller_oppdatert === "oppdatering") {
        oppdater_notat();
      }
    }
  }

  async function hent_notater() {
    const respons = await fetch("http://127.0.0.1:5000/hentnotater", {
      method: "POST",
      body: JSON.stringify({ id }),
      headers: { "Content-Type": "application/json" },
    });
    const data = await respons.json();
    suksess = data.suksess;
    if (suksess == false) {
      string = "Du har ingen lagrede notater";
    }
    if (suksess == true) {
      notater = data.notater;
    }
  }

  function vis_notat(notat) {
    notat_toggle = true;
    notat_visning = notat;
    notat_innhold = notat[3];
    notat_tittel = notat[2];
    notat_id = notat[0];
    string = "";
    return;
  }

  const max_pa_lengde = 24;
  const min_pa_lengde = 8;
  let string = "";
  let string1 = "";
  let passord = "";
  let passord_text = "";
  $: passord_lengde = passord_text.length;
  $: passord_text = passord;
  $: tittel_lengde = notat_tittel.length;
  $: innhold_lengde = notat_innhold.length;

  let vis_input = false;
  let notat = false;

  function logg_ut() {
    goto(`/`);
  }

  onMount(() => {
    hent_notater();
  });
</script>

<h1>Hei {data.brukernavn}</h1>
<button on:click={() => slett_egen_bruker(id)}>Slett egen bruker</button>
<button on:click={() => (vis_input = !vis_input)}>Endre passord</button>
<button on:click={() => logg_ut()}>Logg ut</button>
<button on:click={() => (notat = !notat)}>Lag nytt notat</button>
{#if vis_input}
  <label for="passord">Nytt passord:</label>
  <input type="password" id="passord" bind:value={passord} />
  <button on:click={() => sjekk_lengde()}>Bekreft</button>
{/if}
{#if string}
  <p class="message">{@html string}</p>
{/if}
{#if string1}
  <p class="message">{@html string1}</p>
{/if}
{#if notat}
  <br />
  <label for="notat_tittel">Tittel:</label>
  <input type="text" id="notat_tittel" bind:value={notat_tittel} />
  <br />
  <label for="notat_tekst">Innhold:</label>
  <textarea class="notat_innhold" bind:value={notat_innhold}></textarea>
  <button on:click={() => sjekk_notatlengde(ny_eller_oppdatert)}>Lagre</button>
{/if}

{#if notat_toggle}
  <table>
    <thead>
      <tr>
        <th>Tittel</th>
        <th>Opprettet</th>
        <th>Sist oppdatert</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>{notat_visning[2]}</td>
        <td>{notat_visning[4]}</td>
        <td>{notat_visning[5]}</td>
      </tr>
    </tbody>
  </table>
  <label for="notat_tittel">Tittel:</label>
  <input type="text" id="notat_tittel" bind:value={notat_tittel} />
  <label for="notat_tekst">Innhold:</label>
  <textarea class="notat_innhold" bind:value={notat_innhold}></textarea>
  <button on:click={() => sjekk_notatlengde("oppdatering")}
    >Lagre endringer</button
  >
{/if}

<h2>Notater</h2>
<div class="notater">
  {#each notater as notat}
    <button class="notat_innhold" on:click={() => vis_notat(notat)}>
      {notat[2]}
    </button>
  {/each}
</div>

<style>
  .notat_innhold {
    height: 50px;
    width: 250px;
    border: 1px solid;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .notater {
    display: flex;
    flex-direction: column;
    gap: 15px;
  }

  th,
  td {
    padding: 10px;
  }
</style>

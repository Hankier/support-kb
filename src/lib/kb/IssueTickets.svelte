<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import tickets from '#lib/data/tickets-4471.json';
  import { clusterReports, uniqueById, type Report } from '#lib/kb/fingerprint';
  import { placeOf } from '#lib/kb/places';

  // One row per issue found in the reports merged into `ticket`, with the ticket it has now.
  let { ticket }: { ticket: number } = $props();

  const rows = reports as Report[];
  const merged = $derived(uniqueById(rows).filter((r) => r.mergedInto === ticket));
  const clusters = $derived(clusterReports(merged, placeOf));
  type Ticket = { number: number; state: string; reason: string; by: string };
  const ticketOf = (fp: string) => (tickets.byFingerprint as Record<string, Ticket>)[fp];
  const url = (n: number) => `https://github.com/${tickets.repo}/issues/${n}`;
  const time = (iso: string) => iso.slice(11, 16);
  const retagged = (c: { place: string; reports: Report[] }) => c.reports.filter((r) => r.product !== c.place).length;
  const first = $derived([...merged].sort((a, b) => a.receivedAt.localeCompare(b.receivedAt))[0]);
</script>

<p class="sum">
  <strong>{rows.length}</strong> rows in the export · <strong>{merged.length}</strong> unique reports ·
  <strong>{clusters.length}</strong> issues · <strong>{clusters.filter((c) => !ticketOf(c.fingerprint)).length}</strong> without a ticket ·
  <strong>{clusters.filter((c) => ticketOf(c.fingerprint)?.state === 'open').length}</strong> open
</p>

<table>
  <thead>
    <tr><th>ticket</th><th>place</th><th>code</th><th>reports</th><th>people</th><th>tagged elsewhere</th><th>seen</th><th>state</th></tr>
  </thead>
  <tbody>
    {#each clusters as c (c.fingerprint)}
      <tr class:missing={!ticketOf(c.fingerprint)}>
        <td>{#if ticketOf(c.fingerprint)}<a href={url(ticketOf(c.fingerprint).number)}>#{ticketOf(c.fingerprint).number}</a>{:else}none{/if}</td>
        <td>{c.place}</td>
        <td><code>{c.code}</code></td>
        <td>{c.reports.length}</td>
        <td>{c.people}</td>
        <td>{retagged(c)}</td>
        <td>{time(c.firstSeen)} → {time(c.lastSeen)}</td>
        <td>{#if ticketOf(c.fingerprint)}{ticketOf(c.fingerprint).state} · {ticketOf(c.fingerprint).reason} · {ticketOf(c.fingerprint).by}{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#if first}
  <p class="first">
    First row merged into #{ticket}: <code>{first.id}</code> at <b>{time(first.receivedAt)}</b> · reporter
    <code>{first.reporter}</code> · source <code>{first.source}</code> · body <code>{first.body}</code>
  </p>
{/if}

<style>
  .sum { font-size: 1.1rem; }
  .missing td { background: var(--warn-bg); color: var(--warn); }
  .first { color: var(--muted); font-size: 0.9rem; }
</style>

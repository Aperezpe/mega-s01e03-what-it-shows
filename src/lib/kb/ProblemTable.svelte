<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // Counted the desk's way (CauseTable): by tenant. One-day moves stay counted where they're filed
  // and get a row of their own, because they are a separate problem.
  type Filed = { title: string; href: string; status: string };
  let { ticket, filed }: { ticket: number; filed: Record<string, Filed> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const rows = $derived([
    ...groups.map((g) => ({ problem: g.cause as string, count: g.reports.length, tenants: g.tenants, seen: `${g.firstSeen} → ${g.lastSeen}`, within: null as string | null })),
    ...groups
      .filter((g) => readAs(g, notes.oneDayMove) > 0)
      .map((g) => {
        const rs = g.reports.filter((r) => notes.oneDayMove.includes(r.id));
        const days = rs.map((r) => r.receivedAt.slice(0, 10)).sort();
        return { problem: 'shift', count: rs.length, tenants: new Set(rs.map((r) => r.tenant)).size, seen: `${days[0]} → ${days[days.length - 1]}`, within: g.cause as string };
      }),
  ]);
</script>

<p class="sum"><strong>{merged.length}</strong> reports merged into #{ticket} · <strong>{rows.length}</strong> problems</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>status</th></tr>
  </thead>
  <tbody>
    {#each rows as r (r.problem)}
      <tr class:open={filed[r.problem]?.status === 'open'}>
        <td><code>{r.problem}</code></td>
        <td>{r.count}{#if r.within}<sup>*</sup>{/if}</td>
        <td>{r.tenants}</td>
        <td>{r.seen}</td>
        <td>{#if filed[r.problem]}<a href={filed[r.problem].href}>{filed[r.problem].title}</a>{:else}not filed{/if}</td>
        <td>{filed[r.problem]?.status ?? '—'}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each rows.filter((r) => r.within) as r (r.problem)}
  <p class="note">* the desk counts these {r.count} under <code>{r.within}</code>, by tenant; they are also in that row.</p>
{/each}

<style>
  .sum { font-size: 1.1rem; }
  .note { color: var(--muted); font-size: 0.85em; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>

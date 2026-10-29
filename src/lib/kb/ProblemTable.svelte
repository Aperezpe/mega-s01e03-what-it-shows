<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { uniqueById, type Report } from '#lib/kb/causes';
  import { groupByProblem, type Problem } from '#lib/kb/problems';
  import { tenantOf } from '#lib/kb/locales';

  type Filed = { title: string; href: string };
  let { ticket, filed }: { ticket: number; filed: Partial<Record<Problem, Filed>> } = $props();

  const rows = $derived(uniqueById(reports as Report[]));
  const merged = $derived(rows.filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByProblem(merged, tenantOf, notes.oneDayMove));
</script>

<p class="sum">
  <strong>{merged.length}</strong> reports merged into #{ticket} ({reports.length} rows in the export, {reports.length - rows.length} repeated) ·
  <strong>{groups.length}</strong> problems
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      <tr class:unfiled={!filed[g.problem]}>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if filed[g.problem]}<a href={filed[g.problem]?.href}>{filed[g.problem]?.title}</a>{:else}not filed{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.1rem; }
  .unfiled td { background: var(--warn-bg); color: var(--warn); }
</style>

<script lang="ts">
  import type { Writable } from "svelte/store";

  import DateRange from "./RangeOfDates.svelte";
  import { getContext } from "svelte";
  import { INPUTS_CONTEXT_KEY } from "@evidence-dev/component-utilities/globalContexts";
  import type { QueryStore } from "@evidence-dev/query-store";
  import { buildQuery } from "@evidence-dev/component-utilities/buildQuery";
  import { getLocalTimeZone } from "@internationalized/date";
  import HiddenInPrint from "../shared/HiddenInPrint.svelte";
  import { page } from "$app/stores";

  function dateToYYYYMMDD(date: Date) {
    return date.toISOString().split("T")[0];
  }

  const inputs: Writable<object> = getContext(INPUTS_CONTEXT_KEY);

  export let name: string;
  export let title: string | undefined;
  export let hideDuringPrint = true;

  // Static API
  export let start: string | Date | undefined;
  export let end: string | Date | undefined;

  // Universal SQL API
  export let data: QueryStore | string | undefined;
  export let dates: string | undefined;

  let query;
  $: if (data && dates) {
    const source = typeof data === "string" ? data : `(${data.text})`;
    query = buildQuery(
      `SELECT min(${dates}) as start, max(${dates}) as end FROM ${source}`,
      `DateRange-${name}`,
      $page.data.data[`DateRange-${name}`]
    );
  }

  const YYYYMMDD = /^\d{4}-\d{2}-\d{2}$/;
  $: calendarStart = typeof start === 'string' && YYYYMMDD.test(start)? start : start instanceof Date? dateToYYYYMMDD(start) : $query?.[0].start instanceof Date? dateToYYYYMMDD($query?.[0].start) : dateToYYYYMMDD(new Date(0));
  $: calendarEnd = typeof end === 'string' && YYYYMMDD.test(end)? end : end instanceof Date? dateToYYYYMMDD(end) : $query?.[0].end instanceof Date? dateToYYYYMMDD($query?.[0].end) : dateToYYYYMMDD(new Date());

  // reactive statements don't run in SSR so we just suck in everything
  $: $inputs[name] = { start: calendarStart, end: calendarEnd };

  let selectedDateRange;
  $: if (
    selectedDateRange &&
    (selectedDateRange.start || selectedDateRange.end)
  ) {
    $inputs[name] = {
      start: dateToYYYYMMDD(selectedDateRange.start?.toDate(getLocalTimeZone()) ?? new Date(0)),
      end: dateToYYYYMMDD(selectedDateRange.end?.toDate(getLocalTimeZone()) ?? new Date()),
    };
  }
</script>

<HiddenInPrint enabled={hideDuringPrint}>
  <div class="mt-2 mb-4 mx-1 inline-block">
    {#if title}
      <span class="text-sm text-gray-500 block">{title}</span>
    {/if}

    {#if $query?.error}
      <span
        class="group inline-flex items-center relative cursor-help cursor-helpfont-sans px-1 border border-red-200 py-[1px] bg-red-50 rounded"
      >
        <span class="inline font-sans font-medium text-xs text-red-600"
          >error</span
        >
        <span
          class="hidden text-white font-sans group-hover:inline absolute -top-1 left-[105%] text-sm z-10 px-2 py-1 bg-gray-800/80 leading-relaxed min-w-[150px] w-max max-w-[400px] rounded-md"
        >
          {$query.error}
        </span>
      </span>
    {:else}
      <DateRange
        bind:selectedDateRange
        start={calendarStart}
        end={calendarEnd}
      />
    {/if}
  </div>
</HiddenInPrint>

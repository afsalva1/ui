<script lang="ts">
  import { page } from '$app/stores';

  import Badge from '$lib/holocene/badge.svelte';
  import Copyable from '$lib/holocene/copyable/index.svelte';
  import Link from '$lib/holocene/link.svelte';
  import { translate } from '$lib/i18n/translate';
  import type { Payloads } from '$lib/types';
  import { format } from '$lib/utilities/format-camel-case';
  import type { CombinedAttributes } from '$lib/utilities/format-event-attributes';
  import {
    shouldDisplayAsExecutionLink,
    shouldDisplayAsTaskQueueLink,
    shouldDisplayChildWorkflowLink,
  } from '$lib/utilities/get-single-attribute-for-event';
  import {
    routeForEventHistory,
    routeForTaskQueue,
  } from '$lib/utilities/route-for';

  import PayloadDecoder from './payload-decoder.svelte';

  export let key: string;
  export let value: string | Record<string, unknown> | Payloads;
  export let attributes: CombinedAttributes;
  export let showKey = true;

  const { workflow, namespace } = $page.params;
</script>

{#if key}
  <div
    class="flex max-w-lg flex-row items-center gap-2 overflow-hidden first:pt-0 last:border-b-0 xl:max-w-xl {$$props.class}"
  >
    {#if showKey}
      <p class="max-w-fit whitespace-nowrap text-right text-xs">
        {format(key)}
      </p>
    {/if}
    {#if typeof value === 'object'}
      <div
        class="flex w-full items-center justify-between gap-2 overflow-hidden pr-1 xl:flex-nowrap"
      >
        <PayloadDecoder {value} key="payloads" let:decodedValue>
          <div class="payload {$$props.class}">
            <code><pre>{decodedValue}</pre></code>
          </div>
        </PayloadDecoder>
      </div>
    {:else if shouldDisplayAsExecutionLink(key)}
      <div class="flex w-full items-center gap-2 pr-1">
        <div class="truncate text-sm">
          <Copyable
            copyIconTitle={translate('common.copy-icon-title')}
            copySuccessIconTitle={translate('common.copy-success-icon-title')}
            content={value}
            container-class="xl:flex-row"
          >
            <Badge type="subtle" class="select-none">
              <Link
                class="truncate"
                href={routeForEventHistory({
                  namespace,
                  workflow,
                  run: value,
                })}
              >
                {value}
              </Link>
            </Badge>
          </Copyable>
        </div>
      </div>
    {:else if shouldDisplayChildWorkflowLink(key, attributes)}
      <div class="flex w-full items-center gap-2 pr-1">
        <div class="truncate text-sm">
          <Copyable
            copyIconTitle={translate('common.copy-icon-title')}
            copySuccessIconTitle={translate('common.copy-success-icon-title')}
            content={value}
            container-class="xl:flex-row"
          >
            <Badge type="subtle" class="select-none">
              <Link
                class="truncate"
                href={routeForEventHistory({
                  namespace,
                  workflow: attributes.workflowExecutionWorkflowId,
                  run: attributes.workflowExecutionRunId,
                })}
              >
                {value}
              </Link>
            </Badge>
          </Copyable>
        </div>
      </div>
    {:else if shouldDisplayAsTaskQueueLink(key)}
      <div class="flex w-full items-center gap-2 pr-1">
        <div class="truncate text-sm">
          <Copyable
            copyIconTitle={translate('common.copy-icon-title')}
            copySuccessIconTitle={translate('common.copy-success-icon-title')}
            content={value}
          >
            <Badge type="subtle" class="select-none">
              <Link
                class="truncate"
                href={routeForTaskQueue({ namespace, queue: value })}
              >
                {value}
              </Link>
            </Badge>
          </Copyable>
        </div>
      </div>
    {:else}
      <div class="flex w-full items-center gap-2 pr-1">
        <p class="truncate text-right text-sm xl:text-left">
          <Badge type="subtle" class="select-none">
            {value}
          </Badge>
        </p>
      </div>
    {/if}
  </div>
{/if}

<style lang="postcss">
  .payload {
    @apply overflow-hidden rounded bg-space-black px-1 py-0.5 font-mono text-xs text-white;
  }
</style>

<script lang="ts">
  import { TabGroup, Tab } from '@skeletonlabs/skeleton';
  import RegionTabPanel from '../components/RegionTabPanel.svelte';

  let regionTab = 0;
</script>

<svelte:head>
  <title>Nostrends</title>
  <meta name="description" content="What's trending on Nostr?" />
  <meta name="keywords" content="nostr,trends,notes,damus,snort" />
  <meta property="og:url" content="https://nostrends.vercel.app" />
  <meta property="og:title" content="Nostrends" />
  <meta property="og:description" content="What's trending on Nostr?" />
</svelte:head>

<h1>Nostrends</h1>

<p>What's trending on <a href="https://nostr.com">Nostr</a>?</p>

<TabGroup>
  <Tab bind:group={regionTab} name="global" value={0}>Global</Tab>
  <Tab bind:group={regionTab} name="jp" value={1}>JP</Tab>

  <svelte:fragment slot="panel">
    {#if regionTab === 0}
      <RegionTabPanel region="global" relayUrl="wss://relay.damus.io" />
    {:else if regionTab === 1}
      <RegionTabPanel region="jp" relayUrl="wss://relay-jp.nostr.wirednet.jp" />
    {/if}
  </svelte:fragment>
</TabGroup>

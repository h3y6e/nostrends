<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import { browser } from '$app/environment';
  import AsyncRelay from '../lib/AsyncRelay';
  import type { Event } from 'nostr-tools';
  import { ReactionCountJsonLoader } from '../lib/ReactionCountJsonLoader';
  import { Note } from '../lib/Note';
  import NoteListItem from '../components/NoteListItem.svelte';
  import LoadingSpinner from '../components/LoadingSpinner.svelte';

  export let relayUrl: string;
  export let region: string;

  const relay = new AsyncRelay(relayUrl);
  let noteEvents: Event[] = [];
  const profileEventByPubkey: { [key: string]: Event } = {};
  const reactionCounts = ReactionCountJsonLoader.loadTopNRank(region, 50);

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  const uniq = (xs: any[]) => Array.from(new Set(xs));

  onMount(async () => {
    if (browser) {
      await relay.connect();
      const noteIds = Object.keys(reactionCounts);
      noteEvents = await relay.sub([{ ids: noteIds }]);
      const pubkeys = uniq(noteEvents.map((note) => note.pubkey));
      const profileEvents = await relay.sub([{ authors: pubkeys, kinds: [0] }]);
      profileEvents.forEach((profile) => (profileEventByPubkey[profile.pubkey] = profile));
    }
  });

  onDestroy(async () => {
    if (browser) {
      await relay.close();
    }
  });

  $: notes = noteEvents.reduce((acc: Note[], note: Event) => {
    if (note.id != null) {
      acc.push(Note.fromEvent(note, profileEventByPubkey[note.pubkey], reactionCounts[note.id]));
    }

    return acc;
  }, []);
</script>

{#each notes as note}
  <div class="my-8">
    {#if note.id}
      <a
        href="https://snort.social/e/{note.nip19Id()}"
        target="_blank"
        rel="noreferrer"
        class="unstyled"
      >
        <NoteListItem {note} />
      </a>
    {:else}
      <NoteListItem {note} />
    {/if}
  </div>
{:else}
  <LoadingSpinner />
{/each}

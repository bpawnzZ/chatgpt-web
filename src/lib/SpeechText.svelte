<script lang="ts">
  // Renders a markdown text node, splitting quoted dialogue out so it can be
  // coloured apart from narration. Handles straight, curly and guillemet quotes.
  export let text: string = ''

  const QUOTE_RE = /("[^"]*"|“[^”]*”|«[^»]*»)/g

  type Part = { text: string; quoted: boolean }

  const splitQuotes = (input: string): Part[] => {
    const parts: Part[] = []
    let last = 0
    for (const match of input.matchAll(QUOTE_RE)) {
      const index = match.index ?? 0
      if (index > last) parts.push({ text: input.slice(last, index), quoted: false })
      parts.push({ text: match[0], quoted: true })
      last = index + match[0].length
    }
    if (last < input.length) parts.push({ text: input.slice(last), quoted: false })
    return parts
  }

  $: parts = splitQuotes(text || '')
</script>

{#each parts as part}{#if part.quoted}<span class="speech">{part.text}</span>{:else}{part.text}{/if}{/each}

<script>
  import { onMount } from 'svelte'

  import Analyser from './Analyser.svelte'

  let lyrics = []
  let isShowCredit = false
  let totalDuration = 4 * 60 - 3
  let currentTime = 0
  let determinedIndex = 0
  let songTitle = 'DJ Snake - A Different Way (feat. Lauv)'

  function constructLyrics(rawLyrics) {
    const parsedLyrics = rawLyrics.split('\n')
    const tsRegex = /\[(\d+):(\d+)]/g

    for (let i = 0; i < parsedLyrics.length; i++) {
      const line = parsedLyrics[i]
      const ts = tsRegex.exec(line)

      if (ts) {
        const text = line.replace(tsRegex, '').trim()

        if (text) {
          const minutes = ts[1] * 60 * 1000
          const seconds = ts[2] * 1000

          lyrics.push({
            ts: minutes + seconds,
            text,
          })
        }
      }
    }
  }

  onMount(() => {
    window
      .fetch('/lirik.txt')
      .then((res) => res.text())
      .then((res) => {
        constructLyrics(res)
      })

    setTimeout(() => {
      document.body.style.opacity = 1
    }, 1000)
  })

  function updateCurrentTime(time) {
    currentTime = Math.floor(time * 1000)

    checkLyrics(currentTime)
  }

  function checkLyrics(time) {
    determinedIndex = lyrics.findIndex(({ ts }) => time <= ts)
  }

  function showCredit() {
    document.getElementsByTagName('canvas')[0].style.filter = 'blur(5px)'

    setTimeout(() => {
      isShowCredit = true
    }, 1000)
  }
</script>

<div>
  <div class="o-lyrics">
    {#if lyrics.length > 0 && determinedIndex !== -1}
      <h4>{lyrics[determinedIndex].text}</h4>
    {/if}

    {#if isShowCredit}
      <p>
        <span>Song by {songTitle}</span>
        <a href="https://blog.umarhadi.xyz">@umarhadi</a>
      </p>
    {/if}
  </div>
  <Analyser
    audio="/entah.m4a"
    {totalDuration}
    {updateCurrentTime}
    {showCredit} />
</div>

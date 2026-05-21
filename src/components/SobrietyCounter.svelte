<script lang="ts">
  import { onMount } from 'svelte';

  let days = 0, hrs = '00', min = '00', sec = '00';
  const start = new Date('2023-11-27T00:00:00').getTime();

  function tick() {
    const diff = Math.floor((Date.now() - start) / 1000);
    days = Math.floor(diff / 86400);
    hrs  = String(Math.floor((diff % 86400) / 3600)).padStart(2, '0');
    min  = String(Math.floor((diff % 3600) / 60)).padStart(2, '0');
    sec  = String(diff % 60).padStart(2, '0');
  }

  onMount(() => {
    tick();
    const id = setInterval(tick, 1000);
    return () => clearInterval(id);
  });
</script>

<div class="counter">
  <div class="label">SOBER SINCE NOV 27, 2023</div>
  <div class="units">
    <div class="unit"><span class="val">{days}</span><span class="key">DAYS</span></div>
    <span class="sep">:</span>
    <div class="unit"><span class="val">{hrs}</span><span class="key">HRS</span></div>
    <span class="sep">:</span>
    <div class="unit"><span class="val">{min}</span><span class="key">MIN</span></div>
    <span class="sep">:</span>
    <div class="unit"><span class="val">{sec}</span><span class="key">SEC</span></div>
  </div>
</div>

<style>
  .counter {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 0.4rem;
    margin: 1.75rem 0;
  }
  .label {
    font-family: var(--font-mono);
    font-size: 0.6rem;
    letter-spacing: 0.2em;
    color: var(--emerald);
    opacity: 0.7;
  }
  .units {
    display: flex;
    align-items: center;
    gap: 0.2rem;
  }
  .unit {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-width: 3.2rem;
  }
  .val {
    font-family: var(--font-mono);
    font-size: 1.65rem;
    font-weight: 700;
    color: var(--emerald);
    line-height: 1;
    text-shadow: 0 0 24px rgba(16,185,129,0.5);
    animation: glowPulse 3s ease-in-out infinite;
  }
  .key {
    font-family: var(--font-mono);
    font-size: 0.48rem;
    letter-spacing: 0.12em;
    color: rgba(16,185,129,0.5);
    margin-top: 0.2rem;
  }
  .sep {
    font-family: var(--font-mono);
    font-size: 1.4rem;
    color: rgba(16,185,129,0.35);
    margin-bottom: 0.9rem;
    padding: 0 0.1rem;
  }
</style>

<script>
  export let currentPath = '/';
  export let isEn = false;

  function getAlternatePath() {
    if (isEn) {
      return currentPath.replace(/^\/en/, '') || '/';
    } else {
      return '/en' + (currentPath === '/' ? '/' : currentPath);
    }
  }
</script>

<a href={getAlternatePath()} class="lang-btn" aria-label={isEn ? 'Ver en español' : 'View in English'}>
  {isEn ? 'ES' : 'EN'}
</a>

<style>
  .lang-btn {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 36px;
    height: 36px;
    border-radius: var(--radius-sm);
    border: 0.5px solid var(--text-border);
    background: transparent;
    color: var(--text-muted);
    font-size: 0.75rem;
    font-weight: 500;
    letter-spacing: 0.05em;
    transition: color 0.15s, background 0.15s;
  }

  .lang-btn:hover {
    color: var(--text);
    background: var(--bg-card);
  }
</style>

<script>
  import ResourceScheduler from './lib/ResourceScheduler.svelte';

  const resources = [
    { id: 1, name: 'Worker 1' },
    { id: 2, name: 'Worker 2' },
    { id: 3, name: 'Worker 3' },
    { id: 4, name: 'Worker 4' },
  ];

  const events = [
    { id: 1, resourceId: 1, title: 'Task 1', start: '2023-04-10T09:00:00', end: '2023-04-10T10:30:00' },
    { id: 2, resourceId: 2, title: 'Meeting', start: '2023-04-10T13:00:00', end: '2023-04-10T14:00:00' },
    { id: 3, resourceId: 3, title: 'Project A', start: '2023-04-10T11:00:00', end: '2023-04-10T15:00:00' },
    { id: 4, resourceId: 4, title: 'Break', start: '2023-04-10T12:00:00', end: '2023-04-10T13:00:00' },
  ];
</script>

<main>
  <ResourceScheduler {resources} {events} />
</main>

<style>
  :global(body) {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
  }

  main {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
  }

  h1 {
    text-align: center;
    margin-bottom: 20px;
  }
</style>
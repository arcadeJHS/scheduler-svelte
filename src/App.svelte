<script>
  import ResourceScheduler from './lib/ResourceScheduler.svelte';
  import ResourceSchedulerFlipped from './lib/ResourceSchedulerFlipped.svelte';

  const resources = [
    { id: 1, name: 'Worker 1' },
    { id: 2, name: 'Worker 2' },
    { id: 3, name: 'Worker 3' },
    { id: 4, name: 'Worker 4' },
  ];

  const events = [
    { id: 1, resourceId: 1, title: 'Task 1', start: '2023-10-16T09:00:00', end: '2023-10-16T10:30:00' },
    { id: 2, resourceId: 2, title: 'Task 2', start: '2023-10-16T13:00:00', end: '2023-10-16T14:00:00' },
    { id: 3, resourceId: 3, title: 'Task 3', start: '2023-10-16T11:00:00', end: '2023-10-16T15:00:00' },
    { id: 4, resourceId: 4, title: 'Task 4', start: '2023-10-16T12:00:00', end: '2023-10-16T13:00:00' },
  ];
</script>

<main>
  <ResourceScheduler {resources} {events} />
</main>

<div class="flipped">
  <ResourceSchedulerFlipped {resources} {events} />
</div>

<style>
  :global(body) {
    font-family: Arial, sans-serif;
    margin: 0;
    padding: 0;
    background: #ffffff;
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

  .flipped {
    max-width: fit-content;
    margin: 0 auto;
  }
</style>
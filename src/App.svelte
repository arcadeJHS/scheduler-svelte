<script>
  import ResourceScheduler from './lib/ResourceScheduler.svelte';
  import ResourceSchedulerFlipped from './lib/ResourceSchedulerFlipped.svelte';

  const resources = [
    { id: 1, name: 'Tecnico 1' },
    { id: 2, name: 'Tecnico 2' },
    { id: 3, name: 'Tecnico 3' },
    { id: 4, name: 'Tecnico 4' },
    { id: 5, name: 'Tecnico 5' },
    { id: 6, name: 'Tecnico 6' }
  ];

  const events = [
    { id: 1, resourceId: 1, title: 'Task 1', start: '2024-10-17T09:00:00', end: '2024-10-17T10:30:00' },
    { id: 2, resourceId: 2, title: 'Task 2 one one one', start: '2024-10-17T13:00:00', end: '2024-10-17T14:00:00' },
    { id: 3, resourceId: 3, title: 'Task 3', start: '2024-10-17T11:00:00', end: '2024-10-17T15:00:00' },
    { id: 4, resourceId: 4, title: 'Task 4', start: '2024-10-17T12:00:00', end: '2024-10-17T13:00:00' },
    { id: 5, resourceId: 5, title: 'Task 5', start: '2024-10-18T12:00:00', end: '2024-10-18T13:00:00' },
    { id: 6, resourceId: 6, title: 'Task 6', start: '2024-10-18T08:00:00', end: '2024-10-18T13:00:00' }
  ];
</script>

<main>
  <ResourceScheduler {resources} {events} />
</main>

<div class="flipped">
  <ResourceSchedulerFlipped {resources} {events} />
</div>

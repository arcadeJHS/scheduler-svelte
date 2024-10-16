<script>
  import { onMount } from 'svelte';

  export let resources = [];
  export let events = [];

  let schedulerElement;
  let currentDate = new Date();
  let draggedEvent = null;

  $: formattedDate = currentDate.toLocaleDateString('en-US', { weekday: 'long', month: 'long', day: 'numeric', year: 'numeric' });

  function renderScheduler() {
    if (!schedulerElement) return;

    const startHour = 7;
    const endHour = 19;
    const intervals = (endHour - startHour) * 2;

    let html = `<div class="scheduler-header">
      <div class="scheduler-time-column">Resource</div>
      ${Array.from({ length: intervals }, (_, i) => {
        const hour = Math.floor(i / 2) + startHour;
        const minute = i % 2 === 0 ? '00' : '30';
        return `<div class="scheduler-time">${hour.toString().padStart(2, '0')}:${minute}</div>`;
      }).join('')}
    </div>`;

    html += '<div class="scheduler-body">';

    resources.forEach(resource => {
      html += `<div class="scheduler-row">
        <div class="scheduler-resource">${resource.name}</div>`;
      for (let i = 0; i < intervals; i++) {
        const hour = Math.floor(i / 2) + startHour;
        const minute = i % 2 === 0 ? '00' : '30';
        html += `<div class="scheduler-cell" data-resource="${resource.id}" data-time="${hour}:${minute}"></div>`;
      }
      html += '</div>';
    });

    html += '</div>';

    schedulerElement.innerHTML = html;

    events.forEach(event => {
      const startDate = new Date(event.start);
      const endDate = new Date(event.end);
      const startTime = startDate.getHours() + startDate.getMinutes() / 60;
      const endTime = endDate.getHours() + endDate.getMinutes() / 60;

      if (startTime >= startHour && startTime < endHour) {
        const cell = schedulerElement.querySelector(`.scheduler-cell[data-resource="${event.resourceId}"][data-time="${Math.floor(startTime)}:${startTime % 1 === 0 ? '00' : '30'}"]`);

        if (cell) {
          const eventElement = document.createElement('div');
          eventElement.className = 'scheduler-event';
          eventElement.style.width = `${(endTime - startTime) * 80}px`;
          eventElement.textContent = event.title;
          eventElement.draggable = true;
          eventElement.setAttribute('data-event-id', event.id);
          eventElement.addEventListener('dragstart', onDragStart);
          cell.appendChild(eventElement);
        }
      }
    });

    schedulerElement.querySelectorAll('.scheduler-cell').forEach(cell => {
      cell.addEventListener('dragover', onDragOver);
      cell.addEventListener('drop', onDrop);
    });
  }

  function onDragStart(e) {
    const eventElement = e.target.closest('.scheduler-event');
    if (eventElement) {
      draggedEvent = events.find(event => event.id === parseInt(eventElement.getAttribute('data-event-id')));
      e.dataTransfer.setData('text/plain', eventElement.getAttribute('data-event-id'));
    }
  }

  function onDragOver(e) {
    e.preventDefault();
  }

  function onDrop(e) {
    e.preventDefault();
    const eventId = parseInt(e.dataTransfer.getData('text'));
    const targetCell = e.target.closest('.scheduler-cell');

    if (targetCell && draggedEvent) {
      const newResourceId = parseInt(targetCell.getAttribute('data-resource'));
      const [newHour, newMinute] = targetCell.getAttribute('data-time').split(':');

      const newStartDate = new Date(draggedEvent.start);
      newStartDate.setHours(parseInt(newHour));
      newStartDate.setMinutes(parseInt(newMinute));

      const duration = new Date(draggedEvent.end).getTime() - new Date(draggedEvent.start).getTime();
      const newEndDate = new Date(newStartDate.getTime() + duration);

      const updatedEvent = {
        ...draggedEvent,
        resourceId: newResourceId,
        start: newStartDate.toISOString(),
        end: newEndDate.toISOString()
      };

      events = events.map(event => event.id === updatedEvent.id ? updatedEvent : event);
      renderScheduler();
    }
  }

  function previousDay() {
    currentDate.setDate(currentDate.getDate() - 1);
    currentDate = currentDate;
    renderScheduler();
  }

  function nextDay() {
    currentDate.setDate(currentDate.getDate() + 1);
    currentDate = currentDate;
    renderScheduler();
  }

  onMount(() => {
    renderScheduler();
  });
</script>

<div class="day-planner">
  <div class="planner-controls">
    <button on:click={previousDay}>&lt;</button>
    <span>{formattedDate}</span>
    <button on:click={nextDay}>&gt;</button>
  </div>
  <div class="planner-container" bind:this={schedulerElement}></div>
</div>

<style>
  .day-planner {
    font-family: Arial, sans-serif;
    margin: 20px;
  }

  .planner-controls {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 10px;
  }

  .planner-controls button {
    margin: 0 5px;
    padding: 5px 10px;
    font-size: 16px;
  }

  .planner-container {
    border: 1px solid #ccc;
    overflow-x: auto;
  }

  :global(.scheduler-header) {
    display: flex;
    border-bottom: 1px solid #ccc;
    position: sticky;
    top: 0;
    background-color: #f0f0f0;
    z-index: 1;
  }

  :global(.scheduler-time-column) {
    width: 60px;
    border-right: 1px solid #ccc;
    font-weight: bold;
    text-align: center;
    padding: 5px;
  }

  :global(.scheduler-resource) {
    flex: 1;
    text-align: center;
    padding: 5px;
    border-right: 1px solid #ccc;
    font-weight: bold;
  }

  :global(.scheduler-body) {
    display: flex;
    flex-direction: column;
  }

  :global(.scheduler-row) {
    display: flex;
    border-bottom: 1px solid #eee;
    height: 40px;
  }

  :global(.scheduler-time) {
    width: 60px;
    padding: 5px;
    border-right: 1px solid #ccc;
    font-size: 12px;
    text-align: right;
  }

  :global(.scheduler-cell) {
    flex: 1;
    border-right: 1px solid #eee;
    position: relative;
  }

  :global(.scheduler-event) {
    position: absolute;
    left: 0;
    right: 0;
    background-color: #3498db;
    color: white;
    padding: 2px;
    font-size: 12px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    cursor: move;
  }
</style>
<script>
  import { onMount } from 'svelte';

  export let resources = [];
  export let events = [];

  let currentDate = new Date(2023, 9, 16); // Use local time
  let draggedEvent = null;
  let resizingEvent = null;
  let resizeDirection = null;
  let initialX = 0;

  const startHour = 7;
  const endHour = 19;
  const intervals = (endHour - startHour) * 2;

  $: formattedDate = currentDate.toLocaleDateString('en-US', { 
    weekday: 'long', 
    month: 'long', 
    day: 'numeric', 
    year: 'numeric'
  });

  $: timeSlots = Array.from({ length: intervals }, (_, i) => {
    const hour = Math.floor(i / 2) + startHour;
    const minute = i % 2 === 0 ? '00' : '30';
    return `${hour.toString().padStart(2, '0')}:${minute}`;
  });

  $: visibleEvents = events
    .filter(event => {
      const eventDate = new Date(event.start);
      return eventDate.getFullYear() === currentDate.getFullYear() &&
             eventDate.getMonth() === currentDate.getMonth() &&
             eventDate.getDate() === currentDate.getDate();
    })
    .map(event => {
      const startDate = new Date(event.start);
      const endDate = new Date(event.end);
      
      const startTime = (startDate.getHours() - startHour) * 2 + startDate.getMinutes() / 30;
      const endTime = (endDate.getHours() - startHour) * 2 + endDate.getMinutes() / 30;

      // Adjust the left position calculation
      const left = startTime * 41; // 41px to account for the 1px border
      const width = (endTime - startTime) * 41 - 1; // Subtract 1px to account for the right border

      return { ...event, left, width };
    });

  function onEventMouseDown(event, e) {
    if (e.target.classList.contains('resize-handle')) {
      e.preventDefault();
      e.stopPropagation();
      resizingEvent = event;
      resizeDirection = e.target.classList.contains('left') ? 'left' : 'right';
      initialX = e.clientX;
      document.addEventListener('mousemove', onResize);
      document.addEventListener('mouseup', onResizeEnd);
    }
  }

  function onResize(e) {
    if (!resizingEvent) return;

    const delta = e.clientX - initialX;
    const cellWidth = 41; // 30 minutes = 41px (including border)
    const timeChange = Math.round(delta / cellWidth) * 30 * 60 * 1000; // Convert to milliseconds

    let newStart = new Date(resizingEvent.start);
    let newEnd = new Date(resizingEvent.end);

    if (resizeDirection === 'left') {
      newStart = new Date(new Date(resizingEvent.start).getTime() + timeChange);
      newStart.setHours(Math.max(newStart.getHours(), startHour), newStart.getMinutes());
    } else {
      newEnd = new Date(new Date(resizingEvent.end).getTime() + timeChange);
      newEnd.setHours(Math.min(newEnd.getHours(), endHour), newEnd.getMinutes());
    }

    // Ensure the event duration is at least 30 minutes
    if (newEnd.getTime() - newStart.getTime() >= 30 * 60 * 1000) {
      const updatedEvent = {
        ...resizingEvent,
        start: newStart.toISOString(),
        end: newEnd.toISOString()
      };
      events = events.map(event => event.id === updatedEvent.id ? updatedEvent : event);
    }
  }

  function onResizeEnd() {
    resizingEvent = null;
    resizeDirection = null;
    document.removeEventListener('mousemove', onResize);
    document.removeEventListener('mouseup', onResizeEnd);
  }

  function onDragStart(event, e) {
    if (e.target.classList.contains('resize-handle')) {
      e.preventDefault();
      return;
    }
    draggedEvent = event;
    e.dataTransfer.setData('text/plain', event.id.toString());
  }

  function onDragOver(e) {
    e.preventDefault();
  }

  function onDrop(resourceId, time, e) {
    e.preventDefault();
    const eventId = parseInt(e.dataTransfer.getData('text'));

    if (draggedEvent && draggedEvent.id === eventId) {
      const [newHour, newMinute] = time.split(':').map(Number);
      
      let newStart = new Date(draggedEvent.start);
      newStart.setHours(newHour, newMinute, 0, 0);

      const duration = new Date(draggedEvent.end).getTime() - new Date(draggedEvent.start).getTime();
      let newEnd = new Date(newStart.getTime() + duration);

      // Ensure the event stays within the valid time range
      if (newStart.getHours() >= startHour && newEnd.getHours() <= endHour) {
        const updatedEvent = {
          ...draggedEvent,
          resourceId,
          start: newStart.toISOString(),
          end: newEnd.toISOString()
        };

        events = events.map(event => event.id === updatedEvent.id ? updatedEvent : event);
      }
    }
  }

  function previousDay() {
    currentDate = new Date(currentDate.getTime() - 24 * 60 * 60 * 1000);
  }

  function nextDay() {
    currentDate = new Date(currentDate.getTime() + 24 * 60 * 60 * 1000);
  }

  let currentTimePosition;

  function getCurrentTimePosition() {
    const now = new Date();
    const currentHour = now.getHours();
    const currentMinute = now.getMinutes();
    const currentTime = currentHour + currentMinute / 60;
    return Math.max(0, (currentTime - startHour) * 82); // 82px for 1 hour (2 cells)
  }

  function updateCurrentTimePosition() {
    currentTimePosition = getCurrentTimePosition();
  }

  onMount(() => {
    updateCurrentTimePosition();
    const interval = setInterval(updateCurrentTimePosition, 60000); // Update every minute
    return () => clearInterval(interval);
  });
</script>

<div class="day-planner">
  <div class="planner-controls">
    <button on:click={previousDay}>&lt;</button>
    <span>{formattedDate}</span>
    <button on:click={nextDay}>&gt;</button>
  </div>
  <div class="planner-container">
    <div class="scheduler-resources">
      <div class="scheduler-resource">Resource</div>
      {#each resources as resource}
        <div class="scheduler-resource">{resource.name}</div>
      {/each}
    </div>
    <div class="scheduler-content">
      <div class="scheduler-header">
        {#each timeSlots as time}
          <div class="scheduler-time">{time}</div>
        {/each}
      </div>
      <div class="scheduler-events">
        {#each resources as resource}
          <div class="scheduler-row">
            {#each timeSlots as time}
              <div 
                class="scheduler-cell"
                on:dragover={onDragOver}
                on:drop={(e) => onDrop(resource.id, time, e)}
              ></div>
            {/each}
            {#each visibleEvents.filter(event => event.resourceId === resource.id) as event (event.id)}
              <div 
                class="scheduler-event"
                style="left: {event.left}px; width: {event.width}px;"
                draggable="true"
                on:mousedown={(e) => onEventMouseDown(event, e)}
                on:dragstart={(e) => onDragStart(event, e)}
              >
                <div class="resize-handle left"></div>
                <div class="event-content">{event.title}</div>
                <div class="resize-handle right"></div>
              </div>
            {/each}
          </div>
        {/each}

        <div class="current-time-indicator" style="left: {currentTimePosition}px;"></div>

      </div>
    </div>
  </div>
</div>

<style>
  .day-planner {
    font-family: Arial, sans-serif;
    margin: 20px;
    color: #000;
  }

  .planner-controls {
    margin-bottom: 0.5rem;
  }

  .planner-container {
    display: flex;
    border: 1px solid #ccc;
    overflow: hidden;
  }

  .scheduler-resources {
    width: 100px;
    border-right: 1px solid #ccc;
    background-color: #f0f0f0;
    z-index: 1;
  }

  .scheduler-resource {
    height: 40px;
    border-bottom: 1px solid #ccc;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
  }

  .scheduler-content {
    flex: 1;
    overflow-x: auto;
  }

  .scheduler-header {
    display: flex;
    border-bottom: 1px solid #ccc;
    position: sticky;
    top: 0;
    background-color: #f0f0f0;
    z-index: 2;
  }

  .scheduler-time {
    width: 40px;
    height: 40px;
    border-right: 1px solid #ccc;
    font-size: 12px;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .scheduler-events {
    position: relative;
  }

  .scheduler-row {
    display: flex;
    border-bottom: 1px solid #eee;
    height: 40px;
    position: relative;
  }

  .scheduler-cell {
    width: 40px;
    height: 40px;
    border-right: 1px solid #eee;
    flex-shrink: 0;
  }

  .scheduler-event {
    position: absolute;
    top: 2px;
    bottom: 2px;
    background-color: #3498db;
    color: white;
    padding: 2px;
    font-size: 12px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    cursor: move;
    display: flex;
    align-items: center;
    border-radius: 4px;
  }

  .event-content {
    flex-grow: 1;
    text-align: center;
  }

  .resize-handle {
    width: 5px;
    height: 100%;
    position: absolute;
    top: 0;
    background-color: rgba(0, 0, 0, 0.2);
    cursor: ew-resize;
  }

  .resize-handle.left {
    left: 0;
  }

  .resize-handle.right {
    right: 0;
  }

  .current-time-indicator {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 2px;
    background-color: red;
    z-index: 3;
    pointer-events: none;
  }

  button {
    border-radius: 4px;
    border: 1px solid blue;
    padding: 0.3em 0.5em;
    font-size: 1em;
    font-weight: 500;
    font-family: inherit;
    background-color: white;
    cursor: pointer;
    transition: border-color 0.25s;
    color: blue;
  }
</style>
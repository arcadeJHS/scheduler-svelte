<script>
  import { onMount } from 'svelte';

  export let resources = [];
  export let events = [];

  let currentDate = new Date(2023, 9, 16); // Use local time
  let draggedEvent = null;
  let resizingEvent = null;
  let resizeDirection = null;
  let initialY = 0;

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

      const top = Math.max(0, startTime * 41); // 30 minutes = 41px (including 1px border)
      const height = Math.max(0, (endTime - startTime) * 41 - 1); // Subtract 1px to account for the bottom border

      return { ...event, top, height };
    });

  function onEventMouseDown(event, e) {
    if (e.target.classList.contains('resize-handle')) {
      e.preventDefault();
      e.stopPropagation();
      resizingEvent = event;
      resizeDirection = e.target.classList.contains('top') ? 'top' : 'bottom';
      initialY = e.clientY;
      document.addEventListener('mousemove', onResize);
      document.addEventListener('mouseup', onResizeEnd);
    }
  }

  function onResize(e) {
    if (!resizingEvent) return;

    const delta = e.clientY - initialY;
    const cellHeight = 41; // 30 minutes = 41px (including 1px border)
    const timeChange = Math.round(delta / cellHeight) * 30 * 60 * 1000; // Convert to milliseconds

    let newStart = new Date(resizingEvent.start);
    let newEnd = new Date(resizingEvent.end);

    if (resizeDirection === 'top') {
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
    <div class="scheduler-times">
      <div class="scheduler-time-header">Time</div>
      {#each timeSlots as time}
        <div class="scheduler-time">{time}</div>
      {/each}
    </div>
    <div class="scheduler-content">
      <div class="scheduler-header">
        {#each resources as resource}
          <div class="scheduler-resource">{resource.name}</div>
        {/each}
      </div>
      <div class="scheduler-events">
        {#each timeSlots as time}
          <div class="scheduler-row">
            {#each resources as resource}
              <div 
                class="scheduler-cell"
                on:dragover={onDragOver}
                on:drop={(e) => onDrop(resource.id, time, e)}
              ></div>
            {/each}
          </div>
        {/each}
        {#each visibleEvents as event (event.id)}
          <div 
            class="scheduler-event"
            style="top: {event.top}px; height: {event.height}px; left: {resources.findIndex(r => r.id === event.resourceId) * 101}px;"
            draggable="true"
            on:mousedown={(e) => onEventMouseDown(event, e)}
            on:dragstart={(e) => onDragStart(event, e)}
          >
            <div class="resize-handle top"></div>
            <div class="event-content">{event.title}</div>
            <div class="resize-handle bottom"></div>
          </div>
        {/each}
        <div class="current-time-indicator" style="top: {currentTimePosition}px;"></div>
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
    overflow: auto;
  }

  .scheduler-times {
    width: 100px;
    border-right: 1px solid #ccc;
    background-color: #f0f0f0;
    z-index: 1;
  }

  .scheduler-time-header, .scheduler-time {
    height: 40px;
    border-bottom: 1px solid #ccc;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
  }

  .scheduler-content {
    flex: 1;
    overflow: auto;
  }

  .scheduler-header {
    display: flex;
    border-bottom: 1px solid #ccc;
    position: sticky;
    top: 0;
    background-color: #f0f0f0;
    z-index: 2;
  }

  .scheduler-resource {
    width: 100px;
    height: 40px;
    border-right: 1px solid #ccc;
    font-size: 12px;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .scheduler-events {
    position: relative;
  }

  .scheduler-row {
    display: flex;
    border-bottom: 1px solid #eee;
    height: 40px;
  }

  .scheduler-cell {
    width: 100px;
    border-right: 1px solid #eee;
    position: relative;
  }

  .scheduler-event {
    position: absolute;
    background-color: #3498db;
    color: white;
    padding: 2px;
    font-size: 12px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
    cursor: move;
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 98px;
    border-radius: 4px;
  }

  .event-content {
    flex-grow: 1;
    text-align: center;
  }

  .resize-handle {
    width: 100%;
    height: 5px;
    position: absolute;
    left: 0;
    background-color: rgba(0, 0, 0, 0.2);
    cursor: ns-resize;
  }

  .resize-handle.top {
    top: 0;
  }

  .resize-handle.bottom {
    bottom: 0;
  }

  .current-time-indicator {
    position: absolute;
    left: 0;
    right: 0;
    height: 2px;
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
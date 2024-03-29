<script lang="ts">
  import { onMount, createEventDispatcher } from 'svelte';
    import type { CourseT } from './types';

  export let selected: number | null;
  export let course: CourseT;
  export let draggingLock = false;
  let { name, credits, courseID } = course;

  const dispatch = createEventDispatcher();
  let isDragging = false;
  let courseTaken = false;
  let offsetX = 0, offsetY = 0;
  let mouseX = 0, mouseY = 0;
	let divElement: HTMLButtonElement | null = null;
  let mouseOver = false;

  $: if(isDragging) {
    emitDrag();
  }

  $: {
    if(divElement) {
      divElement.style.left = `${course.x}px`;
      divElement.style.top = `${course.y}px`;
    }
  }

  onMount(() => {
    document.addEventListener('mousemove', (e) => {
      mouseX = e.x;
      mouseY = e.y;

      if(isDragging) {
        course.x = mouseX - offsetX;
        course.y = mouseY - offsetY;

        handleDragUpdate()
      }
    })

    document.addEventListener('mouseup', () => {
      if (isDragging) {
        isDragging = false;
        handleDragEnd();
      }
    });


    if(divElement) {
      divElement.style.left = `${course.x}px`;
      divElement.style.top = `${course.y}px`;

      divElement.style.width = `${course.w}px`;
      divElement.style.height = `${course.h}px`;
    }

  })

  function handleDragEnd() {
    // Logic to handle the end of a drag, if needed
    // This might include state reset or cleanup
  }

  function handleDragUpdate() {
    dispatch('updatePosition', { courseID, newX: course.x, newY: course.y });
  }

  function emitDrag() {
    dispatch('drag', { name, credits, courseID, x: mouseX - offsetX, y: mouseY - offsetY });
  }
  
  function emitClick() {
    dispatch('click', { name, credits, courseID, x: mouseX - offsetX, y: mouseY - offsetY });
  }

/* Bug with clicking new line when editing content: fix (no new lines) */
</script>

<button
  bind:this={divElement}
  on:focus={() => { /* A11y */ }}
  on:mouseover={() => { mouseOver = true }}
  on:mouseleave={() => { mouseOver = false }}
  on:mouseup={() => { isDragging = false }}
  on:mousedown={(e) => {
    if(divElement && !draggingLock) {
      offsetX = e.x - divElement.offsetLeft;
      offsetY = e.y - divElement.offsetTop;

      isDragging = true;
    }
  }}
  on:click={emitClick}
  class={`flex flex-col items-center justify-center gap-4 border-2 rounded-3xl absolute cursor-grab ${selected === courseID ? 'border-amber-500' : 'border-black'} ${isDragging ? 'cursor-grabbing' : ''}`}
>
  <button 
    on:click={() => courseTaken = !courseTaken} 
    class={`rounded-full w-[25px] h-[25px] ${courseTaken ? 'bg-green-500' : 'bg-gray-300'} absolute top-[-5px] right-[-5px] cursor-pointer`}
  ></button>
  <button 
    on:mousedown={(e) => {
      // todo: handle re-sizing
    }}
    class="cursor-se-resize w-[25px] h-[25px] absolute bottom-[-5px] right-[-5px]"
  ></button>
  <!-- connector buttons -->
  <button 
    on:focus={() => { /* A11y */ }}
    on:blur={() => { /* A11y */ }}
    on:mousedown={(e) => {
      draggingLock = true;
      dispatch('southDown', {e, courseID })
    }}
    on:mousemove={(e) => dispatch('southDrag', {e, courseID })}
    on:mouseover={() => dispatch('southOver', { courseID })}
    on:mouseout={() => dispatch('southOut', { courseID })}
    class={`rounded-full w-[20px] h-[20px] bg-black absolute ${mouseOver || draggingLock ? 'block' : 'hidden'} bottom-[-10px] cursor-crosshair z-50`}
  ></button>

  <!--
  <button 
    on:mousedown={(e) => {
      draggingLock = true;
      dispatch('northDown', e)
    }}
    on:mousemove={(e) => dispatch('northDrag', e)}
    class={`rounded-full w-[20px] h-[20px] bg-black absolute ${mouseOver || draggingLock ? 'block' : 'hidden'} top-[-10px] cursor-crosshair z-50`}
  ></button>

  <button 
    on:mousedown={(e) => {
      draggingLock = true;
      dispatch('westDown', e)
    }}
    on:mousemove={(e) => dispatch('westDrag', e)}
    class={`rounded-full w-[20px] h-[20px] bg-black absolute ${mouseOver || draggingLock ? 'block' : 'hidden'} start-[-10px] cursor-crosshair z-50`}
  ></button>
  
  <button 
    on:mousedown={(e) => {
      draggingLock = true;
      dispatch('eastDown', e)
    }}
    on:mousemove={(e) => dispatch('eastDrag', e)}
    class={`rounded-full w-[20px] h-[20px] bg-black absolute ${mouseOver || draggingLock ? 'block' : 'hidden'} end-[-10px] cursor-crosshair z-50`}
  ></button> -->
  <!-- connector buttons -->
  
  <h1 
    contenteditable
    spellcheck={false} 
    on:input={(e) => { name = e.target.innerText; }} 
    class="text-xl"
  >{name}</h1>
  <h2 
    contenteditable
    spellcheck={false} 
    on:input={(e) => { credits = Number(e.target.innerText); }} 
    class="text-lg"
  >{credits}</h2>
</button>

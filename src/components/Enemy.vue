<template>
    <div
    class="enemy"
    ref="enemyRoot"
    :style="{
              position: 'absolute',
              top: props.y + 'px',
              left: props.x + 'px',
              width: '100px',
              height: '100px',
              pointerEvents: 'none'
            }"
    >
        <img
            :src="goblinSprite"
            :style="{
              position: 'absolute',
              width: '100px',
              height: '100px',
              pointerEvents: 'none'
            }"
            alt="Overlay"
          />
          <div :style="{
              position: 'absolute',
              top: -60 + 'px',
              width: '100px',
              height: '20px',
              pointerEvents: 'none',
              textAlign: 'center',
              fontWeight: 'bold',
              fontSize: '32px',
              color: 'black',
            }">
            {{ props.currentHealth + '/' + props.maxHealth }}
          </div>
          <div
            :style="{
              position: 'absolute',
              top: -30 + 'px',
              width: '100px',
              height: '20px',
              pointerEvents: 'none',
              border: '4px solid red',
              background: 'white',
            }"
            >
            
            <div
            :style="{
              height: '100%',
              width: (100 * props.currentHealth / props.maxHealth) + '%',
              background: 'limegreen',
              transition: 'width 0.3s'
            }"
            >
            </div>
          
          </div>
    </div>
</template>

<script setup>
  import goblinSprite from '@/assets/Goblin_idle.gif'; // adjust path as needed
  import { defineProps } from 'vue';

  const props = defineProps({
    x: {
      type: Number,
      default: "100px"
    },
    y: {
      type: Number,
      default: "300px"
    },
    currentHealth: {
      type: Number,
      default: 100
    },
    maxHealth: {
      type: Number,
      default: 100
    }
  });

</script>
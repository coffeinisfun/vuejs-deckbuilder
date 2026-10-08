<script setup></script>

<template>
  <div style="width: 1000px; margin: 0 auto;">
    <h1>Card Game</h1>

    <div class="bg-container" :style="`background-image: url(${bgImage}); background-size: cover; background-position: center;#
                  width='640';
                  height='480';
                  margin-left: 100px;
                  `">
      <canvas ref="battle_map" 
                  width="640"
                  height="480"
                  style="border: 1px solid black;">
      </canvas> 
      
      <div>
          <Enemy 
                v-for="(enemy, idx) in enemies"
                :key="enemy.id"
                 :x="enemy.x"
                 :y="enemy.y"
                 :currentHealth="enemy.currentHealth"
                 :maxHealth="enemy.maxHealth"
                 :ref="el => setEnemyRef(el, enemy.id)"
                 ></Enemy>
      </div>
    </div> 

    <div>
      <div 
          v-for="(card, index) in cards"
          :key="card.id"
          class="card"
          :style="{
            position: 'absolute',
            left: card.x + 'px',
            top: card.y + 'px',
            zIndex: card.isDragging ? 1000 : 1
          }"
          @mousedown="startDrag(index, $event)">
          {{ card.name }}
      </div>
      <div style="display: flex; align-items: center; width: 100%;">
        <div style="width: 100px; text-align: center;">
          <div class="card">Draw Pile</div> 
        </div>
        <canvas ref="card-table" 
                    width="640"
                    height="240"
                    style="border: 1px solid black;">
          
        </canvas> 
          <!-- Right element -->
        <div style="width: 100px; text-align: center;">
          <div class="card">Discard Pile</div>
        </div>
      </div> 
      <!-- health bar -->
      <div style="width: 400px; height: 20px; border: 2px solid red; background-color: white; margin: 0 auto;">
        <div :style="{ width: (currentHealth / maxHealth * 100) + '%', height: '100%', backgroundColor: 'limegreen' }">
         <div style="margin: auto;" >{{ currentHealth }} / {{ maxHealth }}</div>
        </div>
      </div>
      <!-- energy -->
      <div style="width: 50px; height: 20px; border: 2px solid yellow; background-color: white; margin: 0 auto;">
        <div :style="{ width: '100%', height: '100%', backgroundColor: 'gold',display: 'flex', alignItems: 'center', justifyContent: 'center' }">
         <div style="margin: auto;" >{{ energy }} / 3</div>
        </div>
      </div>
      <!-- shield -->
      <div style="width: 50px; height: 20px; border: 2px solid blue; background-color: white; margin: 0 auto;">
        <div :style="{ width: '100%', height: '100%', backgroundColor: 'lightblue', display: 'flex', alignItems: 'center', justifyContent: 'center'  }">
         <div style="margin: auto;" >{{ shield }}</div>
        </div>
      </div>
      <!-- end turn -->
      <div :style="{ display: 'flex', justifyContent: 'flex-end', width: '100%' }">
        <button @click="endTurn()">End Turn</button>
      </div>  

      
    </div>         
  </div>
</template>

<style scoped></style>

<script setup>
  import Enemy from './components/Enemy.vue'


  import bgImage from '/background.jpg'; // adjust path as needed
</script>

<script>
  export default {
    data() {
      return {
        maxHealth: 100,
        currentHealth: 100,
        shield: 0,
        energy: 3,
        cards: [
          { id: 1, name: 'Strike', x: 600, y: 650, isDragging: false,
            type: 'attack', value: 20, cost: 1
          },
          { id: 2, name: 'Defend', x: 700, y: 650, isDragging: false,
            type: 'defend', value: 20, cost: 1
          },
          { id: 3, name: 'Bash', x: 800, y: 650, isDragging: false,            
            type: 'attack', value: 40, cost: 2
          },
          { id: 4, name: 'Parry', x: 900, y: 650, isDragging: false,
            type: 'defend', value: 40, cost: 2
          },
        ],
        enemies: [
          { id: 1, name: 'Goblin', x: 100, y: 300, currentHealth: 100, maxHealth: 100 },
          { id: 2, name: 'Goblin', x: 250, y: 300, currentHealth: 100, maxHealth: 100 },
          { id: 3, name: 'Goblin', x: 400, y: 300, currentHealth: 100, maxHealth: 100 },
        ],
        enemyRefs: {},
        enemyRects: {},
        dragIndex: null,
        offsetX: 0,
        offsetY: 0,
        enemyTurn: false,
      };
    },
    mounted(){
      this.$nextTick(this.updateEnemyRects);
      window.addEventListener('resize', this.updateEnemyRects);
      this.tidyUpCards(); // Reset card positions on mount
    },
    updated() {
      this.$nextTick(this.updateEnemyRects);
    },
    beforeUnmount() {
      window.removeEventListener('resize', this.updateEnemyRects);
    },
    methods: {
      tidyUpCards() {
        let i = 0;
        for (const card of this.cards) {
          const dist = 640/this.cards.length
          card.x = 620 + i * dist; // Reset x position
          card.y = 600; // Reset y position
          card.orig_x = card.x; // Store original x position
          card.orig_y = card.y; // Store original y position
          i++;
        }
      },
      setEnemyRef(el, id) {
        if (el) {
          this.enemyRefs[id] = el
        } else {
          delete this.enemyRefs[id] // called with null when the element unmounts
        }
      },
      updateEnemyRects(){
        this.enemyRects = {};
        for (const id in this.enemyRefs) {
          const comp = this.enemyRefs[id];
          const el = comp?.$el; // Access the root DOM element of the Enemy component
          if (el) {
            const rect = el.getBoundingClientRect();
            //console.log(`Enemy ${id} rect:`, rect); // Debug log
            this.enemyRects[id] = rect;
          }
        }
      },
      startDrag(index, event) {
        if (this.enemyTurn) {
          console.log("It's the enemy's turn. You cannot play cards.");
          return;
        }
        this.dragIndex = index;
        let card = this.cards[index];
        if (this.energy < card.cost) {
          console.log("Not enough energy to play this card.");
          return;
        }
        this.cards[index].isDragging = true;
        this.offsetX = event.clientX - this.cards[index].x;
        this.offsetY = event.clientY - this.cards[index].y;
        window.addEventListener('mousemove', this.onDrag);
        window.addEventListener('mouseup', this.stopDrag);
      },
      onDrag(event) {
        if (this.dragIndex !== null && this.cards[this.dragIndex]) {
          this.cards[this.dragIndex].x = event.clientX - this.offsetX;
          this.cards[this.dragIndex].y = event.clientY - this.offsetY;
        }
      },
      stopDrag(event) {
        let card = this.cards[this.dragIndex];
        if (this.inBoundsEnemy(event)) {
          if (card.type === 'attack') {
            const enemyId = this.inBoundsEnemy(event);
            console.log(`Card dropped on enemy with ID: ${enemyId}`);


            const enemy = this.enemies.find(e => e.id === parseInt(enemyId));
            if (enemy) {
              enemy.currentHealth -= card.value;
              if (enemy.currentHealth < 0) enemy.currentHealth = 0;
            }

          }
          else if (card.type === 'defend') {
            console.log(`Card dropped on enemy with ID: ${this.inBoundsEnemy(event)}`);
            //increase shield
            this.shield += card.value;
          }
          this.energy -= card.cost;
          //remove the dragged card
          this.cards.splice(this.dragIndex, 1);
          this.tidyUpCards();
        }
        else{
          // Reset card position if not dropped on an enemy 
          this.cards[this.dragIndex].x = this.cards[this.dragIndex].orig_x;
          this.cards[this.dragIndex].y = this.cards[this.dragIndex].orig_y;
        
        }
        if (this.dragIndex !== null  && this.cards[this.dragIndex]) {
          this.cards[this.dragIndex].isDragging = false;
          this.dragIndex = null;
        }
        window.removeEventListener('mousemove', this.onDrag);
        window.removeEventListener('mouseup', this.stopDrag);
      },
      inBoundsEnemy(event) {
        for (const id in this.enemyRects) {
          const rect = this.enemyRects[id];
          if (
            event.clientX >= rect.left &&
            event.clientX <= rect.right &&
            event.clientY >= rect.top &&
            event.clientY <= rect.bottom
          ) {
            return id;
          }
        }
        return null;
      },
      endTurn() {
        // Implement end turn logic here
        console.log("End Turn clicked");
        this.enemyAttacks()
        
      },
      async enemyAttacks() {
          this.enemyTurn = true;
          for (const enemy of this.enemies) {
              if (enemy.currentHealth > 0) {
                  if (this.shield > 0) {
                      const damage = 10; // Example damage value
                      if (this.shield >= damage) {
                          this.shield -= damage;
                      } else {
                          const remainingDamage = damage - this.shield;
                          this.shield = 0;
                          this.currentHealth -= remainingDamage;
                      }
                  } else {
                      this.currentHealth -= 10; // Example damage value
                  }  

                  enemy.y += 10; // Move enemy down to indicate attack
                  await new Promise(resolve => setTimeout(resolve, 100));
                  enemy.y -= 10;
                  await new Promise(resolve => setTimeout(resolve, 600));
              }
          }
          this.enemyTurn = false;
          this.reset()
      },
      reset() {
        this.energy = 3;
        this.shield = 0;
        this.tidyUpCards();
      } 
  }
}
</script>

<style>
.bg-container {
  position: relative;
  width: 640px;
  height: 480px;
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
}
.card {
  width: 100px;
  height: 150px;
  background: #fff;
  border: 1px solid #333;
  cursor: grab;
  user-select: none;
  text-align: center;
  line-height: 60px;
}
</style>
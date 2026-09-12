<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

import gardenRose from './assets/garden-rose.jpg'
import terraceGarden from './assets/terrace-garden.jpg'
import flowerArch from './assets/flower-arch.jpg'
import meadowLake from './assets/meadow-lake.jpg'

/* EDIT YOUR INVITATION TEXT HERE */
const invitation = ref({
  eyebrow: 'MAJLIS MENYULAM JANJI',
  subtitle: '& KASIH',
  bride: 'Ida',
  groom: 'Shah',
  invitationLine: 'JEMPUTAN KE MAJLIS KE PERTUNANGAN KAMI',
  date: '19.09.26',
  time: '9:00 AM - 4:00 PM',
  venue: 'FOREST VALLEY HALL',
  location: 'CHERAS'
})

/* ADD/REPLACE BACKGROUNDS HERE */
const backgrounds = [
  { name: 'Rose Garden', src: gardenRose },
  { name: 'Terrace Garden', src: terraceGarden },
  { name: 'Flower Arch', src: flowerArch },
  { name: 'Meadow Lake', src: meadowLake }
]

const selectedBackground = ref(0)
const showPicker = ref(false)
const audio = ref(null)
const musicOn = ref(false)

const currentBackground = computed(() => backgrounds[selectedBackground.value])

/* HTML/CSS falling leaves: change 22 to control the amount. */
const leaves = Array.from({ length: 22 }, (_, index) => ({
  id: index,
  left: `${(index * 47) % 105 - 3}%`,
  delay: `${-((index * 1.73) % 12)}s`,
  duration: `${8 + ((index * 1.37) % 7)}s`,
  size: `${0.65 + ((index * 0.41) % 0.65)}rem`,
  drift: `${-70 + ((index * 83) % 140)}px`,
  rotate: `${index % 2 ? 25 : -25}deg`,
  type: index % 5
}))

onMounted(() => {
  audio.value = new Audio('./audio/ambient-garden.wav')
  audio.value.loop = true
  audio.value.volume = 0.30
})

onBeforeUnmount(() => audio.value?.pause())

async function toggleMusic() {
  if (!audio.value) return
  if (musicOn.value) {
    audio.value.pause()
    musicOn.value = false
    return
  }
  try {
    await audio.value.play()
    musicOn.value = true
  } catch {
    musicOn.value = false
  }
}

function selectBackground(index) {
  selectedBackground.value = index
  showPicker.value = false
}
</script>

<template>
  <main class="ecard" :style="{ backgroundImage: `url(${currentBackground.src})` }">
    <div class="paper-overlay"></div>

    <div class="falling-leaves" aria-hidden="true">
      <span
        v-for="leaf in leaves"
        :key="leaf.id"
        class="leaf"
        :class="`leaf-type-${leaf.type}`"
        :style="{
          left: leaf.left,
          '--delay': leaf.delay,
          '--duration': leaf.duration,
          '--size': leaf.size,
          '--drift': leaf.drift,
          '--rotate': leaf.rotate
        }"
      ><i></i></span>
    </div>

    <!-- REAL HTML/VUE TEXT: nothing below is baked into the background -->
    <section class="invitation" aria-label="Digital invitation">
      <p class="eyebrow">{{ invitation.eyebrow }}</p>
      <p class="subtitle">{{ invitation.subtitle }}</p>

      <div class="names">
        <div class="script-name">{{ invitation.bride }}</div>
        <div class="ampersand">&amp;</div>
        <div class="script-name">{{ invitation.groom }}</div>
      </div>

      <p class="invitation-line">{{ invitation.invitationLine }}</p>
      <div class="date">{{ invitation.date }}</div>
      <p class="details">{{ invitation.time }}</p>

      <div class="venue">
        <div>{{ invitation.venue }}</div>
        <div>{{ invitation.location }}</div>
      </div>
    </section>

    <div class="controls">
      <button class="control" type="button" @click="toggleMusic">
        <span class="icon">{{ musicOn ? '❚❚' : '♪' }}</span>
        <span>{{ musicOn ? 'Pause' : 'Music' }}</span>
      </button>
      <button class="control" type="button" @click="showPicker = !showPicker">
        <span class="icon">✦</span><span>Theme</span>
      </button>
    </div>

    <aside v-if="showPicker" class="theme-picker">
      <div class="picker-title">Choose a background</div>
      <button
        v-for="(background,index) in backgrounds"
        :key="background.name"
        type="button"
        class="theme-option"
        :class="{ active:index===selectedBackground }"
        @click="selectBackground(index)"
      >
        <img :src="background.src" :alt="background.name">
        <span>{{ background.name }}</span>
        <b v-if="index===selectedBackground">✓</b>
      </button>
    </aside>
  </main>
</template>

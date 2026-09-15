<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref } from "vue";

const currentBackground = ref(2);
const musicPlaying = ref(false);
const showLocation = ref(false);
const hasEntered = ref(false);
const progress = ref(0);

const invitation = {
  heading: "JEMPUTAN",
  subheading: "MAJLIS PERTUNANGAN",
  bride: "Ida",
  groom: "Shah",
  dateDay: "14",
  dateMonth: "NOV.",
  dateYear: "2026",
  day: "Sabtu",
  time: "11:00AM",
  message:
    "Dengan penuh kesyukuran dan rasa rendah diri, kami menjemput kehadiran anda semua ke majlis pertunangan kami.",
  songTitle: "A Thousand Years (Instrumental)",
  locationTitle: "Lokasi Majlis",
  locationName: "Rumah belah perempuan",
  locationAddress: "",
};

// Change this to your real venue.
const locationQuery =
  "Dewan Majlis Seri Anggerik, Jalan Melati 4, 43000 Kajang, Selangor";

const backgrounds = [
  // { name: "Background 1", src: "/backgrounds/background-1.jpg" },
  // { name: "Background 2", src: "/backgrounds/background-2.jpg" },
  { name: "Background 3", src: "/backgrounds/background-3.jpg" },
  { name: "Background 4", src: "/backgrounds/background-4.jpg" },
  { name: "Background 5", src: "/backgrounds/background-5.jpeg" },
  { name: "Background 4", src: "/backgrounds/background-6.jpg" },
];

const petals = Array.from({ length: 34 }, (_, i) => ({
  id: i,
  left: `${(i * 37) % 101}%`,
  delay: `${-((i * 1.73) % 12)}s`,
  duration: `${8 + ((i * 1.37) % 8)}s`,
  size: `${8 + ((i * 11) % 9)}px`,
  drift: `${-100 + ((i * 67) % 201)}px`,
  rotate: `${180 + ((i * 43) % 360)}deg`,
  opacity: 0.42 + ((i * 17) % 45) / 100,
}));

const currentImage = computed(() => backgrounds[currentBackground.value].src);

const mapUrl = computed(
  () =>
    `https://www.google.com/maps?q=${encodeURIComponent(
      locationQuery,
    )}&output=embed`,
);

const mapsUrl = computed(
  () =>
    `https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(
      locationQuery,
    )}`,
);

function changeBackground(index) {
  currentBackground.value = index;
}

/**
 * Start background music.
 *
 * Browsers may block autoplay with sound.
 * Therefore this function is also called on the
 * first user interaction anywhere on the page.
 */
async function startBackgroundMusic() {
  const audio = document.getElementById("backgroundMusic");

  if (!audio || !audio.paused) {
    return;
  }

  try {
    await audio.play();
    musicPlaying.value = true;

    // Once music successfully starts, we no longer
    // need the first-interaction listeners.
    removeFirstInteractionListeners();
  } catch (error) {
    console.log("Autoplay was blocked by the browser.");
    musicPlaying.value = false;
  }
}

async function toggleMusic() {
  const audio = document.getElementById("backgroundMusic");

  if (!audio) return;

  if (audio.paused) {
    try {
      await audio.play();
      musicPlaying.value = true;
      removeFirstInteractionListeners();
    } catch (error) {
      musicPlaying.value = false;

      alert(
        "Sila pastikan fail instrumental.mp3 berada di dalam public/music/",
      );
    }
  } else {
    audio.pause();
    musicPlaying.value = false;
  }
}

function closeLocation() {
  showLocation.value = false;
}

function onTimeUpdate() {
  const audio = document.getElementById("backgroundMusic");

  if (!audio || !audio.duration) return;

  progress.value = (audio.currentTime / audio.duration) * 100;
}

function seek(event) {
  const audio = document.getElementById("backgroundMusic");

  if (!audio || !audio.duration) return;

  const bar = event.currentTarget;
  const rect = bar.getBoundingClientRect();

  const ratio = Math.max(
    0,
    Math.min(1, (event.clientX - rect.left) / rect.width),
  );

  audio.currentTime = ratio * audio.duration;
}

function handleEscape(event) {
  if (event.key === "Escape") {
    closeLocation();
  }
}

/**
 * These events allow the music to start on the visitor's
 * first interaction anywhere on the invitation.
 */
function handleFirstInteraction() {
  startBackgroundMusic();
}

function addFirstInteractionListeners() {
  document.addEventListener("click", handleFirstInteraction);
  document.addEventListener("touchstart", handleFirstInteraction, {
    passive: true,
  });
  document.addEventListener("keydown", handleFirstInteraction);
}

function removeFirstInteractionListeners() {
  document.removeEventListener("click", handleFirstInteraction);
  document.removeEventListener("touchstart", handleFirstInteraction);
  document.removeEventListener("keydown", handleFirstInteraction);
}

onMounted(async () => {
  await nextTick();

  // Text entrance animation
  requestAnimationFrame(() => {
    hasEntered.value = true;
  });

  window.addEventListener("keydown", handleEscape);

  // Try to start music immediately.
  // This may work on some browsers if autoplay is allowed.
  startBackgroundMusic();

  // If autoplay is blocked, the music will start
  // automatically after the visitor's first interaction.
  addFirstInteractionListeners();
});

onUnmounted(() => {
  window.removeEventListener("keydown", handleEscape);
  removeFirstInteractionListeners();
});
</script>

<template>
  <main class="page">
    <section class="invitation-shell" :class="{ 'is-entered': hasEntered }">
      <div
        class="background"
        :style="{ backgroundImage: `url(${currentImage})` }"
      ></div>
      <div class="background-overlay"></div>

      <div class="petal-layer" aria-hidden="true">
        <span
          v-for="petal in petals"
          :key="petal.id"
          class="petal"
          :style="{
            left: petal.left,
            animationDelay: petal.delay,
            animationDuration: petal.duration,
            width: petal.size,
            height: `calc(${petal.size} * 0.62)`,
            '--drift': petal.drift,
            '--rotate': petal.rotate,
            opacity: petal.opacity,
          }"
        ></span>
      </div>

      <div class="background-switcher" aria-label="Pilih latar belakang">
        <button
          v-for="(background, index) in backgrounds"
          :key="background.src"
          class="background-thumb"
          :class="{ active: currentBackground === index }"
          :aria-label="`Pilih ${background.name}`"
          :aria-pressed="currentBackground === index"
          @click="changeBackground(index)"
        >
          <img :src="background.src" :alt="background.name" />
        </button>
      </div>

      <!-- <button
        class="music-fab"
        :class="{ playing: musicPlaying }"
        type="button"
        :aria-label="musicPlaying ? 'Jeda muzik' : 'Mainkan muzik'"
        @click="toggleMusic"
      >♫</button> -->

      <div class="content">
        <div class="entrance entrance-1">
          <p class="eyebrow">{{ invitation.heading }}</p>
          <p class="eyebrow">{{ invitation.subheading }}</p>
        </div>

        <h1 class="names entrance entrance-2">
          <span>{{ invitation.bride }}</span>
          <small>&amp;</small>
          <span>{{ invitation.groom }}</span>
        </h1>

        <div class="date-row entrance entrance-3">
          <div class="date-side">
            <span>{{ invitation.day }}</span
            ><i></i>
          </div>
          <div class="date-main">
            <span class="month">{{ invitation.dateMonth }}</span>
            <strong>{{ invitation.dateDay }}</strong>
            <span>{{ invitation.dateYear }}</span>
          </div>
          <div class="date-side">
            <span>{{ invitation.time }}</span
            ><i></i>
          </div>
        </div>

        <p class="description entrance entrance-4">{{ invitation.message }}</p>

        <div class="location-hint entrance entrance-5" aria-hidden="true">
          <span class="hint-arrow">⌄</span> Klik untuk lihat lokasi
        </div>
        <button
          class="location-button entrance entrance-5"
          type="button"
          @click="showLocation = true"
        >
          <span class="pin">
            <svg viewBox="0 0 24 24">
              <path
                d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5a2.5 2.5 0 1 1 0-5 2.5 2.5 0 0 1 0 5z"
              />
            </svg>
          </span>
          <span class="location-copy">{{ invitation.locationName }}</span>
          <span class="arrow" aria-hidden="true">
            <svg viewBox="0 0 24 24">
              <path
                d="M9 6l6 6-6 6"
                fill="none"
                stroke="currentColor"
                stroke-width="2.5"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
          </span>
        </button>

        <div class="music-player entrance entrance-7">
          <!-- <div class="song-title">
            <span class="music-note">♫</span>
            <span>{{ invitation.songTitle }}</span>
          </div>
          <div class="player-line" @click="seek">
            <span :style="{ width: progress + '%' }"></span>
          </div> -->
          <div class="player-controls">
            <button type="button" aria-label="Sebelumnya">‹</button>
            <button class="play-button" type="button" @click="toggleMusic">
              {{ musicPlaying ? "Ⅱ" : "▶" }}
            </button>
            <button type="button" aria-label="Seterusnya">›</button>
          </div>
        </div>
      </div>

      <audio
        id="backgroundMusic"
        src="/music/instrumental.mp3"
        preload="auto"
        loop
      ></audio>
    </section>

    <Transition name="modal">
      <div
        v-if="showLocation"
        class="modal-backdrop"
        @click.self="closeLocation"
      >
        <section class="location-modal" role="dialog" aria-modal="true">
          <button class="close-button" type="button" @click="closeLocation">
            ×
          </button>
          <p class="modal-eyebrow">LOKASI</p>
          <h2>{{ invitation.locationTitle }}</h2>
          <iframe
            class="map"
            :src="mapUrl"
            title="Peta lokasi majlis"
            loading="lazy"
            referrerpolicy="no-referrer-when-downgrade"
          ></iframe>
          <div class="location-copy">
            <strong>{{ invitation.locationName }}</strong>
            <span>{{ invitation.locationAddress }}</span>
          </div>
          <a
            class="maps-link"
            :href="mapsUrl"
            target="_blank"
            rel="noopener noreferrer"
          >
            ⌖ &nbsp; Buka di Google Maps
          </a>
        </section>
      </div>
    </Transition>
  </main>
</template>

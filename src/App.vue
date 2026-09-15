<script setup>
import { computed, nextTick, onMounted, onUnmounted, ref } from "vue";

const currentBackground = ref(6);
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
  locationName: "Paka, Terengganu",
  locationAddress: "",
};

// Change this to your real venue.
const locationQuery = "Paka, Terengganu, Malaysia";

const backgroundModules = import.meta.glob(
  "/public/backgrounds/background-*.*",
  { eager: true, query: "?url", import: "default" },
);

const backgrounds = Object.entries(backgroundModules)
  .map(([path]) => {
    const file = path.split("/").pop();
    const index = file.match(/(\d+)/)?.[1] ?? "0";
    return { name: `Background ${index}`, src: `/backgrounds/${file}` };
  })
  .sort((a, b) => {
    const numA = parseInt(a.name.replace(/\D/g, ""), 10);
    const numB = parseInt(b.name.replace(/\D/g, ""), 10);
    return numA - numB;
  });

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

const wazeUrl = computed(
  () =>
    `https://waze.com/ul?q=${encodeURIComponent(locationQuery)}&navigate=yes`,
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

      <!-- <div class="background-switcher" aria-label="Pilih latar belakang">
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
      </div> -->

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
            <!-- <button type="button" aria-label="Sebelumnya">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M6 6h2v12H6zm3.5 6 8.5 6V6z" />
              </svg>
            </button> -->
            <button class="play-button" type="button" @click="toggleMusic">
              <svg
                v-if="musicPlaying"
                class="pause-icon"
                viewBox="0 0 24 24"
                aria-hidden="true"
              >
                <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" />
              </svg>
              <svg
                v-else
                class="play-icon"
                viewBox="0 0 24 24"
                aria-hidden="true"
              >
                <path d="M8 5v14l11-7z" />
              </svg>
            </button>
            <!-- <button type="button" aria-label="Seterusnya">
              <svg viewBox="0 0 24 24" aria-hidden="true">
                <path d="M6 6l8.5 6L6 18zM16 6h2v12h-2z" />
              </svg>
            </button> -->
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
          <!-- <p class="modal-eyebrow">LOKASI</p> -->
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
<div class="maps-row">
            <a
              class="maps-link maps-link-maps"
              :href="mapsUrl"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Buka di Google Maps"
            >
              <svg class="maps-icon" viewBox="0 0 24 24" aria-hidden="true">
                <path
                  d="M19.527 4.799c1.212 2.608.937 5.678-.405 8.173-1.101 2.047-2.744 3.74-4.098 5.614-.619.858-1.244 1.75-1.669 2.727-.141.325-.263.658-.383.992-.121.333-.224.673-.34 1.008-.109.314-.236.684-.627.687h-.007c-.466-.001-.579-.53-.695-.887-.284-.874-.581-1.713-1.019-2.525-.51-.944-1.145-1.817-1.79-2.671L19.527 4.799zM8.545 7.705l-3.959 4.707c.724 1.54 1.821 2.863 2.871 4.18.247.31.494.622.737.936l4.984-5.925-.029.01c-1.741.601-3.691-.291-4.392-1.987a3.377 3.377 0 0 1-.209-.716c-.063-.437-.077-.761-.004-1.198l.001-.007zM5.492 3.149l-.003.004c-1.947 2.466-2.281 5.88-1.117 8.77l4.785-5.689-.058-.05-3.607-3.035zM14.661.436l-3.838 4.563a.295.295 0 0 1 .027-.01c1.6-.551 3.403.15 4.22 1.626.176.319.323.683.377 1.045.068.446.085.773.012 1.22l-.003.016 3.836-4.561A8.382 8.382 0 0 0 14.67.439l-.009-.003zM9.466 5.868L14.162.285l-.047-.012A8.31 8.31 0 0 0 11.986 0a8.439 8.439 0 0 0-6.169 2.766l-.016.018 3.665 3.084z"
                />
              </svg>
            </a>
            <a
              class="maps-link maps-link-waze"
              :href="wazeUrl"
              target="_blank"
              rel="noopener noreferrer"
              aria-label="Buka di Waze"
            >
              <svg class="maps-icon" viewBox="0 0 24 24" aria-hidden="true">
                <path
                  d="M13.218 0C9.915 0 6.835 1.49 4.723 4.148c-1.515 1.913-2.31 4.272-2.31 6.706v1.739c0 .894-.62 1.738-1.862 1.813-.298.025-.547.224-.547.522-.05.82.82 2.31 2.012 3.502.82.844 1.788 1.515 2.832 2.036a3 3 0 0 0 2.955 3.528 2.966 2.966 0 0 0 2.931-2.385h2.509c.323 1.689 2.086 2.856 3.974 2.21 1.64-.546 2.36-2.409 1.763-3.924a12.84 12.84 0 0 0 1.838-1.465 10.73 10.73 0 0 0 3.18-7.65c0-2.882-1.118-5.589-3.155-7.625A10.899 10.899 0 0 0 13.218 0zm0 1.217c2.558 0 4.967.994 6.78 2.807a9.525 9.525 0 0 1 2.807 6.78A9.526 9.526 0 0 1 20 17.585a9.647 9.647 0 0 1-6.78 2.807h-2.46a3.008 3.008 0 0 0-2.93-2.41 3.03 3.03 0 0 0-2.534 1.367v.024a8.945 8.945 0 0 1-2.41-1.788c-.844-.844-1.316-1.614-1.515-2.11a2.858 2.858 0 0 0 1.441-.846 2.959 2.959 0 0 0 .795-2.036v-1.789c0-2.11.696-4.197 2.012-5.861 1.863-2.385 4.62-3.726 7.6-3.726zm-2.41 5.986a1.192 1.192 0 0 0-1.191 1.192 1.192 1.192 0 0 0 1.192 1.193A1.192 1.192 0 0 0 12 8.395a1.192 1.192 0 0 0-1.192-1.192zm7.204 0a1.192 1.192 0 0 0-1.192 1.192 1.192 1.192 0 0 0 1.192 1.193 1.192 1.192 0 0 0 1.192-1.193 1.192 1.192 0 0 0-1.192-1.192zm-7.377 4.769a.596.596 0 0 0-.546.845 4.813 4.813 0 0 0 4.346 2.757 4.77 4.77 0 0 0 4.347-2.757.596.596 0 0 0-.547-.845h-.025a.561.561 0 0 0-.521.348 3.59 3.59 0 0 1-3.254 2.061 3.591 3.591 0 0 1-3.254-2.061.64.64 0 0 0-.546-.348z"
                />
              </svg>
            </a>
          </div>
        </section>
      </div>
    </Transition>
  </main>
</template>

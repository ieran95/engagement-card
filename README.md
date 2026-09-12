# Fina & Syafiq Vue/Vite E-card

## Edit text
Edit the `invitation` object at the top of `src/App.vue`. All invitation text is real HTML/Vue text and is NOT baked into the backgrounds.

## Backgrounds
Four separate text-free watercolor JPGs are in `src/assets/`. Change the `backgrounds` array in `src/App.vue` to add or replace themes.

## Falling leaves
The leaves are HTML elements animated with CSS. Change `Array.from({ length: 22 })` in `src/App.vue` to control quantity. Animation is in `src/style.css` under `@keyframes leaf-fall` and `@keyframes leaf-sway`.

## Music
Replace `public/audio/ambient-garden.wav` with your own track if desired. Browsers usually require a user click before audio can start.

## Run
`npm install`
`npm run dev`

## GitHub Pages
Push to `main`, enable GitHub Actions as the Pages source in repository Settings > Pages, and the included workflow will deploy it.

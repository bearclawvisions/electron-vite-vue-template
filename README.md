# Vue 3 + TypeScript + Vite

Install with `npm electron-vite@latest <project>`

## Install prettier
Install prettier `npm install --save-dev prettier`

```json
// in .prettierrc, might need extra setup via (Rider) File → Settings → Languages & Frameworks → JavaScript → Prettier
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5"
}
```

## Install Tailwindcss
We install tailwindcss with postcss and autoprefixer. Autoprefixer automatically adds vendor prefixes to CSS properties that need them for cross-browser compatibility.

`npm install tailwindcss @tailwindcss/postcss postcss autoprefixer`

Add a file named `postcss.config.mjs` at the root of the project and add the below:
```js
export default {
   plugins: {
      "@tailwindcss/postcss": {},
      autoprefixer: {},
   }
}
```

Add this line to the main css file: `@import "tailwindcss";`

## Install Vue-devtools
Follow instructions according to this website: https://devtools.vuejs.org/guide/standalone
Standalone as dependency injection. No additional config needed for this part.

Now also add the Vue devtools to the Electron builder.
Follow the install instructions from: https://github.com/MarshallOfSound/electron-devtools-installer

Import the VUEJS_DEVTOOLS and installExtension() during startup in Electron's `main.ts`:
```js
import installExtension, { VUEJS_DEVTOOLS } from 'electron-devtools-installer'


function createWindow() {
   win = new BrowserWindow({
      icon: path.join(process.env.VITE_PUBLIC, 'electron-vite.svg'),
      webPreferences: {
         preload: path.join(__dirname, 'preload.mjs'),
         nodeIntegration: true,
         contextIsolation: false, // this
         devTools: true, // this
      },
   })

   // Test active push message to Renderer-process.
   win.webContents.on('did-finish-load', () => {
      win?.webContents.send('main-process-message', (new Date).toLocaleString())
   })

   if (VITE_DEV_SERVER_URL) {
      win.loadURL(VITE_DEV_SERVER_URL)
   } else {
      // win.loadFile('dist/index.html')
      win.loadFile(path.join(RENDERER_DIST, 'index.html'))
   }

   // Install Vue Devtools
   installExtension(VUEJS_DEVTOOLS)
           .then((name) => console.log(`Added Extension:  ${name}`))
           .catch((err) => console.log('An error occurred: ', err));

   // below for chrome devtools
   // win.webContents.openDevTools()
}
```

In the `main.ts` from the Vue application add the devtools like below:
```js
import { createApp } from 'vue';
import { devtools } from '@vue/devtools'; // this
import './style.css';
import App from './App.vue';

const app = createApp(App);

app.mount('#app').$nextTick(() => {
  // Use contextBridge
  window.ipcRenderer.on('main-process-message', (_event, message) => {
    console.log(message);
  });
});

// this adds the devtools as DI
if (process.env.NODE_ENV === 'development') {
  devtools.connect(/* host (the default is "http://localhost"), port (the default is 8090) */);
}

```

Also update the package.json to be able to run the devtools. This needs to be run before the application itself runs.
```json
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build && electron-builder",
    "preview": "vite preview",
    "devtools": "./node_modules/.bin/vue-devtools" // this
  },
```

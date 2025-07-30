# Vue 3 + TypeScript + Vite

This template should help get you started developing with Vue 3 and TypeScript in Vite. The template uses Vue 3 `<script setup>` SFCs, check out the [script setup docs](https://v3.vuejs.org/api/sfc-script-setup.html#sfc-script-setup) to learn more.

## Type Support For `.vue` Imports in TS

TypeScript cannot handle type information for `.vue` imports by default, so we replace the `tsc` CLI with `vue-tsc` for type checking. In editors, we need [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin) to make the TypeScript language service aware of `.vue` types.

If the standalone TypeScript plugin doesn't feel fast enough to you, Volar has also implemented a [Take Over Mode](https://github.com/johnsoncodehk/volar/discussions/471#discussioncomment-1361669) that is more performant. You can enable it by the following steps:

1. Disable the built-in TypeScript Extension
   1. Run `Extensions: Show Built-in Extensions` from VSCode's command palette
   2. Find `TypeScript and JavaScript Language Features`, right click and select `Disable (Workspace)`
2. Reload the VSCode window by running `Developer: Reload Window` from the command palette.

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
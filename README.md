# Zoom Zoom

A skeleton project for Zoom Zoom. Uses Svelte (not sveltekit) to generate a single index.html file.

Can be tested with https://github.com/Stengo/DeskPad (hopefully?)

**NOTE:** The place where I found the Svelte PresentationAPI code is https://github.com/search?q=presentationConnection+svelte+canCast&type=code (might not be very good though).

## Process

Here's my current thinking:

1. The web part of the app gets built in Svelte + Vite (TypeScript).
  - This would work great as a standalone html file, except `new PresentationRequest(url)` needs a valid url for the "receiver", see [this](https://googlechrome.github.io/samples/presentation-api/) and [this thread](https://www.google.com/search?q=Failed+to+construct+%27PresentationRequest%27%3A+An+empty+sequence+of+URLs+is+not+supported.&sourceid=chrome&ie=UTF-8)
  - Here's the [viewer.html](https://github.com/tomoyukilabs/presentation-api-demo/blob/d04aba7c3024187ba0ede2dc5b0f0731be19a315/viewer.html) example which is what would have to be served in the `url` in `new PresentationRequest(url)`.
2. That means we need some kind of server. I tried two options:
  1. Creating a Go server works surprisingly well, see my conversation with Claude. HOWEVER, it would need https://github.com/progrium/darwinkit to run the app's lifecycle properly if I wanna bundle into a an app bundle, like [this](https://medium.com/@mattholt/packaging-a-go-application-for-macos-f7084b00f6b5)
  2. There is `py2app` for python too, and that *should* work except when I tried it it was a horrible mess of dependencies and lots of horrible stuff. It's way heavier than the go server above.

## TODO

- [ ] Create the server such that it just exposes the index.html (generated from this Svelte project) and the viewer (with any needed JS and stuff there, see [viewer.html](https://github.com/tomoyukilabs/presentation-api-demo/blob/d04aba7c3024187ba0ede2dc5b0f0731be19a315/viewer.html))
- [ ] Modify the Go example from https://github.com/progrium/darwinkit to run a simple server, and try to bundle that into a .app. It should work
- [ ] Test that once that is working, the Presentation API actually works as intended.
- [ ] Finish all the other stuff! (The actual UI)

# Notes on Christian's requirements

- [ ] Alphabetic order to preserve some kind of muscle memory. Rearranging the images isn't good
- [ ] "Clicking" should only be used for changing the image.
- [ ] Scrolling with the mouse takes too long.
- [ ] Don't dim the images.
- [ ] There will be an archivist looking at this repo. Make their life easy
- [ ] My idea for the above is to host a version of this on GitHub pages (a publicly accessible one that he can use)  but also to give them a bundled executable which should work offline and under all other circumstances.

## Useful commands

This repo was created with:
```
pnpm create vite@latest 
pnpm install vite-plugin-singlefile --save-dev
```

Build Svelte app:
```
pnpm i
pnpm run build
```
Building Go app binary into dist folder:
```
GOOS=darwin GOARCH=amd64 go build -o dist/zoomzoom server.go  
```

Buidling if if using https://github.com/progrium/darwinkit:
```
GOOS=darwin go build server.go
```

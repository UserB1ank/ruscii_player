# ruscii_player

English · **[中文](./README.md)**

---

A terminal-based ASCII player written in Rust. It converts **images** and **videos** into true-color ASCII art and plays them directly in your terminal.

## ✨ Features

- 🎬 **Video playback** — Decodes video streams via `ffmpeg-next` and renders each frame into an ASCII animation.
- 🖼️ **Image conversion** — Turns common image formats (PNG / JPG / ...) into ASCII art printed to stdout.
- 🌈 **24-bit true color** — Each ASCII character keeps its original pixel's RGB color via ANSI true-color escapes.
- 📐 **Auto-fit terminal** — Scales to your terminal's columns/rows and compensates for the ~2:1 cell aspect ratio, so portrait videos won't overflow.
- 🧱 **Modular design** — `terminal` / `image` / `ffmpeg` are isolated for easy extension.

## 📦 Dependencies

| crate | Purpose |
|-------|---------|
| [`ffmpeg-next`](https://crates.io/crates/ffmpeg-next) | Video decoding |
| [`image`](https://crates.io/crates/image) | Image decoding & resizing |
| [`crossterm`](https://crates.io/crates/crossterm) | Terminal control (hide cursor, clear, position) |
| [`term_size`](https://crates.io/crates/term_size) | Read terminal dimensions |
| [`clap`](https://crates.io/crates/clap) | CLI argument parsing |

> ⚠️ Video support requires system-installed **FFmpeg** libraries (libavcodec / libavformat / libavutil / libavdevice / libavfilter / libswscale / libswresample). See the [ffmpeg-next install guide](https://crates.io/crates/ffmpeg-next#installing-ffmpeg).

## 🚀 Installation

```bash
git clone https://github.com/UserB1ank/ruscii_player.git
cd ruscii_player
cargo build --release
```

The binary is built at `target/release/ruscii_player`.

## 🎮 Usage

`-v/--video-path` and `-i/--img-path` are **mutually exclusive** — exactly one is required.

### Render an image as ASCII

```bash
# Prints to stdout (can be redirected to a file)
cargo run --release -- -i path/to/image.png
./target/release/ruscii_player -i photo.jpg
```

### Play a video in the terminal

```bash
cargo run --release -- -v path/to/video.mp4
./target/release/ruscii_player -v clip.mp4
```

The video switches to crossterm's alternate screen with the cursor hidden and prints ASCII frame by frame; the original terminal is restored when playback ends.

### Show help

```bash
./target/release/ruscii_player --help
```

```
Options:
  -v, --video-path <VIDEO_PATH>  the path of video which will be converted to ASCII
  -i, --img-path <IMG_PATH>      the path of image which will be converted to ASCII
  -h, --help                     Print help
```

## 🧩 How it works

1. **Size fitting** (`terminal::get_term_size`) — Reads the terminal's columns/rows, then derives the target resolution from the source aspect ratio plus the cell aspect ratio (default `0.5`). Portrait videos switch to filling by height to avoid row overflow.
2. **Decode & scale** (`ffmpeg::render`) — Uses ffmpeg's `software::scaling` to scale each frame straight to the target resolution in RGB24, skipping a second resize pass.
3. **Luma mapping** (`image::render_img`) — Maps each pixel's luma to a density-ordered ASCII ramp:

   ```text
   "WM$@%&#NB8E9GAmK6w5HRkbYT43V0JL7gpaesyxznocv?Ijftr1li|*=-~^`':;,. "
   ```

   Darker pixels → denser characters, each colored with `\x1b[38;2;R;G;Bm` from its original pixel.

## 🗂️ Project layout

```
src/
├── main.rs            # clap arg parsing, dispatches to image / video
├── lib.rs             # module re-exports
├── terminal/
│   ├── size.rs        # get_term_size: terminal size + aspect fit
│   └── term.rs
├── image/
│   └── render.rs      # render_img: pixel → colored ASCII string
└── ffmpeg/
    └── render.rs      # render_video: video decoding + live playback
```

## ⚠️ Known limitations

- Video frame rate is bounded by decode and terminal-refresh speed; high-res / high-FPS videos may drop frames.
- The terminal must support 24-bit true color for correct coloring (Windows Terminal / iTerm2 / modern terminals all qualify).
- No keyboard controls yet (pause / quit, etc.); use `Ctrl+C` to force exit.

## 📄 License

[MIT](./LICENSE) © Neo

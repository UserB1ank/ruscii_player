# ruscii_player

**[English](./README.en.md)** · 中文

---

一个用 Rust 编写的终端 ASCII 播放器，能将**图片**和**视频**实时渲染成带真彩色的 ASCII 艺术并在终端中播放。

## ✨ 特性

- 🎬 **视频播放**：基于 `ffmpeg-next` 解码视频流，逐帧渲染为 ASCII 动画
- 🖼️ **图片转换**：将常见格式图片（PNG / JPG / ...）转成 ASCII 艺术输出
- 🌈 **24-bit 真彩色**：每个 ASCII 字符使用其原像素的 RGB 颜色（ANSI true-color 转义）
- 📐 **自动适配终端**：根据当前终端的列/行数自动缩放，并修正等宽字符 ~2:1 的宽高比，竖屏视频不会溢出
- 🧱 **模块化结构**：`terminal` / `image` / `ffmpeg` 各自独立，便于扩展

## 📦 依赖

| crate | 用途 |
|-------|------|
| [`ffmpeg-next`](https://crates.io/crates/ffmpeg-next) | 视频解码 |
| [`image`](https://crates.io/crates/image) | 图片解码与缩放 |
| [`crossterm`](https://crates.io/crates/crossterm) | 终端控制（光标隐藏、清屏、定位） |
| [`term_size`](https://crates.io/crates/term_size) | 获取终端尺寸 |
| [`clap`](https://crates.io/crates/clap) | 命令行参数解析 |

> ⚠️ 视频功能依赖系统已安装的 **FFmpeg** 库（libavcodec / libavformat / libavutil / libavdevice / libavfilter / libswscale / libswresample）。请参考 [ffmpeg-next 的安装说明](https://crates.io/crates/ffmpeg-next#installing-ffmpeg)。

## 🚀 安装

```bash
git clone https://github.com/UserB1ank/ruscii_player.git
cd ruscii_player
cargo build --release
```

编译产物位于 `target/release/ruscii_player`。

## 🎮 使用方式

`-v/--video-path` 与 `-i/--img-path` **互斥**，且必须二选一。

### 把图片渲染成 ASCII

```bash
# 输出到 stdout（可重定向到文件）
cargo run --release -- -i path/to/image.png
./target/release/ruscii_player -i photo.jpg
```

### 在终端里播放视频

```bash
cargo run --release -- -v path/to/video.mp4
./target/release/ruscii_player -v clip.mp4
```

视频会进入 crossterm 的 alternate screen 并隐藏光标，逐帧打印 ASCII；播放结束后返回原终端。

### 查看帮助

```bash
./target/release/ruscii_player --help
```

```
Options:
  -v, --video-path <VIDEO_PATH>  the path of video which will be converted to ASCII
  -i, --img-path <IMG_PATH>      the path of image which will be converted to ASCII
  -h, --help                     Print help
```

## 🧩 工作原理

1. **尺寸适配** (`terminal::get_term_size`)：读取终端列/行数，按源画面宽高比 + 单元格宽高比（默认 `0.5`）反推目标分辨率，竖屏视频自动改用按高度填满，避免行数溢出。
2. **解码 + 缩放** (`ffmpeg::render`)：用 ffmpeg 的 `software::scaling` 把每一帧直接缩放到目标分辨率并转成 RGB24，省去二次缩放。
3. **灰度映射** (`image::render_img`)：把每个像素的 luma 值映射到一组从密到疏的 ASCII 字符：

   ```text
   "WM$@%&#NB8E9GAmK6w5HRkbYT43V0JL7gpaesyxznocv?Ijftr1li|*=-~^`':;,. "
   ```

   越暗 → 字符越密集，并用 `\x1b[38;2;R;G;Bm` 给字符上原像素的真彩色。

## 🗂️ 项目结构

```
src/
├── main.rs            # clap 参数解析，分发到 image / video 渲染
├── lib.rs             # 模块导出
├── terminal/
│   ├── size.rs        # get_term_size: 终端尺寸 + 宽高比适配
│   └── term.rs
├── image/
│   └── render.rs      # render_img: 像素 → 彩色 ASCII 字符串
└── ffmpeg/
    └── render.rs      # render_video: 视频解码 + 实时播放
```

## ⚠️ 已知限制

- 视频帧率受解码与终端刷新速度限制，高分辨率 / 高帧率视频可能掉帧
- 终端必须支持 24-bit true color 才能正确显示颜色（Windows Terminal / iTerm2 / 现代终端均支持）
- 目前未实现键盘控制（暂停 / 退出等），强制退出可用 `Ctrl+C`

## 📄 License

[MIT](./LICENSE) © Neo

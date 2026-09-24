# DDNet 动态背景 / Dynamic Background

**本项目只为解决一个问题：让 DDNet / TClient 能用你自己的图片或视频当背景（动态背景）。**
**不提供 Release** —— 想测试请**自行编译**。

| 项目 | 说明 |
|---|---|
| 基于 | **TClient 10.9.0**（[TaterClient/TClient](https://github.com/TaterClient/TClient)），其上游为 [DDNet](https://github.com/ddnet/ddnet) |
| 动态背景实现参考 | [DDNet 论坛：Dynamic background（t=7269）](https://forum.ddnet.org/viewtopic.php?t=7269) |
| 改动范围 | 纯本地、纯视觉：不改网络协议、预测、碰撞、tick，**不影响游戏平衡** |

## 添加功能（验证基本通过）

### 设置 → Background 页

- **自定义背景**：主菜单背景与游戏内实体层背景，两个独立开关；理论支持主流图片与主流视频格式实现动态与静态背景展示
- **背景列表**：列出背景文件夹内容，点一行即切换；另有「打开背景文件夹」与「重载」按钮、以及状态行（已载入 / 失败原因）
- **视频帧率上限**：最大支持 120fps（0 = 跟随片源帧率）
- **背景适配**（折叠表）：auto / 屏幕尺寸 / 多个预设（只决定背景画面比例，不改变游戏分辨率）
- **播放分辨率**（折叠表）：默认原始片源（不压缩），也可限制到 2160p/1440p/1080p/720p —— 只决定解码尺寸
- **显示方式**（折叠表）：上下均等裁剪 / 左右均等裁剪 / 拉伸 / 平铺 —— 完整覆盖、不留黑边，并有一行「当前实际」告示实际生效的裁剪方向
- **Wallpaper Engine 壁纸**：直接用 Wallpaper Engine 的视频/图片壁纸（不含场景和网页壁纸），理论支持壁纸自带声音、音量、失焦暂停（未测试）
- **选项阴影**：按五档分级命名 —— 完全透明 / 朦胧 / 淡藏 / 微淡 / 正常
- **主菜单大标题**显隐开关

### 主菜单

- **隐藏 GUI 按钮**（主页右下角）：点击后除该按钮外隐藏全部界面，只留背景；再点一次或按 `Esc` 恢复。该状态不写入配置，重启必定回到正常界面。

### 性能与引擎（为上面这些服务）

- **视频卡顿根治**：OpenGL 后端对非 2 的幂尺寸纹理每帧在 CPU 上重缩放（2560x1440 → 4096x2048，约 33 MB/帧）；现改为上传纹理补到 2 的幂并用 UV 只采样真实画面 —— 实测菜单帧率 **4.2fps → 271fps**
- 解码线程数可调（默认 4）、追赶跳帧仍尊重帧率上限、可选的诊断落盘开关
- 引擎侧新增：`IGraphics::UpdateTextureRgba()` + `CMD_TEXTURE_UPDATE`（纹理更新不再每帧重建）、`TEXLOAD_NO_MIPMAPS`、`WrapRepeat()`
- 重新链接 FFmpeg 8.1（用于识别并打开视频文件）

## 怎么用

1. **自行编译**：参考 [DDNet 官方构建指南](https://github.com/ddnet/ddnet?tab=readme-ov-file#cloning)。
2. 把图片 / 视频放进**存档目录下的 `Background` 文件夹**（Windows：`%APPDATA%\DDNet\Background`）。
3. 进游戏 → 设置 → Background → 在列表里点一下该文件即可；**路径留空 = 使用游戏原版背景**。

## 说明

- 只做背景相关改动，未触碰玩法逻辑，因此**不影响平衡**，也不会影响联机兼容性（客户端侧视觉改动）。
- Wallpaper Engine 的**声音部分尚未测试**；其余功能均已在实机截图验证。

---

# Dynamic Background for DDNet / TClient (English)

**This project exists to solve exactly one problem: letting DDNet / TClient use your own image or video as a background (dynamic background).**
**No releases are provided** — build it yourself to test it.

| | |
|---|---|
| Based on | **TClient 10.9.0** ([TaterClient/TClient](https://github.com/TaterClient/TClient)), which is based on [DDNet](https://github.com/ddnet/ddnet) |
| Dynamic background reference | [DDNet forum: Dynamic background (t=7269)](https://forum.ddnet.org/viewtopic.php?t=7269) |
| Scope | Purely local and visual: no changes to the network protocol, prediction, collisions or ticks, so **gameplay balance is untouched** |

## Added features (basically verified)

### Settings → Background page

- **Custom background**: independent toggles for the main-menu background and for the in-game entity-layer background; mainstream image and video formats are theoretically supported for both static and dynamic backgrounds
- **Background list**: lists the contents of the background folder, one click switches to a file; plus "open backgrounds folder" and "reload" buttons and a status line (loaded / failure reason)
- **Video frame-rate cap**: up to 120 fps (0 = follow the source frame rate)
- **Background fit** (collapsible list): auto / screen size / several presets (decides the background's picture aspect only, never the game resolution)
- **Video decode resolution** (collapsible list): the source size by default (no downscaling), optionally capped to 2160p/1440p/1080p/720p — decode size only
- **Display mode** (collapsible list): equal crop top/bottom, equal crop left/right, stretch, tile — always fully covered with no letterboxing, plus an "effective" line that states which crop direction is actually in use
- **Wallpaper Engine wallpapers**: use Wallpaper Engine video/image wallpapers directly (scenes and web wallpapers are not supported); the wallpaper's own sound, volume and pause-on-blur are theoretically supported (**untested**)
- **Option shadow**: five named levels — transparent / hazy / faint / subtle / normal
- **Main-menu title** toggle

### Main menu

- **Hide-GUI button** (bottom-right of the start menu): hides every interface element except the button itself, leaving only the background; click it again or press `Esc` to restore. The state is never written to the config, so a restart always brings the normal interface back.

### Performance and engine work behind them

- **Video stutter fix at the root**: the OpenGL backend re-scaled non-power-of-two textures on the CPU every frame (2560x1440 → 4096x2048, about 33 MB per frame); upload textures are now padded to powers of two and sampled through UVs — measured menu frame rate **4.2 fps → 271 fps**
- Configurable decoder thread count (default 4), catch-up frame skipping that still respects the fps cap, optional diagnostics dump
- Engine additions: `IGraphics::UpdateTextureRgba()` + `CMD_TEXTURE_UPDATE` (textures are no longer re-created every frame), `TEXLOAD_NO_MIPMAPS`, `WrapRepeat()`
- Relinked against FFmpeg 8.1 (to detect and open video files)

## How to use

1. **Build it yourself**: see the [official DDNet build guide](https://github.com/ddnet/ddnet?tab=readme-ov-file#cloning).
2. Put your images / videos into the **`Background` folder inside the save directory** (on Windows: `%APPDATA%\DDNet\Background`).
3. In game: Settings → Background → click the file in the list. **An empty path means the original game background is used.**

## Notes

- Only background-related changes were made; no gameplay logic was touched, so **balance is unaffected** and online compatibility is preserved (client-side visual changes only).
- The **sound part of Wallpaper Engine is not tested yet**; every other feature was verified in game with screenshots.

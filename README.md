# FFmpeg FDK-AAC Audiobook Edition

面向**有声书 / 纯音频处理**场景的 Windows 64 位 FFmpeg 静态构建，随包附带 `ffmpeg.exe` 与 `ffprobe.exe`。覆盖响度归一、格式转码、多段 m4a 无损合并为带章节 m4b 等核心需求。


## 本版本组件

| 组件 | 来源 / 版本 |
| --- | --- |
| FFmpeg | 官方源码 n9.0 |
| libfdk-aac | mstorsjo/fdk-aac（main，构建当日最新） |
| libmp3lame | lameproject/lame（main，构建当日最新） |
| libopus | xiph/opus（main，构建当日最新） |
| libsoxr | chirlu/soxr（main，构建当日最新） |

## 版本亮点

- **libfdk-aac**：目前音质最好的 AAC 编码器之一（AAC-LC / HE-AAC / HE-AACv2），有声书高质量转码首选。
- **libsoxr**：高质量重采样库，采样率转换场景下质量显著优于默认重采样器，`aresample` 滤镜可挂 soxr 后端。
- **libmp3lame / libopus**：MP3 与 Opus 编码，覆盖主流有损格式。
- **静态 / nonfree 构建**：单文件免安装、随拷随用；构建内 Verify 步骤对编码器、滤镜、复用器逐项体检，确保功能不随版本漂移。

## 常用场景示例

### 1. 响度归一（EBU R128，有声书常用 -16 LUFS）

单遍快速处理：

```
ffmpeg -i in.m4a -af "loudnorm=I=-16:TP=-1.5:LRA=11" -c:a libfdk_aac -b:a 96k out.m4a
```

两遍动态模式（先测量、后处理，效果更稳）：

```
ffmpeg -i in.m4a -af loudnorm=print_format=json -f null -
ffmpeg -i in.m4a -af "loudnorm=I=-16:TP=-1.5:LRA=11:measured_I=...:measured_TP=...:measured_LRA=..." -c:a libfdk_aac -b:a 96k out.m4a
```

### 2. 格式转码

```
# MP3（VBR 高质量）
ffmpeg -i in.m4a -c:a libmp3lame -q:a 2 out.mp3
# AAC（HE-AAC 低码率）
ffmpeg -i in.m4a -c:a libfdk_aac -profile:a aac_he -b:a 64k out.m4a
# Opus
ffmpeg -i in.m4a -c:a libopus -b:a 48k out.opus
# FLAC（无损）
ffmpeg -i in.m4a -c:a flac out.flac
# ALAC（Apple 无损）
ffmpeg -i in.m4a -c:a alac out.m4a
```

### 3. 多段 m4a 合并为 m4b

无损合并（各段编码参数一致时直接流复制）：

```
ffmpeg -f concat -safe 0 -i filelist.txt -c copy merged.m4b
```

`filelist.txt` 示例：

```
file 'part1.m4a'
file 'part2.m4a'
file 'part3.m4a'
```

添加章节书签（重封装为 m4b）：

```
ffmpeg -i merged.m4a -i chapters.txt -map_metadata 1 -c copy -f ipod -movflags +faststart out.m4b
```

`chapters.txt` 示例（时间单位毫秒）：

```
;FFMETADATA1
title=我的有声书
[CHAPTER]
TIMEBASE=1/1000
START=0
END=1800000
title=第一章
[CHAPTER]
TIMEBASE=1/1000
START=1800000
END=3600000
title=第二章
```

### 4. 静音处理与信息探测

```
# 检测静音（可辅助自动切章）
ffmpeg -i in.m4a -af silencedetect=n=-40dB:d=2 -f null -
# 查看文件信息
ffprobe -show_format -show_streams in.m4a
```

## 支持能力清单（本构建已启用）

- **编码器**：libfdk_aac（AAC-LC / HE-AAC / HE-AACv2）、libmp3lame、libopus，以及原生 aac、flac、alac、vorbis、wavpack、pcm 系列等。
- **滤镜（音频）**：loudnorm（EBU R128 响度归一）、ebur128（响度测量）、dynaudnorm、silencedetect、silenceremove、afade、aresample（可挂 soxr 高质量后端）等。
- **复用器**：mp4 / ipod（m4b 章节书签）、ogg、matroska（.mka）、wav、flac 等。
- **其他**：ffprobe 用于时长 / 采样率 / 章节 / 响度探测；静态链接免依赖。

## 使用说明

- 解压 zip 后直接运行 `ffmpeg.exe` / `ffprobe.exe`，无需安装任何运行库。
- 已通过构建内 Verify 步骤检查：编码器 / 滤镜 / 复用器 / libsoxr 配置逐项确认存在。

## 构建信息

- 平台：GitHub Actions `windows-2022` + MSYS2（MINGW64），x86_64
- 构建方式：静态编译（`--enable-static --disable-shared`），单文件分发
- 许可：FFmpeg LGPL 构建并启用 `--enable-version3 --enable-nonfree`；**libfdk-aac 为 Fraunhofer 非自由许可**，分发与商用前请自行确认授权条款。
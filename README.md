# Video Frame Extractor Skill

从视频中提取帧并识别内容的 OpenClaw Skill。

## 功能

- 根据音频文字稿时间戳提取视频帧
- 识别帧中的内容（代码、UI、文字）
- 自动分析结构化笔记，语义匹配定位插入点
- 更新笔记并保存所有输出

## 安装

```bash
openclaw skills install video-frame-extractor.skill
```

## 使用方法

触发词：
- "音频中说'...'，这个内容在视频的哪一帧"
- "音频中提到'...'，提取视频中的这一帧"

## 文件说明

- `SKILL.md` - Skill 主文件（源代码）
- `video-frame-extractor.skill` - 打包后的 Skill 文件（可直接安装）

## 作者

Created by OpenClaw Skill Creator

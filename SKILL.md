---
name: video-frame-extractor
description: >
  MUST USE when user asks "音频中说'...'，这个内容在视频的哪一帧提取出来" or similar phrases about finding video frames based on audio content. Extracts frames from video at timestamps corresponding to audio transcript keywords, recognizes content (code/UI/text), and inserts into structured notes. Analyzes audio-notes relationship, auto-determines insertion location via semantic matching, validates multiple frames to select clearest, and updates notes. Trigger phrases: "音频中说...哪一帧", "音频中提到...提取视频帧", "从视频找到...这一帧", "提取音频...对应的视频画面", "音频里讲的...在视频哪个时间点", "视频哪一帧显示音频说的...". DO NOT use for simple video download, audio-only transcription, general video editing, or screenshots without audio context.
---

# Video Frame Extractor

Extract video frames based on audio transcript, recognize content, and insert into structured notes.

## Workflow Overview

```
接收指令 → 分析音频 → 提取视频帧 → 验证帧内容 → 识别内容 → 
分析笔记结构 → 语义匹配定位 → 插入内容 → 保存所有输出
```

## Step 1: Parse User Input

Extract from user message:
- `audio_transcript_path`: Path to audio transcript file (e.g., `音频.txt`)
- `video_path`: Path to video file (e.g., `视频.mp4`)
- `keyword`: The keyword/sentence to search for in audio
- `time_range`: User-specified time range (e.g., "04:10-04:15" or "250-255 seconds")
- `notes_path`: Path to structured notes file (e.g., `文字稿-整理版.md`)

## Step 2: Analyze Audio Context

### 2.1 Read Audio Transcript
```bash
read <audio_transcript_path>
```

### 2.2 Locate Keyword and Expand Context
- Find the keyword in the transcript within the specified `time_range`
- Expand context: read 30 seconds before and after the keyword
- Analyze the semantic theme of this section (what is being discussed?)
- Identify the topic, operation, or subject matter

**Context Analysis Example**:
```
Keyword: "拿出咱们准备好的代码，复制粘贴"
Context: Docker部署教程 → 创建Compose文件 → 需要代码
Theme: 代码/配置部署
```

## Step 3: Extract Video Frames

### 3.1 Parse Time Range
Convert time_range to seconds:
- "04:10-04:15" → 250-255 seconds
- "250-255" → 250-255 seconds

### 3.2 Extract Three Frames
```bash
# Frame 1: Start of range
ffmpeg -ss <start_time> -i <video_path> -vframes 1 -q:v 2 frame_start.jpg -y

# Frame 2: Middle of range  
ffmpeg -ss <mid_time> -i <video_path> -vframes 1 -q:v 2 frame_mid.jpg -y

# Frame 3: End of range
ffmpeg -ss <end_time> -i <video_path> -vframes 1 -q:v 2 frame_end.jpg -y
```

**Frame Naming**: `{keyword}_{timestamp}.jpg` (sanitize keyword for filename)

## Step 4: Validate Frame Content

### 4.1 Read All Three Frames
```bash
read frame_start.jpg
read frame_mid.jpg
read frame_end.jpg
```

### 4.2 Content Validation Rules
For each frame, check:
- ✅ Contains expected content type (code, UI, text, etc.)
- ✅ Clear and readable (not blurry, not transition frame)
- ✅ Relevant to the keyword context

**Reject frames that**:
- ❌ Are black screens or transitions
- ❌ Show unrelated content
- ❌ Are too blurry to recognize
- ❌ Don't match the expected content type

### 4.3 Select Best Frame
From validated frames, select the clearest one based on:
- Text sharpness
- Content completeness
- Visual clarity

## Step 5: Recognize Content from Frame

### 5.1 Visual Recognition
Analyze the selected frame to extract:
- **Code**: YAML, JSON, Shell script, etc.
- **UI elements**: Interface screenshots, settings pages
- **Text**: Any readable text content
- **Visual elements**: Diagrams, charts, etc.

### 5.2 Format Recognition Results
- Code: Format as code block with language identifier
- Text: Preserve as-is with formatting
- UI: Describe what is shown

## Step 6: Analyze Structured Notes

### 6.1 Read Notes File
```bash
read <notes_path>
```

### 6.2 Parse Structure
Identify:
- Title hierarchy (# ## ### ####)
- Section topics
- Step sequences
- Existing content gaps (placeholders like "视频中未展示具体代码")

### 6.3 Semantic Matching
Match the audio context to notes structure:

| Audio Context | Notes Structure | Match Confidence |
|--------------|-----------------|------------------|
| "创建Compose文件" | "#### 步骤2：创建Compose文件" | High |
| "复制代码" | "粘贴准备好的代码" | High |
| "部署步骤" | "### 3.2 部署步骤" | Medium |

**Matching Logic**:
1. Look for section titles that semantically match the audio topic
2. Find placeholder text indicating missing content
3. Identify the most specific sub-section that corresponds to the keyword

## Step 7: Determine Insertion Location

### 7.1 Location Priority
1. **Exact match**: Section title matches audio topic exactly
2. **Placeholder**: Location with "视频中未展示" or similar
3. **Contextual match**: Section discussing the same operation
4. **Logical flow**: After previous step, before next step

### 7.2 Insertion Format
Based on content type:
- **Code**: Insert as fenced code block with language
- **Text**: Insert as quoted or regular text
- **UI**: Insert with description and image reference

## Step 8: Update Structured Notes

### 8.1 Prepare Insertion
```markdown
#### 步骤X：XXXX（视频截图时间：MM:SS）

**提取的[内容类型]**：

```[language]
[recognized content]
```

**操作步骤**：
1. ...
```

### 8.2 Execute Edit
```bash
edit <notes_path>
```

Replace placeholder or insert at determined location.

## Step 9: Save All Outputs

### 9.1 Save Video Frame
```bash
# Already saved as frame file
```

### 9.2 Save Recognized Content
```bash
write <notes_dir>/{keyword}_extracted.txt
```

Content:
```
提取时间：[timestamp]
来源视频帧：[frame_filename]
识别内容：
[recognized content]
```

### 9.3 Updated Notes
Already updated in Step 8.

## Output Files

| File | Location | Description |
|------|----------|-------------|
| Video Frame | `<notes_dir>/{keyword}_{timestamp}.jpg` | Extracted frame image |
| Extracted Content | `<notes_dir>/{keyword}_extracted.txt` | Recognized text/code |
| Updated Notes | `<notes_path>` | Structured notes with inserted content |

## Error Handling

| Error Scenario | Handling |
|---------------|----------|
| Keyword not found in audio | Report: "关键词未在指定时间范围内找到" |
| Time range out of video bounds | Report: "时间范围超出视频时长" |
| All frames invalid | Report: "无法提取有效帧，请检查时间范围" |
| Cannot recognize content | Report: "无法识别帧内容，请手动检查" |
| No matching location in notes | Report: "无法在笔记中找到匹配位置，请手动指定" |
| Insertion point ambiguous | Report ambiguous options, ask user to choose |

## Example Usage

**User Input**:
> "音频中说'拿出咱们准备好的代码，复制粘贴'，这个代码是在视频的哪一帧提取出来的"

**Parameters**:
- audio_transcript_path: `音频.txt`
- video_path: `视频.mp4`
- keyword: "拿出咱们准备好的代码"
- time_range: "04:10-04:15"
- notes_path: `文字稿-整理版.md`

**Execution**:
1. Read audio.txt → Find keyword at 04:10
2. Expand context → "创建Compose文件...复制粘贴代码"
3. Extract frames at 04:10, 04:12, 04:15
4. Validate → All 3 show code, select clearest (04:10)
5. Recognize → Extract YAML code
6. Read notes → Find "#### 步骤2：创建Compose文件"
7. Match → Context matches "复制代码"
8. Insert → Replace placeholder with code block
9. Save → frame_250.jpg, code_extracted.txt, updated notes

## Notes

- Always validate frame content before recognition
- Use semantic context, not just keyword matching
- Preserve original notes formatting when inserting
- Include video timestamp in insertion for traceability
- Save intermediate files for user verification

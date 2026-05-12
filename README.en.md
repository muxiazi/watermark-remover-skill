# Watermark Remover Skill (Team Installation Guide)

[🇨🇳 简体中文](./README.md) | **🇬🇧 English** | [🇯🇵 日本語](./README.ja.md) | [🇰🇷 한국어](./README.ko.md)


Enables AI agents like Claude / Claude Code to automatically call the "Buyi Batch Image Watermark Remover" to process images.
If you don't have the software yet, search for "布衣图片批量去水印软件" (Buyi Batch Image Watermark Remover) to obtain it.

---

## How It Works (Understand This Before Installing)

```
[Agent] ──drops images──► [Input Dir] ──auto-triggered by watcher──► [GUI Processes] ──writes to──► [Output Dir] ──Agent reads
```

The agent does NOT click GUI buttons directly. Instead, it **copies images into the directory the GUI is watching**, then **polls the output directory** until the results appear.

This means the GUI must be **running the entire time** and the **"📡 Monitor Directory" button must be turned ON**.

---

## Installation Steps

### Membership signup: https://buyitanan.com/bu_yi_tu_pian_pi_liang_qu_shui_yin

### 1. Prepare the GUI Application

> **Note:** The GUI itself is currently Chinese-only. The English button names below are translations for your reference — the actual labels in the app are in Chinese (shown in parentheses).

- Install and launch the "Buyi Batch Image Watermark Remover" (布衣图片批量去水印软件)
- Sign in to your account (**must NOT be a free-tier account** — free tier doesn't have monitoring permission)
- In the GUI:
  - Click "Select Input Folder" (选择输入文件夹) and choose a directory (recommended: create a dedicated one, e.g. `D:\watermark_in`)
  - Click "Select Output Folder" (选择输出文件夹) and choose another directory (e.g. `D:\watermark_out`)
  - Click the "📡 Monitor Directory: OFF" (📡 监控目录：关闭) button — it should turn green and read "📡 Monitor Directory: ON" (📡 监控目录：开启)
- Keep the program running (minimizing is fine, but do not close it)

### 2. Install This Skill

```bash
# Copy this directory anywhere on the team member's machine
# e.g. ~/skills/watermark-remover/
```

Requires Python 3.8+ only. **No third-party dependencies.**

### 3. Create the Config File

```bash
cp config.example.json config.json
```

Edit `config.json` and set `input_dir` / `output_dir` to the exact two directories you selected in the GUI in Step 1. **The two sides must match exactly.**

### 4. Run the Self-Check

```bash
python watermark_remover.py check
```

You should see ✅ indicating that both the input and output directories are reachable.

For an end-to-end live test using a sample image:

```bash
python watermark_remover.py check --sample test.png --timeout 60
```

If you get ✅ within 60 seconds, the entire pipeline from Agent → GUI is wired up correctly.

---

## Deploying for Claude

Just upload the entire `watermark-remover-skill/` directory to your Claude project's knowledge base / skill library. Claude reads the `name` / `description` frontmatter in `SKILL.md` and will invoke the skill automatically when a user mentions watermark-removal tasks.

For CLI agents like Claude Code, place the directory somewhere the agent can access and tell the agent the config path, or set an environment variable:

```bash
export WATERMARK_REMOVER_CONFIG=~/skills/watermark-remover/config.json
```

---

## Common Commands

```bash
# Self-check
python watermark_remover.py check
python watermark_remover.py check --sample test.png

# Process a single image
python watermark_remover.py process input.jpg

# Process multiple images
python watermark_remover.py process a.jpg b.png c.webp

# Process an entire directory
python watermark_remover.py process ~/photos_to_clean/

# Large batch + custom timeout + export results
python watermark_remover.py process ~/big_batch/ --timeout 1800 --json-out result.json

# Skip the per-task subdirectory (drop files directly into the input root)
python watermark_remover.py process input.jpg --no-subdir

# Move instead of copy (THIS DELETES THE ORIGINAL!)
python watermark_remover.py process input.jpg --move
```

---

## Team Collaboration Notes

When multiple team members share a single GUI instance (same machine), agree on the following:

1. **Always use per-task subdirectories for isolation.** By default a UUID subdirectory is generated for every task, so concurrent runs don't collide. **Do not casually pass `--no-subdir`.**
2. **Do not wipe each other's persistence records.** Clicking "Yes to All" on the GUI's "Clear processed list" prompt will wipe `processed_files.json`, which may cause other people's jobs to be reprocessed. **Coordinate with the team before doing this.**
3. **Give the timeout enough headroom.** A single image usually takes 3–30 seconds, but batch processing of large images can take several minutes. If the agent reports "pending," first check whether the GUI is still working before deciding whether to give up or extend the timeout.
4. **Establish a "cleanup after completion" routine.** After processing is done, let the agent delete the task subdirectory from the input folder to keep the environment tidy.

---

## File Manifest

```
watermark-remover-skill/
├── SKILL.md              # Instruction file read by Claude/Agent (do NOT modify the frontmatter)
├── README.md             # This file, for humans
├── watermark_remover.py  # Core script (usable as both CLI and library)
├── config.example.json   # Config template
└── config.json           # Your actual config (copy from the template on first install and edit)
```

---

## Feedback & Extensions

- If you want a "GUI-less" CLI version (calling the algorithm directly without the software), you'll need to extract the `InpaintWorker` processing logic from the original GUI code. That's not covered by this skill yet.
- If you want webhook / HTTP API triggering, you can extend the `WatermarkRemoverClient` class on your own.

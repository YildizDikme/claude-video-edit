# video-edit: a Claude Code skill for editing videos

A Claude Code skill for editing talking-head, tutorial, explainer and Short videos together with Claude:

- cuts raw takes on real word edges, without clipping words
- removes retakes and pauses
- builds motion graphics as HTML + GSAP pages, locked to the spoken words
- synthesizes sound effects in Python
- keeps the voice clean and at the right loudness
- renders frame-exact

[Türkçe açıklama aşağıda](#türkçe).

## Read this first

Claude cannot hear. Everything this skill knows about sound is measured: word timestamps from Whisper, a 10 ms energy envelope, loudness meters. So the workflow is built around that:

- **Cuts follow the measured energy, not guesses.** Each cut snaps to the real start and end of a word in the waveform.
- **Picture and sound share one clock.** Graphics and sound effects are triggered from the same timeline, so effects land on the picture without anyone listening for sync.
- **The voice is never processed by default.** No noise reduction, EQ or compression: one linear gain only. Those filters are the usual cause of muffled voices.
- **A human reviews every stage.** Claude will not produce a perfect edit in one go. You work stage by stage: rough cut, review, chapters, graphics range by range, sound, final render. Expect corrections, and that is how good edits get made.

## What is inside

```
SKILL.md              the workflow Claude follows
reference/audio.md    cutting rules, loudness, sound effects, captions (the most important file)
reference/style.md    glass cards, neutral palette, typography, motion per video type, things to avoid
reference/edl.md      the edit-list format
scripts/              transcribe, cuts, build, verify, loudness, sfx, mix (Python) · render, audit (Node)
template/             a scene page: engine, glass-card styles, chapter veils, demo beats
```

## Install

1. Requirements: Python 3.10+ with numpy, ffmpeg and ffprobe, Node 20+, and Google Chrome or Chromium. You also need one Whisper backend: `pip install faster-whisper` on any OS, or `brew install whisperkit-cli` on Apple Silicon.
2. Put this folder where Claude Code finds skills, for example:

```bash
git clone <this repo> ~/.claude/skills/video-edit
```

3. Try the template without any footage:

```bash
cd ~/.claude/skills/video-edit
python3 scripts/sfx.py lib --out template/sfx
python3 scripts/serve.py --project . --port 5600
```

Open http://127.0.0.1:5600/template/?demo and press Space.

## How to use it

In your project folder, put the raw clips in `src/` and ask Claude, for example:

> Edit these clips into a 10-minute educational video. Follow script.md. Cut retakes and pauses, natural pace. Propose chapters. Graphics calm, no sound effects. Neutral palette with our accent #2E7D6B.

> This is a Short. Tight cuts, captions on, more energetic graphics, quiet effects on the highlights.

Claude asks a few questions, transcribes, makes a rough cut, and hands you a preview with a list of what it decided. You give timestamps and corrections. Then it adds chapters, builds the graphics one range at a time, and renders when you approve.

## Video types

| Type | Graphics | Effects | Cuts |
|---|---|---|---|
| Educational / explainer | calm, clear diagrams, slow camera, three.js only when it explains | none or very quiet | natural |
| Talk | few long-lasting cards, never over the face | quiet transitions | natural or tight |
| Tutorial | zoom into the screen, highlight boxes, steps | optional soft clicks | natural |
| Short | bigger, faster, hook in 2 s, captions | allowed, under the voice | tight |

## Limits

- Claude can't judge taste, emotion, or whether a mix "sounds good". Listen to every flagged join and every effect level.
- Whisper makes mistakes with names and technical words. Check the transcript.
- Rendering through a browser is slow, about 10 fps per worker at 1080p. Use several workers for long videos.

## License

MIT. See `LICENSE`.

---

## Türkçe

Bu repo, Claude Code ile video editlemek için hazırlanmış bir skill. Konuşma, eğitim, ekran kaydı ve Short videolarında şunları yapar:

- ham çekimleri kelime sınırlarından keser, kelimeleri kırpmaz
- tekrar çekimleri ve duraksamaları çıkarır
- konuşmaya kilitli animasyonları HTML + GSAP sayfaları olarak kurar
- ses efektlerini Python ile matematikten üretir
- sesi temiz ve doğru seviyede tutar
- kare kare render alır

### En önemli kısım

Claude duymaz. Bu skill sesle ilgili her şeyi ölçerek yapar: Whisper kelime zamanları, 10 ms'lik enerji zarfı ve ses yüksekliği ölçümleri. Bu yüzden:

- **Kesimler ölçülen enerjiye göre yapılır.** Kelime başları ve sonları dalga formundaki gerçek yerinden kesilir.
- **Görüntü ve ses aynı saatten tetiklenir.** Animasyon da efekt de aynı zaman çizelgesinden çalıştığı için efekt görüntüye oturur.
- **Ses varsayılan olarak işlenmez.** Gürültü azaltma, EQ ya da kompresör yok, sadece doğrusal kazanç var. Boğuk sesin en sık sebebi bu filtreler.
- **Her aşamayı bir insan kontrol eder.** Claude ilk seferde mükemmel edit çıkarmaz. Aşama aşama çalışılır: kaba kurgu, kontrol, bölümler, animasyonlar aralık aralık, ses, final render.

### Kurulum ve deneme

Gereksinimler: Python 3.10+, numpy, ffmpeg, Node 20+, Chrome ve bir Whisper (`pip install faster-whisper` ya da Apple Silicon'da `brew install whisperkit-cli`).

```bash
git clone <bu repo> ~/.claude/skills/video-edit
```

Görüntü olmadan şablonu denemek için `python3 scripts/serve.py --project . --port 5600` çalıştırıp http://127.0.0.1:5600/template/?demo adresini açın, Space'e basın.

### Kullanım

Proje klasörünüzde ham klipleri `src/` içine koyun ve Claude'a ne istediğinizi söyleyin. Örneğin: "Bu klipleri 10 dakikalık bir eğitim videosuna çevir, tekrarları ve duraksamaları kes, bölüm öner, animasyonlar sakin olsun, efekt koyma."

Video türüne göre tempo değişir:

- Eğitim videosunda animasyonlar sakin ve açıklayıcı olur, gerekirse three.js kullanılır.
- Short'ta animasyonlar daha hareketli olur, kesimler sıkıdır ve altyazı açıktır.

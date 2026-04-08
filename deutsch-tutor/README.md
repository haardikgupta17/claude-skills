<p align="center">
  <img src="https://img.shields.io/badge/🇩🇪_Deutsch-Tutor-black?style=for-the-badge&labelColor=DD0000&color=FFCE00" alt="Deutsch Tutor" />
</p>

<h1 align="center">Deutsch Tutor</h1>

<p align="center">
  <strong>Your AI German teacher, right inside Claude Code.</strong><br/>
  From zero to Goethe-Zertifikat B1 — one daily lesson at a time.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Level-A1_→_B1-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/Time-5--10_min%2Flesson-green?style=flat-square" />
  <img src="https://img.shields.io/badge/Style-Fun_%26_Practical-orange?style=flat-square" />
  <img src="https://img.shields.io/badge/Platform-Claude_Code-blueviolet?style=flat-square" />
  <img src="https://img.shields.io/badge/Exam-Goethe_B1-red?style=flat-square" />
</p>

---

## What is this?

A Claude Code skill that turns your terminal into a German classroom. No apps, no subscriptions, no Duolingo owl guilt-tripping you at 11pm.

Just type `/deutsch-tutor` and get a structured, bite-sized lesson covering **all four exam skills**:

<table>
<tr><td>📖</td><td><strong>Vocabulary</strong></td><td>5 new words per lesson, always with articles</td></tr>
<tr><td>📝</td><td><strong>Grammar</strong></td><td>Sneaked in through real-life scenarios, never boring</td></tr>
<tr><td>🎭</td><td><strong>Dialogs</strong></td><td>Mini conversations set in Baeckereien, Bahnhoefe, and Bueros</td></tr>
<tr><td>🗣️</td><td><strong>Speaking</strong></td><td>Full response prompts — introduce yourself, order food, express opinions</td></tr>
<tr><td>✍️</td><td><strong>Writing</strong></td><td>Emails, forms, messages — mirrors Goethe exam format</td></tr>
<tr><td>📄</td><td><strong>Reading</strong></td><td>Real German texts with comprehension questions</td></tr>
<tr><td>👂</td><td><strong>Listening Sim</strong></td><td>Dictation, cloze passages, sentence reconstruction</td></tr>
<tr><td>🧠</td><td><strong>Spaced Review</strong></td><td>Failed concepts come back until they stick</td></tr>
</table>

## Why not just use Duolingo?

| | Duolingo | Deutsch Tutor |
|---|---------|--------------|
| Cost | Free / $7.99/mo | Free |
| Real-life context | "The elephant eats bread" | "Your landlord sent a Nebenkostenabrechnung" |
| Grammar depth | Vibes | Actual rules with tables |
| Speaking practice | Repeat after robot | Write full responses to real scenarios |
| Writing practice | ❌ | Emails, forms, messages in exam format |
| Reading practice | Translate single sentences | Full passages with comprehension Qs |
| Exam prep | ❌ | Built for Goethe A1/A2/B1 |
| Customization | None | Adapts to your pace, city & weak spots |
| Guilt trips | 🔥 Streak anxiety | Zero guilt, your pace |
| Skipping allowed | Yes | **No.** You earn the next lesson. |

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/haardikgupta17/claude-skills.git
```

### 2. Copy the skill to your Claude Code commands folder

```bash
cp claude-skills/deutsch-tutor/deutsch-tutor.md ~/.claude/commands/
```

### 3. Run it

Open Claude Code and type:

```
/deutsch-tutor
```

That's it. No API keys, no config files, no setup wizard.

## How it works

```
┌──────────────────────────────────────────────────────┐
│                    /deutsch-tutor                     │
├──────────────────────────────────────────────────────┤
│                                                      │
│  1. 📋 Check Progress                                │
│     └── Reads ~/Documents/German/progress.md         │
│                                                      │
│  2. 🎓 Deliver Lesson (one focused type per lesson)  │
│     ├── V  Vocabulary — 5 new words + drills         │
│     ├── G  Grammar — one concept + exercises         │
│     ├── D  Dialog — real-life conversation            │
│     ├── S  Speaking — full response to a prompt       │
│     ├── W  Writing — email/message in exam format     │
│     ├── R  Reading — passage + comprehension Qs       │
│     ├── L  Listening — dictation & reconstruction     │
│     └── Q  Review — flashcard quiz of all vocab       │
│                                                      │
│  3. ✅ Grade & Correct                                │
│     └── Every answer explained, not just "wrong"     │
│                                                      │
│  4. 📝 Update Files (at session end only)             │
│     ├── progress.md  → lesson status                 │
│     ├── vocabulary.md → new words added              │
│     └── grammar.md   → grammar rules added           │
│                                                      │
│  5. 🔄 Plan Ahead                                    │
│     └── Generate next week when current is done      │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## Lesson types in detail

Each lesson is **small and focused** — one thing done well in 5-10 minutes. No cramming vocab + grammar + dialog into a single lesson.

### 📖 Vocabulary (V)
> *5 words, each with article, pronunciation, meaning, example sentence.*

Learn words you'll actually use — at REWE, at the Auslaenderbehoerde, with your landlord. Every noun comes with its article. Always.

### 📝 Grammar (G)
> *One concept, disguised as "here's how to say X in real life."*

No grammar walls. No dry conjugation tables (unless they actually help). Just patterns shown through examples, then a simple rule.

### 🎭 Dialog (D)
> *4-8 line conversation in a real place, with translation.*

Set in a Baeckerei, WG kitchen, Arztpraxis, or Bahnhof. Usually funny. Always relatable if you live in Germany.

### 🗣️ Speaking (S)
> *Real-life prompt → you write your full response in German.*

"Introduce yourself to your new neighbor." "You're at the doctor — explain what hurts." Graded on grammar, vocab, naturalness. Mirrors Goethe Sprechen format.

### ✍️ Writing (W)
> *Write an email, fill a form, compose a message.*

Starts with 2-3 sentence messages at A1, builds to 80-word semi-formal emails at B1. Graded with a corrected rewrite showing improvements.

### 📄 Reading (R)
> *Short German text + comprehension questions.*

Apartment notices, restaurant menus, Deutsche Bahn announcements, job listings. True/false, multiple choice, "what does X mean?"

### 👂 Listening Simulation (L)
> *Dictation, cloze passages, sentence reconstruction.*

Can't play audio in a terminal, but we can test the same skills: fill in blanks, unscramble sentences, complete passages from context.

### 🧠 Review (Q)
> *Full flashcard quiz of everything learned so far.*

Every 5th lesson. All vocabulary, all grammar, all weak spots. The checkpoint you must pass.

### Rotation pattern

Every ~10 lessons follows a balanced cycle:

```
V → G → V → D → S → V → G → W → R → Q/L
```

Adjusted based on your weak spots — struggling with grammar? More G lessons. Speaking shaky? More S lessons.

## First run

On your first `/deutsch-tutor`, the skill asks a few questions:

- **Target exam** — Goethe A1, A2, B1, TestDaF, or just conversational
- **Current level** — complete beginner, A1, A2, etc.
- **Why you're learning** — citizenship, work, partner, travel
- **Your city** — for realistic scenario context
- **Daily time** — 15 min? 30 min? 1 hour?

It then generates a personalized curriculum and creates your progress files.

## Progress files

All progress is stored locally in `~/Documents/German/`:

| File | Purpose |
|------|---------|
| `progress.md` | Lesson tracker — planned vs actual dates, lesson type, status |
| `vocabulary.md` | Every word you've learned, organized by lesson |
| `grammar.md` | Grammar rules reference, built up over time |

These files persist across sessions — pick up right where you left off.

## Curriculum structure

The skill auto-generates a phased curriculum based on your target:

| Phase | Focus | Target |
|-------|-------|--------|
| 🟢 Phase 1 | Basics — greetings, numbers, articles, present tense, first dialogs | A1 |
| 🟡 Phase 2 | Past tenses, modal verbs, prepositions, emails, reading passages | A2 |
| 🟠 Phase 3 | Complex grammar, Nebensaetze, opinion writing, speaking prompts | B1 |
| 🔴 Phase 4 | Full mock exams (all 4 parts), timed practice, weak spot drilling | Exam ready |

## The rules

> *"Treat the student like a smart but reluctant 5-year-old."*

1. **No skipping.** You don't get the next lesson until you complete the current one's exercises. No exceptions.
2. **No grammar walls.** Rules come disguised inside real-life scenarios.
3. **Articles are sacred.** Never learn a noun without der/die/das. EVER.
4. **Drill, don't lecture.** Every lesson ends with exercises you must complete and pass.
5. **Keyboard-friendly.** Can't type umlauts? `ae oe ue ss` are always accepted — never marked wrong.
6. **Adapt to pace.** Crushing it? We push harder. Struggling? We slow down and review.
7. **All four skills.** Vocab and grammar aren't enough — you need speaking, writing, reading, and listening to pass the exam.

## Example lessons

<details>
<summary><strong>📖 Vocabulary lesson</strong></summary>

```
Lektion 12: Essen & Trinken — Vocab

| Deutsch     | Article | Pronunciation | English | Example              |
|-------------|---------|---------------|---------|----------------------|
| das Wasser  | (n)     | (vah-ser)     | water   | Ein Wasser, bitte.   |
| das Brot    | (n)     | (broht)       | bread   | Das Brot ist frisch. |
| die Milch   | (f)     | (milh)        | milk    | Die Milch ist kalt.  |

Q1. What's the article? Wasser → ___, Brot → ___
Q2. Translate: "One water, please."
```
</details>

<details>
<summary><strong>🗣️ Speaking lesson</strong></summary>

```
Lektion 20: Introduce Yourself — Speaking

Scenario: You just moved into a new WG in Berlin.
Your neighbor knocks on the door and says "Hallo!"

Write 4-5 sentences introducing yourself:
- Your name
- Where you're from
- Where you work
- What you drink in the morning

(Write your full response in German. I'll grade it.)
```
</details>

<details>
<summary><strong>✍️ Writing lesson</strong></summary>

```
Lektion 35: Email to Your Landlord — Writing

Situation: Your heating (die Heizung) is broken.
Write a short email to your landlord (Herr Mueller):
- Greeting
- Explain the problem
- Ask when someone can come fix it
- Sign off politely

(I'll grade format, grammar, and vocabulary.)
```
</details>

<details>
<summary><strong>📄 Reading lesson</strong></summary>

```
Lektion 42: Supermarkt Notice — Reading

Read this notice posted at REWE:

"Ab Montag, den 15. April, ist unser Supermarkt
von 7:00 bis 22:00 Uhr geoeffnet. Am Sonntag
bleiben wir geschlossen. Vielen Dank!"

Q1. When does the new schedule start?
Q2. True or false: The store is open on Sundays.
Q3. What are the new opening hours?
```
</details>

## Supplementary resources (all free)

This skill is designed to be your **primary tutor**, but it can't play audio or have a real-time conversation. Fill those gaps with these free resources:

### 👂 Listening (start at A1)
| Resource | What it is | Link |
|----------|-----------|------|
| **Easy German** | Street interviews with German + English subtitles | YouTube — search "Easy German" |
| **Deutsche Welle** | Free A1-B1 courses with audio exercises | dw.com/learngerman |
| **Slow German** | Podcast speaking slowly on everyday topics | Available on all podcast apps |

### 🗣️ Speaking practice (start at A2)
| Resource | What it is | Cost |
|----------|-----------|------|
| **italki** | 30-min sessions with native tutors | ~10 EUR/session |
| **Tandem app** | Language exchange — teach English, learn German | Free |
| **Mirror practice** | Talk to yourself in German. Seriously. It works. | Free + mild embarrassment |

### 📺 Immersion (start at A2+)
| Resource | What it is |
|----------|-----------|
| **Netflix with German audio + German subs** | Watch shows you already know (dubbed) |
| **ARD Mediathek / ZDF Mediathek** | Free German TV, news, documentaries |
| **German radio** | Deutschlandfunk, WDR — background listening while working |

### 📝 Exam prep (1-2 months before exam)
| Resource | What it is |
|----------|-----------|
| **Goethe Institut practice exams** | Official mock exams for A1/A2/B1 — buy one set |
| **Prüfungstraining books (Cornelsen)** | Structured exam prep with answer keys |

### How to combine

```
Daily:          /deutsch-tutor (10-20 min) ← your main driver
2-3x per week:  Easy German or video course (20-30 min) ← listening
1x per week:    italki or Tandem (30 min) ← real speaking (once A2)
Background:     German Netflix/radio ← passive immersion
```

The skill should always stay **ahead** of supplementary material. Everything else should feel like review, not new content.

## Tips

- **Be consistent** — even 5 minutes daily beats 2 hours on weekends
- **Do the exercises** — the skill won't let you move on until you answer
- **Review vocabulary.md** — it's your personal dictionary, use it
- **Don't skip speaking & writing** — they feel harder, but that's where exam points live
- **Trust the pace** — the curriculum is designed with exam dates in mind

## Contributing

Found a bug? Have an idea? Open an issue or PR. Especially welcome:
- New lesson scenarios (dialogs, reading passages)
- Curriculum improvements
- Support for other CEFR target levels (A2, B2)

## License

MIT — Viel Erfolg! 🇩🇪

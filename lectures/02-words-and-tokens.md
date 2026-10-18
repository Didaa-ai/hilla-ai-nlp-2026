# Lecture 2: Words and Tokens

**University of Hilla** · Dept. AI / NLP · 2026/27
**Assistant lecturer:** Hussein Mahdi Al-Rubaie
**Duration:** 90 minutes (theory) · **Date:** 10/18/2026

---

## Questions for today

By the end of this lecture, you will be able to answer:

1. What is a **word**, and why is it hard to define?
2. What is the difference between **word types** and **word instances**?
3. Why does the **vocabulary keep growing** (Herdan's / Heaps' Law)?
4. What are **morphemes**, and why do we need units **smaller than words**?
5. How do computers store characters (**Unicode, UTF-8**)?
6. How do we find patterns in text with **regular expressions**?

---

## 1. A Simple Introduction: ELIZA

```
ELIZA: Is something troubling you ?
YOU:   Men are all alike.
ELIZA: What is the connection, do you suppose ?
YOU:   They're always bugging us about something or other.
ELIZA: Can you think of a specific example ?
YOU:   Well, my boyfriend made me come here.
ELIZA: Is it important to you that your boyfriend made you come here ?
YOU:   He says I'm depressed much of the time.
ELIZA: I am sorry to hear that you are depressed.
```

- **ELIZA** (Weizenbaum, 1966) was an early NLP system that held a limited conversation by imitating a psychotherapist.
- It used **pattern matching** on words: "I need X" → "What would it mean to you if you got X?"
- Many users came to believe ELIZA really understood them.
- Modern **chatbots don't work** this way, but the **pattern-based idea survives in tokenization**: separating words and word parts from running text.

*Tokenization is the **first step** in modern NLP. To do it, we first ask: **what is a word?***

---

## 2. What is a Word?

- A simple answer: a string separated by spaces.
- Words are useful units of representation, but **difficult to define formally**.

**Question to students:** How many words are in this sentence?

> They picnicked by the pool, then lay back on the grass and looked at the stars.

```
[They] [picnicked] [by] [the] [pool] (,) [then] [lay] [back]
[on] [the] [grass] [and] [looked] [at] [the] [stars] (.)

[ ] only        = 16 words  (no punctuation)
[ ] + ( )       = 18 words  (with the comma and the period)
```

### Does punctuation count?

It **depends on the task**:

- Commas, periods and colons help find **boundaries**.
- Question marks, exclamation marks and quotation marks carry **meaning**.
- **Large language models** generally count punctuation as separate words.

### Simple decisions when counting words

| Question                            | The book's answer                                                                 |
| ----------------------------------- | --------------------------------------------------------------------------------- |
| Is punctuation a word?              | Depends on the task; LLMs count it separately                                     |
| Are *They* and *they* the same word? | Depends on the task; sometimes two versions of a model are kept, with and without capitalization |
| Is *I'm* one word or two?           | One **orthographic** word (by writing), but grammatically two: *I* + *'m* (= am)  |

*\* **Orthographic words** = words defined by our writing system (separated by spaces in English).*

---

## 3. Spoken Language

- Speech adds **new problems** for defining words.
- **Utterance** = the spoken equivalent of a sentence.

> I do uh main- mainly business data processing

### Key terms

| Term                      | Meaning                                         | Example                                       |
| ------------------------- | ----------------------------------------------- | --------------------------------------------- |
| **Disfluency**            | A break in fluent speech                        | The utterance above has two kinds: a fragment and a filler |
| **Fragment**              | A word the speaker starts, cuts off, then restarts | *main-*                                    |
| **Filler / filled pause** | Sounds that fill a pause                        | *uh*, *um*                                    |

- **How *main-* is a fragment:** the speaker began to say *mainly*, stopped after *main*, then said the whole word again: *main- mainly*. The hyphen marks where the word was cut off.

### Another example

*(My own illustration, not from the book)*

> I'll see you on um Thurs- Thursday

- **Fragment:** *Thurs-* (cut off, then restarted as *Thursday*)
- **Filler:** *um*

### Should fillers be counted as words?

It **depends on the application**:

- A **transcription** system may strip them out.
- **Speech recognition** keeps them: they help predict the next word, since they signal the speaker is restarting.
- They can also help **identify the speaker**.

**Question to students:** Clark and Fox Tree (2002) showed that *uh* and *um* have **different meanings**. What might they be?

---

## 4. Word Types vs. Word Instances

| Term                | Meaning                                    | Symbol |
| ------------------- | ------------------------------------------ | ------ |
| **Word types**      | The number of **distinct** words in a corpus | \|V\| (vocabulary size) |
| **Word instances**  | The **total** number of running words      | N      |

**Example** (ignoring punctuation):

> They picnicked by the pool, then lay back on the grass and looked at the stars.

```
[1]They [2]picnicked [3]by [4]THE [5]pool [6]then [7]lay [8]back
[9]on [10]THE [11]grass [12]and [13]looked [14]at [15]THE [16]stars

Instances  N  = 16   (count every word)
Types     |V| = 14   ("the" appears 3 times, but counts once)
```

*Note: older texts call instances "word tokens"; the textbook keeps the word **token** for the output of **subword tokenizers**.*

---

## 5. The Problem: Languages Without Orthographic Words

- **Chinese, Japanese and Thai** don't use spaces to mark word boundaries.
- Chinese words are made of **characters** (*hanzi*). Each character is usually one unit of meaning and one syllable; words average **2.4 characters**.
- So deciding what counts as a word in Chinese is **hard**.

### Proposed solutions for Chinese

*Sentence:* 姚明进入总决赛 — "Yao Ming reaches the finals."

| Standard                     | Rule                                      | Segmentation          | Count        |
| ---------------------------- | ----------------------------------------- | --------------------- | ------------ |
| 1. **Chinese Treebank (CTB)** | Names are one word                        | 姚明 / 进入 / 总决赛    | 3 words      |
| 2. **Peking University (PKU)** | Names split; some adjectives separate     | 姚 / 明 / 进入 / 总 / 决赛 | 5 words      |
| 3. **Characters only**       | Ignore words                              | 姚 / 明 / 进 / 入 / 总 / 决 / 赛 | 7 characters |

- Using characters **works well for Chinese**, because a character is about **the right meaning size**.
- It does **not** work for **Japanese and Thai**, where one character is too small a unit.

*Tokenization **depends on the segmentation standard**.*

---

## 6. Herdan's Law / Heaps' Law

### Why it appeared

A second problem: **there are too many words**. How many words are in English?

The bigger the corpus, the more word types we find (book's Fig. 2.1):

| Corpus                         | Types \|V\|  | Instances N |
| ------------------------------ | ------------ | ----------- |
| Shakespeare                    | 31 thousand  | 884 thousand |
| Brown corpus                   | 38 thousand  | 1 million   |
| Switchboard (phone conversations) | 20 thousand | 2.4 million |
| COCA                           | 2 million    | 440 million |
| Google n-grams                 | 13 million   | 1 trillion  |

*There is **no fixed answer**; the count keeps growing with more data.*

### The law

```
|V| = k · N^β
```

- **k** and **β** are positive constants, and **0 < β < 1**.
- Named **Herdan's Law** (linguistics) or **Heaps' Law** (information retrieval) after its discoverers.
- **β** depends on corpus size and genre; values from **0.44 to 0.56** are often reported.
- In simple words: **vocabulary size grows a little faster than the square root of the text length.**

**Example from the table:**

- Brown: N = 1 million, about 38 thousand types.
- COCA: **440×** more instances (440 million), but only about **53×** more types (2 million).

*Types grow **much more slowly** than instances.*

### Two classes of words and β

| Class              | Description                                          | Examples                 | Growth                                         |
| ------------------ | ---------------------------------------------------- | ------------------------ | ---------------------------------------------- |
| **Function words** | Grammatical words                                    | *a*, *of*                | Roughly fixed; a language has a fixed number of these |
| **Content words**  | Nouns, adjectives, verbs about people, places, events | names, technical terms  | Keep growing indefinitely                      |

Some versions of the law use **two values of β**: one for the beginning of a corpus, and one later.

 
Vocabulary size vs. text length (Gutenberg corpus, log–log scale) , Figure 2.2 in the book, from Tria et al. (2018)

 

- **x-axis:** words read (instances N); **y-axis:** different words found (types |V|); both on log scale.
- **At the start, β ≈ 1:** almost every word is new.
- **Later, β ≈ 0.44:** only content words keep adding new types.

---

## 7. The Unknown Word Problem

- Because words keep growing, **no vocabulary can ever hold every possible word**.
- A model will constantly meet **unknown words**: words it has never seen before. This is a huge problem for machine learning models.

**Two problems together:**

1. Many languages **don't have orthographic words**, and defining them is hard.
2. The number of words **grows without bound**.

**Solution:** language models use smaller units called **subwords**, which can be recombined to build new, unseen words.

*To define subwords, we first look at units **smaller than words**: **morphemes** and **characters**.*

---

## 8. Morphemes

### What is a morpheme?

- **Morpheme** = a minimal **meaning-bearing unit** in a language.
- **Morphology** = the study of morphemes.

| Word   | Morphemes        | Count |
| ------ | ---------------- | ----- |
| fox    | fox              | 1     |
| cats   | cat + -s (plural) | 2    |

An English sentence split into morphemes:

```
Doc work-ed care-ful-ly wash-ing the glass-es
```

*In Chinese, each character mainly describes one morpheme.*

### Roots and affixes

| Class     | Role                                          | Example                              |
| --------- | --------------------------------------------- | ------------------------------------ |
| **Root**  | The central morpheme; gives the main meaning  | *work* in *worked*; *glass* in *glasses* |
| **Affix** | Adds extra meaning                            | *-ed* in *worked*; *-es* in *glasses*    |

### Types of affixes (plus clitics)

| Type              | Description                                                           | Example                                                          |
| ----------------- | --------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Inflectional**  | Productive, often required, predictable meaning                       | *-s / -es* plural (*glasses*); *-ed* past tense (*worked*)       |
| **Derivational**  | Less predictable; applies to some words only; usually changes the word class | *care* (noun) → *careful* (adjective) → *carefully* (adverb) |
| **Clitic**        | Acts like a word grammatically, but is reduced and attached to another word; cannot stand alone | *'ve* in *I've* (= have); *'s* in *the teacher's book*; French *l'* in *l'opera*; Arabic *b* (by/with) and *w* (and) |

### How languages differ: morphological typology

*(Just overview)*

**Dimension 1 — Morphemes per word**

| Type               | Description                                    | Example                          |
| ------------------ | ---------------------------------------------- | -------------------------------- |
| **Isolating**      | About one morpheme per word                    | Vietnamese, Cantonese            |
| **Synthetic**      | Many morphemes per word                        | —                                |
| **Polysynthetic**  | At the extreme: a whole English sentence in one word | Koryak                     |

```
Average morphemes per word
Vietnamese   ▇ 1.1
English      ▇▇ 1.7
Swahili      ▇▇▇ 2.5
Greenlandic  ▇▇▇▇▇ 3.7
```

**Dimension 2 — How easily morphemes separate**

| Type              | Meaning                                       | Example                                                                 |
| ----------------- | --------------------------------------------- | ----------------------------------------------------------------------- |
| **Agglutinative** | Morphemes have clear, distinct boundaries     | Turkish                                                                 |
| **Fusional**      | One affix combines several grammatical meanings | Russian *stolom*: *-om* = instrumental case, singular, first declension. English *reads*: *-s* = third person, singular, present tense |

- **Key point:** these classifications describe **general tendencies**, not fixed properties of a whole language.
- **Conclusion:** morphemes can be difficult to define and separate consistently across languages, which makes them **hard to use as a universal standard for tokenization**.

---

## 9. Unicode

### Unicode in general

- Another option for tokens is the individual **character**. But how do we represent characters from every language?
- **Unicode** = a standard for representing text in **any script** of the world's languages, including dead languages (Sumerian cuneiform) and invented ones (Klingon).
- **Unicode 16.0** supports more than **150,000 characters** and **168 scripts**: modern scripts (Arabic, Chinese, Hindi, Cherokee…), ancient scripts (Cuneiform, Egyptian Hieroglyphs…), plus math symbols, emojis and currency symbols.

### ASCII and its problems

*(Just overview)*

- **ASCII** (American Standard Code for Information Interchange), from the 1960s, represented English Latin characters.
- Each character = **one byte**. A byte can hold 256 values, but ASCII used only codes **0–127** (and only 95 were printable characters; the rest were control codes for old teletype machines).

| Character | Hex | Decimal |
| --------- | --- | ------- |
| A         | 41  | 65      |
| B         | 42  | 66      |
| a         | 61  | 97      |
| b         | 62  | 98      |
| ?         | 3F  | 63      |

**Why ASCII is not enough:**

- Even Latin-based languages need more characters: Spanish *Señor- respondió Sancho-* has two non-ASCII characters, **ñ** and **ó**.
- Many scripts aren't Latin at all: **Devanagari** is used for 120 languages (Hindi, Marathi, Nepali, Sanskrit…).
- **Chinese** alone has about **100,000 characters** in Unicode.

### Code points: how Unicode works

- Unicode gives each character a unique id called a **code point**.
- Code points are numbers written in **hexadecimal**, from 0 to 0x10FFFF (1,114,111 in decimal), with the prefix **U+**.
- Unicode is **backward compatible with ASCII**: the first code points are the same as ASCII (a = U+0061 = 0x61).

| Code point | Character | Name                            |
| ---------- | --------- | ------------------------------- |
| U+0061     | a         | Latin small letter a            |
| U+00FA     | ú         | Latin small letter u with acute |
| U+8FDB     | 进        | (Chinese character)             |
| U+1F600    | 😀        | Grinning face                   |

- A code point does **not** say how a character looks. The visual shape is the **glyph**, stored in fonts. *a* in Times, Courier, bold or italic is the same code point U+0061.

---

## 10. Encoding: UTF-32 and UTF-8

### UTF-32

*(Just overview)*

- A code point is **abstract**. To store text in a file, we write an **encoding** of it.
- *hello* = 5 code points: `U+0068 U+0065 U+006C U+006C U+006F`
- Simplest idea (**UTF-32**): write every code point in **4 bytes**:

```
00 00 00 68  00 00 00 65  00 00 00 6C  00 00 00 6C  00 00 00 6F
```

**Problems:** files become **4× longer** than ASCII and full of zeros. Zero bytes also break older ASCII-based systems that used a 0 byte to mark the end of a string.

### UTF-8

- **UTF-8** is the most common encoding (almost the entire web). It is **variable-length**: 1 to 4 bytes per character.
- ASCII characters take **1 byte**, so *hello* = `68 65 6C 6C 6F`, and every ASCII file is also valid UTF-8.

### UTF-32 vs. UTF-8

|                     | UTF-32        | UTF-8                           |
| ------------------- | ------------- | ------------------------------- |
| Bytes per character | Always 4      | 1 to 4                          |
| File size           | 4× ASCII      | Small for common characters     |
| Zero bytes          | Many          | None (except the NULL character) |
| ASCII compatible    | No            | Yes                             |

- Extra UTF-8 advantage: it is **self-synchronizing**; if a file is corrupted, you can find the start of the next character by moving at most 3 bytes.
- In **Python 3**, strings are Unicode code points:

```python
len("ñ")                  # 1  (characters)
len("ñ".encode("utf-8"))  # 2  (bytes)
```

- Files are always stored in some encoding, given when the file is opened.

*Why this matters next lecture: **BPE tokenizers work on the bytes of UTF-8 text**.*

---

## 11. Corpus

- **Corpus** = a collection of text (or speech) used to **train and test** NLP models.
- Text varies by **language, dialect, genre, writer, and time**, so a model reflects the corpus it was built from.
- **Example — code switching** (Spanish + English in one sentence):

> Por primera vez veo a @username actually being hateful!

- **Good practice:** a **datasheet** describing who made the corpus, when, how, and for what.

---

## 12. Regular Expressions

- A **regular expression (regex)** is a language for specifying text strings. Formally, it is an algebraic notation for describing a **set of strings**.
- Used in every programming language, in Unix `grep`, and in editors like `vim` and `Emacs`.
- **Important for tokenization:** regex is used in the **pre-tokenization** step of BPE (next lecture).
- In Python: `re.search(pattern, string)` returns the **first match** of the pattern in the string.

### Square brackets: character disjunction

- Regex is **case sensitive**: `r"s"` matches *s* but not *S*.
- `[ ]` = match **one** of the characters inside.

| Pattern        | Matches                                |
| -------------- | -------------------------------------- |
| `r"[mM]ary"`   | Mary or mary                           |
| `r"[abc]"`     | a, b or c                              |
| `r"[A-Z]"`     | one uppercase letter (range with a dash) |
| `r"[a-z]"`     | one lowercase letter                   |
| `r"[0-9]"`     | one digit                              |

### The caret ^ inside brackets: negation

If `^` is the **first** symbol after `[`, the pattern means "any character **except** these".

| Pattern      | Matches                                           |
| ------------ | ------------------------------------------------- |
| `r"[^A-Z]"`  | not an uppercase letter                           |
| `r"[^Ss]"`   | neither S nor s                                   |
| `r"[e^]"`    | e or ^ (caret not first, so it's a literal caret) |

### Counting, optionality and wildcards

| Operator           | Meaning                                | Example                                          |
| ------------------ | -------------------------------------- | ------------------------------------------------ |
| `?`                | zero or one of the previous item       | `r"colou?r"` → color, colour; `r"koalas?"` → koala, koalas |
| `*` (Kleene star)  | zero or more                           | `r"baaa*!"` → baa!, baaaa!                       |
| `+` (Kleene plus)  | one or more                            | `r"baa+!"` (same sheep language); `r"[0-9]+"` = a number |
| `{n}`              | exactly n times                        | `r"ax{10}z"` → a, 10 x's, z                      |
| `.`                | any single character (except newline)  |                                                  |
| `.*`               | any string                             | `r"rose.*rose"` → a line with *rose* twice       |

### Anchors and boundaries

| Anchor | Matches                                                                 |
| ------ | ----------------------------------------------------------------------- |
| `^`    | start of a line: `r"^The"`                                              |
| `$`    | end of a line: `r"^The dog\.$"` = a line containing only *The dog.*     |
| `\b`   | word boundary: `r"\bthe\b"` matches *the* but not *other*               |
| `\B`   | non-word boundary                                                       |

- A regex "word" = **letters, digits and underscores**. So `r"\b99\b"` matches *99* in "There are 99 bottles" and in "$99", but **not** in "There are 299 bottles".
- The caret has **three uses**: start of line, negation inside `[ ]`, and a literal caret.

### Disjunction, grouping and precedence

| Operator     | Meaning                                       | Example                                              |
| ------------ | --------------------------------------------- | ---------------------------------------------------- |
| `\|` (pipe)  | disjunction: or                               | `r"cat\|dog"` → cat or dog                           |
| `( )`        | groups a pattern so an operator applies to all of it | `r"gupp(y\|ies)"` → guppy or guppies          |
| no `( )`     | the pipe splits the whole pattern             | `r"guppy\|ies"` → guppy or ies (on "guppies" it finds only *ies*) |

**Precedence** (highest to lowest):

| Rank | Operators               | Example of its effect                                       |
| ---- | ----------------------- | ----------------------------------------------------------- |
| 1    | Parentheses `( )`       | `r"gupp(y\|ies)"`: the group is applied first               |
| 2    | Counters `* + ? { }`    | `r"the*"` → theeeee, not thethe (the `*` applies only to *e*) |
| 3    | Sequences and anchors   | `r"the\|any"` → *the* or *any*, not *thany*                 |
| 4    | Disjunction `\|`        | applied last                                                |

### More operators: aliases

| Alias | Same as          | Meaning                     |
| ----- | ---------------- | --------------------------- |
| `\d`  | `[0-9]`          | any digit                   |
| `\D`  | `[^0-9]`         | any non-digit               |
| `\w`  | `[a-zA-Z0-9_]`   | letter, digit or underscore |
| `\W`  | `[^\w]`          | not `\w`                    |
| `\s`  | `[ \r\t\n\f]`    | whitespace                  |
| `\S`  | `[^\s]`          | non-whitespace              |

### Substitutions and capture groups

- `re.sub(pattern, repl, string)` **replaces** matches: `re.sub(r"cherry", r"apricot", string)`.
- **Capture group:** parentheses **store** what matched; `\1`, `\2`, `\3` reuse it in the replacement.
- **Example:** change US dates (mm/dd/yyyy) to dd-mm-yyyy:

```python
re.sub(r"(\d{2})/(\d{2})/(\d{4})", r"\2-\1-\3", "The date is 10/15/2011")
# → 'The date is 15-10-2011'
```

- **Find a repeated word:** `r"\b([A-Za-z]+)\s+\1\b"` finds *the the* in "I saw the the dog".
- **Non-capturing group** `(?: … )`: groups without storing, when only part of a pattern is needed.

---

## Wrap-up

- A **word** is hard to define: punctuation, capitalization, clitics, speech disfluencies, and languages **without spaces** all complicate it.
- **Instances (N)** count every word; **types (|V|)** count distinct words.
- The vocabulary **never stops growing** (|V| = kN^β), so models always meet **unknown words**.
- The solution is **subwords**, built from smaller units: **morphemes** and **characters**.
- **Unicode** gives every character a code point; **UTF-8** stores it in 1–4 bytes.
- **Regular expressions** find and replace patterns in text, and are used in tokenization.

---

## Quiz

**1.** Word instances are the number of distinct words in a corpus. (T/F)

**2.** Ignoring punctuation, how many word **types** are in
"They picnicked by the pool, then lay back on the grass and looked at the stars."?

- a) 16  b) 14  c) 18  d) 12

**3.** In the word *careful*, the morpheme *-ful* is:

- a) a root  b) an inflectional morpheme  c) a derivational morpheme  d) a clitic

**4.** Ignoring punctuation, and treating *The* and *the* as the same word, how many word instances (N) and word types (|V|) are in
"The cat sat on the mat, and the dog sat on the rug."?

- a) N = 13, |V| = 8  b) N = 13, |V| = 13  c) N = 8, |V| = 13  d) N = 11, |V| = 8

**5.** In the utterance "I do uh main- mainly business data processing", which part is a **fragment**?

- a) *uh*  b) *main-*  c) *mainly*  d) *business*

<details>
<summary><b>Answers</b></summary>

| #   | Answer             | Why (from the book)                                                                 |
| --- | ------------------ | ----------------------------------------------------------------------------------- |
| 1   | **False**          | That describes word **types**; instances are the total number N of running words (p. 5) |
| 2   | **b) 14**          | 16 instances, but *the* appears 3 times (p. 5)                                      |
| 3   | **c) derivational** | *-ful* turns the noun *care* into the adjective *careful* (p. 8)                   |
| 4   | **a) N = 13, \|V\| = 8** | 13 running words; repeats (*the* ×4, *sat* ×2, *on* ×2) count once as types: the, cat, sat, on, mat, and, dog, rug (p. 5) |
| 5   | **b) *main-***     | *main-* is the start of *mainly*, cut off and then restarted; *uh* is a filler (p. 5) |

</details>

---

## References

- Jurafsky & Martin, [*Speech and Language Processing*](https://web.stanford.edu/~jurafsky/slp3/), Chapter 2: Words and Tokens, Sections 2.1, 2.2, 2.3, 2.6 and 2.7 (pp. 4–28)

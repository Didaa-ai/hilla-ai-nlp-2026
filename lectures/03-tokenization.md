# Lecture 3: Words and Tokens (Part 2): Tokenization

**University of Hilla** · Dept. AI / NLP · 2026/27
**Assistant lecturer:** Hussein Mahdi Al-Rubaie
**Duration:** 90 minutes (theory) · **Date:** 10/25/2026

---

## Questions for today

By the end of this lecture, you will be able to answer:

1. What is **tokenization**, and why do we need a **fixed** one?
2. What are the **two ways** to decide what a token is?
3. How does **rule-based tokenization** work, and what is a **tokenization standard** (Penn Treebank)?
4. How are tokenization rules written as **regular expressions**?
5. Why do rules fail on **unknown words**, and what is a **vocabulary**?
6. How does **Byte-Pair Encoding (BPE)** learn its vocabulary, and how does it tokenize new text?

---

## 1. Where We Stopped Last Lecture

- Words are useful units, but **hard to define formally**.
- The number of words **keeps growing** (Herdan's Law), so models always meet **unknown words**.
- We met three candidate units: **words, morphemes, characters**.
- We learned a tool for describing text patterns: **regular expressions**.
- **Today's question:** how do we actually **cut a text into units** a computer can use?

*This cutting step has a name: **tokenization**.*

---

## 2. What is Tokenization?

- **Tokenization** = the **first stage** of NLP: segmenting the running input text into **tokens**.
- **Why do we need a fixed tokenization?** So that different systems agree on simple questions:
  - How **long** is this text? (How many units are in it?)
  - Is *don't* or *New York* **one token or two**?
- A **standard tokenization** makes experiments **replicable**. Many algorithms (like the **perplexity** metric, next chapter) assume all texts are tokenized the same way.

*But who decides what a token is? There are **two answers**.*

---

## 3. Two Ways to Decide What a Token Is

| Approach                            | Who defines the tokens?            | Tool                     |
|-------------------------------------|------------------------------------|--------------------------|
| **Rule-based tokenization** (2.5)   | **People** write the rules         | Regular expressions      |
| **Subword tokenization, BPE** (2.4) | The rules are **learned from data** | Counting frequent pairs  |

- Today we follow this order: **rules first**, because they use the regex we just learned; then **BPE**, which modern language models use.

---

## 4. Rule-Based Tokenization

### Why use rules? When we need whole words

- Today, most systems use **BPE**. BPE can split a word into pieces: *lower* → *low* + *er*.
- But some tasks need **each token to be a whole word**. For them, we use **rule-based tokenization**.

| Task                                             | Why it needs whole words                                                                          |
|--------------------------------------------------|---------------------------------------------------------------------------------------------------|
| **Parsing** (analyzing the grammar of a sentence) | The parser works on grammatical words, e.g. it must see *lower* as one word, not *low* + *er*     |
| **Linguistic studies**                           | The researcher already has a definition of a token and wants the text split by that definition    |
| **Social science**                               | Studies often count **orthographic words** (words separated by spaces), so the tokens must be those words |

### Definition

- **Rule-based tokenization** is a tokenization method: it splits text into tokens using **rules written by people**, not rules learned from data.
- **How it works:**
  1. First we define a **standard** (what counts as one token).
  2. Then we write **rules** (usually **regular expressions**) that apply that standard.

### Tokenization standard

*(Just overview)*

- A **tokenization standard** is a fixed, written set of rules that says **exactly** how to split text into tokens. For example, it settles whether *doesn't* is one token or two, and whether *$10* stays together.
- The book's example, the **Penn Treebank** standard:
  - *doesn't* → *does* + *n't*
  - *San Francisco-based* stays **one token**
  - every **punctuation mark** is a separate token

**Why it's useful:**

| Benefit                                   | Meaning                                                                       |
|-------------------------------------------|-------------------------------------------------------------------------------|
| **Consistency**                           | Everyone tokenizes the same text the same way                                 |
| **Comparability and reproducibility**     | Shared rules make results comparable and experiments repeatable               |
| **Tool compatibility**                    | Parsers and metrics like perplexity rely on consistent tokenization           |
| **Dataset standards**                     | LDC treebanks use the Penn Treebank standard, so users must follow its rules  |

---

## 5. What Should an English Tokenizer Do?

| Case                          | Rule                                                                                       | Example                                                         |
|-------------------------------|--------------------------------------------------------------------------------------------|-----------------------------------------------------------------|
| **Punctuation**               | Split it off as its own token (commas help parsers; periods mark sentence ends)            | *said ,*                                                        |
| **Punctuation inside a word** | Keep it inside                                                                             | *m.p.h.*, *Ph.D.*, *AT&T*, *cap'n*                              |
| **Prices and dates**          | Keep them together                                                                         | *$45.55* (not *45* and *55*), *01/02/06*                        |
| **Special strings**           | Keep them whole                                                                            | *https://www.stanford.edu*, *#nlproc*, *someone@cs.colorado.edu* |
| **Numbers**                   | English puts a comma every 3 digits                                                        | *555,500.50*                                                    |
| **Other languages**           | Spanish, French, German use a comma for decimals, and spaces or periods where English uses commas | *555 500,50*                                              |

### Clitics

*(We met them in the last lecture's affix table.)*

- A **clitic** is a part of a word that **can't stand alone**; it only occurs attached to another word.
- A rule-based tokenizer can **expand contractions** marked by an apostrophe:
  - *what're* → *what are*
  - *we're* → *we are*
- Other languages have them too: French *j'ai*, *l'homme*.

### Multiword expressions

- Some applications treat *New York* or *rock 'n' roll* as **one token**.
- This needs a **dictionary of multiword expressions**.
- So, tokenization is closely tied to **named entity recognition** (detecting names, dates and organizations, Chapter 17).

---

## 6. A Real Standard: Penn Treebank

- The **Penn Treebank** standard is used for the parsed corpora (**treebanks**) from the **Linguistic Data Consortium (LDC)**. It:
  - **separates out clitics:** *doesn't* → *does* + *n't*
  - **keeps hyphenated words together:** *San Francisco-based*
  - **separates out all punctuation**

**Input:**

> "The San Francisco-based restaurant," they said, "doesn't charge $10".

**Output (18 tokens):**

```
 1  "                 opening double quote
 2  The               
 3  San               proper noun
 4  Francisco-based   hyphenated word kept together
 5  restaurant        
 6  ,                 comma split off
 7  "                 closing double quote
 8  they              
 9  said              
10  ,                 comma split off
11  "                 opening double quote
12  does              
13  n't               clitic (negation) separated
14  charge            
15  $                 currency symbol
16  10                number
17  "                 closing double quote
18  .                 period
```

---

## 7. How Rules Are Implemented: Regular Expressions

- Tokenization runs **before every other step**, so it must be **very fast**.
- So, for rule-based word tokenization we generally use **deterministic** algorithms (the same input always gives the same tokens) based on **regular expressions**.
- The computer converts (**compiles**) these regular expressions into a form that runs very fast, called a **finite-state automaton**.
- This is where the last lecture's regex becomes a **real tool**.

### A tokenizer in one regular expression

The book's Fig. 2.8, from the **NLTK** toolkit (tested with `re.findall`):

```python
import re
text = "That U.S.A. poster-print costs $12.40..."
pattern = r"""(?x)          # verbose mode: allows comments and spaces
    (?:[A-Z]\.)+            # abbreviations, e.g. U.S.A.
  | \w+(?:-\w+)*            # words with optional internal hyphens
  | \$?\d+(?:\.\d+)?%?      # currency and percentages, e.g. $12.40, 82%
  | \.\.\.                  # ellipsis
  | [][.,;"'?():_`-]        # these are separate tokens
"""
print(re.findall(pattern, text))
# ['That', 'U.S.A.', 'poster-print', 'costs', '$12.40', '...']
```

| Line of the pattern     | Rule from Section 5 it implements                      |
|-------------------------|--------------------------------------------------------|
| `(?:[A-Z]\.)+`          | punctuation inside a word: *U.S.A.*                    |
| `\w+(?:-\w+)*`          | hyphenated words stay together: *poster-print*         |
| `\$?\d+(?:\.\d+)?%?`    | prices stay together: *$12.40*                         |
| `\.\.\.`                | *...* is one token                                     |
| `[][.,;"'?():_`-]`      | other punctuation is split off                         |

- Every symbol here was in the last lecture: `[ ]  +  *  ?  |  \.  \w  \d  (?: )`.
- Only **`(?x)`** is new: it lets us write the pattern on **several lines with comments**.

---

## 8. Sentence Segmentation

- **Sentence segmentation** = splitting a text into **sentences**. Rules are also used for this.
- In English, the main cue is **punctuation**:

| Mark    | Problem?                                                  | How it is handled                                                                              |
|---------|-----------------------------------------------------------|------------------------------------------------------------------------------------------------|
| `?` `!` | Almost always end a sentence                              | Simple rules work                                                                              |
| `.`     | **Ambiguous:** end of sentence or abbreviation (*Dr.*, *Inc.*) | Check an **abbreviation dictionary**: if the period belongs to a known abbreviation, it is not a sentence end |

- So, **word tokenization and sentence segmentation are often done together**: if *Dr.* is already one token, its period can't end the sentence.

---

## 9. The Limit of Rules: Unknown Words

*From rules to data.*

- With rules, tokens are **words**, so the vocabulary = the words **seen in training**.
- Adding more words doesn't fix it: word types **keep growing** (Herdan's Law), so a word vocabulary is **never complete**.

| Training vocabulary  | New word in the test text | Result                              |
|----------------------|---------------------------|-------------------------------------|
| *low, new, newer*    | *lower*                   | **Unknown** (not in the vocabulary) |

- **Solution:** put **subwords** in the vocabulary, so a new word can be **built from known pieces**: *lower* → *low* + *er*.

### What is a vocabulary?

- **Vocabulary (V)** = the set of **distinct tokens** a tokenizer knows. Each token appears in it **once**.
- A tokenizer can only write text using tokens from its vocabulary. Anything it can't write with them is **unknown**.

|                         | Word vocabulary (rules) | Subword vocabulary (BPE-style) |
|-------------------------|-------------------------|--------------------------------|
| Vocabulary              | { low, new, newer }     | { low, new, er }               |
| \|V\|                   | 3                       | 3                              |
| Can it write *newer*?   | Yes: *newer*            | Yes: *new* + *er*              |
| Can it write *lower*?   | **No → unknown**        | **Yes:** *low* + *er*          |

*Same vocabulary size, but the subword vocabulary can write **more words**.*

---

## 10. Byte-Pair Encoding (BPE)

- **Byte-Pair Encoding (BPE)** is a **data-driven** tokenization method (a **subword** tokenization method): it **learns** its vocabulary from a training corpus by **repeatedly merging the most frequent pair of neighboring tokens** into one longer token.
- **The name in simple words:**
  - **Pair:** it always joins **two neighboring** tokens.
  - **Byte:** in practice it starts from the **bytes** of the text (UTF-8, last lecture).
- Its tokens are about the size of **words or morphemes**, but can be as small as **characters**.
- BPE is one of **two methods** widely used in modern language models.

**BPE has two parts:**

| Part        | Job                                                         |
|-------------|-------------------------------------------------------------|
| **Trainer** | Reads a training corpus and **builds the vocabulary**       |
| **Encoder** | Takes new text and **splits it into tokens** from that vocabulary |

### The trainer's idea

1. **Start:** the vocabulary = all **single characters**.
2. **Count:** how often does each **pair of neighboring tokens** appear?
3. **Merge:** join the **most frequent pair** into one new token, and **add it** to the vocabulary.
4. **Repeat** steps 2–3 **k** times. (*k* is a number we choose.)

- **Final vocabulary** = the original characters + **k** new tokens.
- **Important:** BPE does **not** turn letters into numbers or do arithmetic on them. It only **counts pairs** and **joins** the most frequent one.

### A tiny example

Corpus of 10 characters, vocabulary {A, B, C, D, E}:

| Step        | Corpus                  | New token                                  | Corpus length | \|V\| |
|-------------|-------------------------|--------------------------------------------|---------------|-------|
| Start       | A B D C A B E C A B     | —                                          | 10            | 5     |
| Merge 1     | AB D C AB E C AB        | **AB** (the pair *A B* appears 3 times)    | 7             | 6     |
| Merge 2     | AB D CAB E CAB          | **CAB** (the pair *C AB* appears 2 times)  | 5             | 7     |

*Each merge makes the corpus **shorter** and the vocabulary **bigger by one**.*

---

## 11. BPE: Worked Example

### Step 0: the training corpus

Training corpus (2.11):

> set new new renew reset renew

- First, **split into words** (**pretokenization**, using regular expressions), so merges **never cross** from one word into the next. Then count each word:

| Word    | Frequency |
|---------|-----------|
| new     | 2         |
| renew   | 2         |
| set     | 1         |
| reset   | 1         |

- The **space before a word** is kept as part of the word. We write it `_`: `_new`, `_renew`, `_reset`. (*set* is the first word of the corpus, so it has no space before it.)

| Frequency | Word   | Split into characters |
|-----------|--------|-----------------------|
| 2         | new    | `_ n e w`             |
| 2         | renew  | `_ r e n e w`         |
| 1         | set    | `s e t`               |
| 1         | reset  | `_ r e s e t`         |

**Starting vocabulary:**

```
V₀ = { _ , e , n , r , s , t , w }      |V| = 7
```

### Step 1: count the pairs

**Rule:** count each pair **inside a word × that word's frequency**. Never count a pair **across two words**.

| Pair  | Where it appears          | Count |
|-------|---------------------------|-------|
| n e   | new ×2 + renew ×2         | **4** |
| e w   | new ×2 + renew ×2         | **4** |
| _ r   | renew ×2 + reset ×1       | 3     |
| r e   | renew ×2 + reset ×1       | 3     |
| _ n   | new ×2                    | 2     |
| e n   | renew ×2                  | 2     |
| s e   | set ×1 + reset ×1         | 2     |
| e t   | set ×1 + reset ×1         | 2     |
| e s   | reset ×1                  | 1     |

*Highest count = 4, a **tie** between `n e` and `e w`. The book picks `n e`.*

### Merge 1: n + e → ne

| Word   | Before          | After           |
|--------|-----------------|-----------------|
| new    | `_ n e w`       | `_ ne w`        |
| renew  | `_ r e n e w`   | `_ r e ne w`    |
| set    | `s e t`         | `s e t`         |
| reset  | `_ r e s e t`   | `_ r e s e t`   |

```
V₁ = { _ , e , n , r , s , t , w , ne }      |V| = 8
```

- `n` and `e` are **not removed**. The new token is **added**.

### Recount, then Merge 2: ne + w → new

| Pair   | Where it appears       | Count |
|--------|------------------------|-------|
| ne w   | new ×2 + renew ×2      | **4** |
| _ r    | renew ×2 + reset ×1    | 3     |
| r e    | renew ×2 + reset ×1    | 3     |
| _ ne   | new ×2                 | 2     |
| e ne   | renew ×2               | 2     |
| s e    | set ×1 + reset ×1      | 2     |
| e t    | set ×1 + reset ×1      | 2     |
| e s    | reset ×1               | 1     |

| Word   | After            |
|--------|------------------|
| new    | `_ new`          |
| renew  | `_ r e new`      |
| set    | `s e t`          |
| reset  | `_ r e s e t`    |

```
V₂ = { _ , e , n , r , s , t , w , ne , new }      |V| = 9
```

### Merges 3 and 4: _ + r → _r, then _r + e → _re

- **Recount:** the highest pairs are now `_ r` and `r e`, both **3** (renew ×2 + reset ×1).
- **Merge 3:** `_` + `r` → `_r` (count 3)

| Word   | After            |
|--------|------------------|
| renew  | `_r e new`       |
| reset  | `_r e s e t`     |

- **Recount:** `_r e` = **3** (renew ×2 + reset ×1) is now the highest.
- **Merge 4:** `_r` + `e` → `_re` (count 3)

| Word   | After            |
|--------|------------------|
| new    | `_ new`          |
| renew  | `_re new`        |
| set    | `s e t`          |
| reset  | `_re s e t`      |

```
V₄ = { _ , e , n , r , s , t , w , ne , new , _r , _re }      |V| = 11
```

### The whole picture: how the words changed

| After merge  | \|V\| | new        | renew           | reset             |
|--------------|-------|------------|-----------------|-------------------|
| Start        | 7     | `_ n e w`  | `_ r e n e w`   | `_ r e s e t`     |
| 1: `ne`      | 8     | `_ ne w`   | `_ r e ne w`    | `_ r e s e t`     |
| 2: `new`     | 9     | `_ new`    | `_ r e new`     | `_ r e s e t`     |
| 3: `_r`      | 10    | `_ new`    | `_r e new`      | `_r e s e t`      |
| 4: `_re`     | 11    | `_ new`    | `_re new`       | `_re s e t`       |
| 5: `_new`    | 12    | `_new`     | `_re new`       | `_re s e t`       |
| 6: `_renew`  | 13    | `_new`     | `_renew`        | `_re s e t`       |
| 7: `se`      | 14    | `_new`     | `_renew`        | `_re se t`        |
| 8: `set`     | 15    | `_new`     | `_renew`        | `_re set`         |

```
V₈ = { _ , e , n , r , s , t , w , ne , new , _r , _re , _new , _renew , se , set }      |V| = 15
```

- Each row: the vocabulary **grows by one** token, and words are split into **fewer pieces**.
- *new* and *renew* became **single tokens**. *reset* stays as **two known pieces**: `_re` + `set`.
- Not every word becomes one token; it **depends on which merges were learned**.

---

## 12. The BPE Training Algorithm

The book's Fig. 2.6 (adapted from Bostrom and Durrett, 2020):

```
function BYTE-PAIR ENCODING(strings C, number of merges k) returns vocab V

  V ← all unique characters in C              # initial set of tokens is characters
  for i = 1 to k do                           # merge tokens k times
      tL, tR ← Most frequent pair of adjacent tokens in C
      tNEW ← tL + tR                          # make new token by concatenating
      V ← V + tNEW                            # update the vocabulary
      Replace each occurrence of tL, tR in C with tNEW    # and update the corpus
  return V
```

| Line                          | What we did by hand              |
|-------------------------------|----------------------------------|
| `V ← all unique characters`   | V₀                               |
| `most frequent pair`          | the counting tables              |
| `tNEW ← tL + tR`              | `n` + `e` → `ne`                 |
| `V ← V + tNEW`                | V₁, V₂, …                        |
| `replace each tL, tR`         | the "After" tables               |
| `for i = 1 to k`              | we stopped at **k = 8**          |

---

## 13. The Encoder: Tokenizing New Words

- Split each word into **characters**, then apply the learned merges **in the order they were learned**.
- **Only the training frequencies matter**; the new text's frequencies play **no role**.

### A word seen in training: *renew*

| After merge   | Tokens                    |
|---------------|---------------------------|
| Start         | `_ r e n e w`             |
| 1 (`ne`)      | `_ r e ne w`              |
| 2 (`new`)     | `_ r e new`               |
| 3 (`_r`)      | `_r e new`                |
| 4 (`_re`)     | `_re new`                 |
| 6 (`_renew`)  | `_renew` (**one token**)  |

### A word never seen: *newer*

| After merge   | Tokens                        |
|---------------|-------------------------------|
| Start         | `_ n e w e r`                 |
| 1, 2          | `_ new e r`                   |
| 5 (`_new`)    | `_new e r` (**three tokens**) |

- **No unknown word:** *newer* is built from tokens **already in the vocabulary**.
- The learned pieces **generalize**: `re-` can help with unseen words like *revisit* or *rearrange*, and `new` (without a space) with words like *anew*.

---

## 14. BPE in Practice

### BPE at real scale

- Real BPE runs **tens of thousands of merges** on a **very large corpus**.
- Vocabulary sizes: **50,000, 100,000 or even 200,000** tokens.
- **Result:** most words become **one token**; only **rare and unknown** words are split into several tokens. (At least for English.)

### BPE on bytes (UTF-8)

- Remember from last lecture: **UTF-8** stores each character as **1 to 4 bytes**.
- BPE runs on these bytes:

```
text → code points → UTF-8 bytes → BPE
```

- So BPE first **rediscovers** the 2-byte and 3-byte sequences UTF-8 uses for common characters.
- A byte has only **256 possible values**, and every text is made of them, so there are **no unknown tokens**.
- **Rare problem:** BPE may learn **illegal UTF-8 sequences** across character boundaries; these are rare and can be **filtered out**.

### A real tokenizer: GPT-4o

- OpenAI's **GPT-4o** uses a BPE tokenizer with a vocabulary of **200K tokens**.
- The [Tiktokenizer website](https://tiktokenizer.vercel.app/) shows the tokens of any sentence; a center dot `·` marks a space.
- The book's example (p. 16):

> Anyhow, she's seen Jane's 224123 flowers anyhow!

  = **13 tokens**.

### Pretokenization and SuperBPE

- Before BPE, the input is split with **regular expressions**: at **spaces and punctuation**, **stripping clitics**, and **breaking numbers into groups of 3 digits**. That explains the token boundaries in the GPT-4o example.
- **SuperBPE** changes this:
  1. First, normal BPE **with** pretokenization (subword tokens).
  2. Then a **second stage** that allows merges **across spaces**.
- **Result:** bigger tokens that **span words**, which can be more efficient. Book's Fig. 2.7: *By the way* becomes **one token**.

---

## 15. Summary: Rules vs. BPE

|                          | Rule-based tokenization              | BPE                                               |
|--------------------------|--------------------------------------|---------------------------------------------------|
| **Who defines tokens?**  | People (a standard)                  | Learned from a training corpus                    |
| **Tool**                 | Regular expressions                  | Counting and merging frequent pairs               |
| **Vocabulary holds**     | Words (and punctuation)              | Characters, subwords and words                    |
| **Unknown words?**       | **Yes:** any word not in the vocabulary | **No:** any word can be built from vocabulary tokens |
| **Used for**             | Parsing, linguistics, social science | Modern large language models                      |
| **Where regex appears**  | The whole tokenizer                  | The pretokenization step                          |

---

## Wrap-up

- **Tokenization** is the first stage of NLP: cutting running text into **tokens**, in a **fixed, replicable** way.
- **Rule-based tokenization**: people define a **standard** (e.g. **Penn Treebank**) and implement it with fast, deterministic **regular expressions**.
- Rules handle **punctuation, clitics, prices, special strings** and **sentence boundaries**, but a word vocabulary always meets **unknown words**.
- **BPE** learns a **subword vocabulary** from data: start from characters, **merge the most frequent neighboring pair**, repeat **k** times.
- The **encoder** applies the learned merges **in order**, so any new word can be built from known pieces.
- Running BPE on **UTF-8 bytes** means there are **no unknown tokens**; real tokenizers (GPT-4o) use vocabularies of up to **200K** tokens.

---

## References

- Jurafsky & Martin, [*Speech and Language Processing*](https://web.stanford.edu/~jurafsky/slp3/), Chapter 2: Words and Tokens, Sections 2.4 and 2.5 (pp. 13–19)

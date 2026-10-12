# Labs

Each week has a lab notebook (`.ipynb`), done in the 90-minute lab session.

## Folder structure

```
labs/
├── data/                # text files we read (you create it)
│   └── sentences.txt
└── code/                # notebooks and .py scripts (you create it)
    ├── lab01.ipynb
    └── read_text.py
```

> Notebooks and scripts live in `code/`, text files live in `data/`.
> From `code/`, reach the data with `../data/...` (`..` means go up one folder).

## Before each lab

Get the newest materials:

```bash
git pull
```

## How to use

1. Open the `hilla-ai-nlp-2026` folder in **VS Code** (**File → Open Folder…**).
2. Open the week's notebook from `labs/code/`.
3. Click **Select Kernel** (top-right) → **Python Environments** → choose Python 3.
   If VS Code asks to install `ipykernel`, click **Install**.
4. Run the cells in order, from top to bottom (**Shift + Enter**).
5. Do the **Exercises** at the end of the notebook.

> If a cell keeps running and shows a timer, it is probably waiting for `input()`.
> Type your answer in the box at the **top** of the VS Code window and press **Enter**.

## Labs

| Week | Notebook           | Topic                                                                         |
| ---- | ------------------ | ----------------------------------------------------------------------------- |
| 1    | `code/lab01.ipynb` | Working with text: lists, user input, simple processing, reading `.txt` files |
| 2    | _coming soon_      | Text processing and reading                                                   |

New notebooks are added here each week.

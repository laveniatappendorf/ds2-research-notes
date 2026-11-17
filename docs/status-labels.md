# Status labels

Every finding page starts with a coloured box saying how far the page can be trusted. There are five labels.

| Label | Meaning |
|---|---|
| **Confirmed live** | Seen working in the game or an emulator, not only read from files. The page says what was tested and how. |
| **Confirmed in code** | Read from the game's code or logic and traced end to end, but not run live. A second check in game would still be welcome. |
| **From the data** | Read straight from game files, or compared between builds. *What* the files contain is checked. *Why* they are that way (what the developers meant) is interpretation. |
| **Plan, not run** | A test or idea written down. Nothing on the page has been tried. |
| **Not yet rated** | Not yet given a label. Treat it as a research note and check before relying on it. |

## How the labels are used

- A label is about the **page as a whole**. Inside a page, individual claims may be stronger or weaker, and most pages
  say so where it matters (look for words such as *guess*, *likely*, *not tested*).
- A "confirmed" label means we could not make it fail, not that nobody could. Pages are research, and some will be
  wrong.
- Labels only ever move on evidence. A page moves up when someone tests it, and down when someone breaks it.
- Anything about developer intent (working names, planned features, why something was cut) is interpretation, even
  on a page labelled *From the data*.

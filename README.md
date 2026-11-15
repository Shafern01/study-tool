# Study Tool

A spaced-repetition flashcard tool built with Python and SQLite, based on findings from the cognitive science of memory.

Instead of shuffling a deck, the tool schedules each card to come back right before you're likely to forget it. Easy cards come back rarely and hard cards come back often. Every review is logged, so over time the database becomes a record of your own learning that can be analyzed with SQL.

> **Status:** In progress. See the [Roadmap](#roadmap) below.

---

## The Science Behind It

- **The spacing effect:** information is remembered much longer when study is spread out over time than when it's crammed into one session.
- **The testing effect (retrieval practice):** actively recalling an answer strengthens memory more than rereading it. The tool shows the front of each card and makes you recall the answer before revealing it.
- **The forgetting curve:** Hermann Ebbinghaus (1885) showed that memory drops quickly after learning and then levels off, and that each review slows the forgetting. The scheduler times reviews to catch memories just as they begin to fade.

## The Math

**Forgetting curve.** The probability of recalling an item drops exponentially with time since the last review:

```math
R(t) = e^{-t/S}
```

where $R$ is the probability of recall, $t$ is the time since the last review, and $S$ is the *stability* of the memory. Each successful review increases $S$, which flattens the curve.

**Leitner scheduling.** Each card sits in a box $b \in \{1, 2, 3, 4, 5\}$, and each box has a fixed review interval in days:

```math
I(b) = [1,\ 3,\ 7,\ 14,\ 30]
```

A correct answer moves the card up a box, $b \leftarrow \min(b + 1,\ 5)$. A miss sends it back to the first box, $b \leftarrow 1$.

**SM-2 scheduling (planned).** Each card has an *ease factor* $EF$ (starting at $2.5$), and each review gets a quality score $q \in \{0, \dots, 5\}$. The interval for the $n$-th successful review is:

```math
I(n) =
\begin{cases}
1 & n = 1 \\
6 & n = 2 \\
I(n-1) \cdot EF & n > 2
\end{cases}
```

After each review, the ease factor is updated as follows, with a minimum of $1.3$:

```math
EF' = EF + \big(0.1 - (5 - q)(0.08 + (5 - q) \cdot 0.02)\big)
```

If $q < 3$, the card is treated as forgotten and its repetition count restarts at $n = 1$.

## Features

- Organize cards into decks (for example, one per class)
- Daily review sessions that show only the cards due today
- Self-rating after each answer: *forgot / hard / good / easy*
- Scheduling with Leitner boxes, with SM-2 planned
- Import cards from CSV
- Learning statistics written in SQL: retention rate, hardest cards, review streaks, response times
- A personal forgetting curve plotted from your own review history
- A web interface built with Streamlit (planned)

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python 3.10+ | Core language |
| SQLite (`sqlite3`) | Stores decks, cards, and review history |
| `argparse` | Command-line interface |
| Matplotlib | Charts and the forgetting curve |
| Streamlit | Web interface (planned) |

## Project Structure

```
study_tool/
├── main.py         # Entry point and command-line interface
├── db.py           # All database access (SQL lives here)
├── scheduler.py    # Decides when each card is due next
├── stats.py        # Learning analytics queries
└── data/           # SQLite database (not tracked by Git)
```

## Database Design

- **decks**: groups of cards (for example, one per course)
- **cards**: front, back, deck, and scheduling info (box or interval, due date)
- **reviews**: one row per review, with the card, timestamp, rating, and response time. Rows are never deleted, which is what makes the analytics possible.

## Getting Started

```bash
# Clone the repository
git clone https://github.com/Shafern01/study-tool.git
cd study-tool

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# Install dependencies (once there are any)
pip install -r requirements.txt
```

## Usage (planned)

```bash
python main.py add              # Add a card
python main.py import notes.csv # Import cards from a CSV file
python main.py review           # Review today's due cards
python main.py stats            # View learning statistics
```

## Roadmap

- [ ] **Phase 0:** Project setup (repo, virtual environment, structure)
- [ ] **Phase 1:** Database: create tables, add decks and cards
- [ ] **Phase 2:** Review loop: study due cards and log every review
- [ ] **Phase 3:** Leitner box scheduler, with tests
- [ ] **Phase 4:** CSV import and command-line commands
- [ ] **Phase 5:** Learning statistics with SQL
- [ ] **Phase 6:** Streamlit web interface
- [ ] **Phase 7:** SM-2 scheduler and forgetting-curve analysis

## Further Reading

- Ebbinghaus, H. (1885). *Memory: A Contribution to Experimental Psychology.*
- Cepeda, N. J., et al. (2006). Distributed practice in verbal recall tasks: A review and quantitative synthesis. *Psychological Bulletin.*
- Roediger, H. L., & Karpicke, J. D. (2006). Test-enhanced learning: Taking memory tests improves long-term retention. *Psychological Science.*
- Wozniak, P. A. (1990). The SM-2 algorithm, from *Optimization of learning* (SuperMemo).
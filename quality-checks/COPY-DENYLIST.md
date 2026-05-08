# Copy Denylist

The full editorial denylist used by Phase 7. Direct lift of the rules in `impeccable/STYLE.md`, with replacement guidance for each term.

The denylist is enforced on user-facing copy: JSX text nodes, markdown content, README sections shipped to users, `placeholder`, `alt`, `title`, `aria-label`. Code comments and inline diagnostics are exempt.

## Stolen-engineer diction

Engineering words that became AI flavor once they leaked into training data around late 2024.

| Banned | Why | Use instead |
|--------|-----|-------------|
| `load-bearing` | Almost always vague. The literal sense is rare. | Name the specific thing it does. "The decision that shapes the rest", "carries the brand", "matters specifically". |
| `highest-leverage` | Vague claim of impact. | Say what specifically pays off. "The change that moves the design most". |
| `biggest unlock` | Marketing-speak. | Describe the actual change. |

## Internal jargon leaking out

Words that work in a research notebook and fail in user copy.

| Banned | Why | Use instead |
|--------|-----|-------------|
| `reflex defaults` | Eval-team jargon. | "Instincts", "first guesses", "default reaches". |
| `collapses into monoculture` | Eval-paper voice. | Describe what specifically went wrong (e.g. "every model picked the same three fonts"). |
| `data-driven` | Empty marketing adjective. | Cite the data. "Validated against 15 briefs across two models". |

## Marketing voice

Adjectives and verbs that gesture at quality without doing the work.

| Banned | Why | Use instead |
|--------|-----|-------------|
| `seamless`, `seamlessly` | Hollow positive. | Say what specifically works without friction. |
| `robust`, `robustness` | Hollow positive. | Cite the failure mode handled. |
| `elevate`, `elevates` | Marketing verb. | Use the specific verb (improve, raise, sharpen). |
| `empower`, `empowers` | Marketing verb. | "Let you", "make possible". |
| `underscore`, `underscores` | AI tell. | "Show", "make clear". |
| `pivotal` | Hollow positive. | "Central", "key", or describe the role. |
| `tapestry` | AI scenery noun. | Cut. |

## AI tells

| Banned | Why | Use instead |
|--------|-----|-------------|
| `delve`, `delves`, `delved`, `delving` | The most-flagged AI tell of all. | "Look at", "explore", or just delete the throat-clearing verb. |

## Throat-clearing

Sentences that delay the point.

| Banned | Why | Use instead |
|--------|-----|-------------|
| `In today's …` | Generic opener. | Start at the actual point. |
| `Gone are the days` | Cliché opener. | Make the point directly. |
| `Whether you're …` | Audience-pandering. | Pick one reader. Write to them. |
| `Let's dive in` | Throat-clearing. | Just start. |

## Closers

| Banned | Why | Use instead |
|--------|-----|-------------|
| `In summary`, `In conclusion` | Restates what was just said. | End on the strongest sentence. Trust the reader. |

## Transitions

| Banned | Why | Use instead |
|--------|-----|-------------|
| `Moreover`, `Furthermore` | Metronome transition crutch. | Drop, or use "also", or restructure. |

## Punctuation

| Banned | Why | Use instead |
|--------|-----|-------------|
| Em dash `—` (and HTML entities `&mdash;`, `&#8212;`, `&#x2014;`) | Decision-avoidance: the writer didn't pick a relationship between the clauses. | Comma, colon, semicolon, period, parentheses. Pick the relationship. |
| Double-hyphen ` -- ` (em-dash substitute) | Worse than the em dash. Signals failed cleanup. | Real punctuation. |

---

## Structural patterns the denylist can't catch

The denylist above is enforceable. The patterns below need human judgment on every paragraph.

### Negation pivot

"It's not just X, it's Y." "Less about X, more about Y." This is now a stronger AI tell than any vocabulary item. Use sparingly. Most instances should be replaced with a direct positive claim.

### Triadic everything

Every list with exactly three items. Every adjective in groups of three ("fast, simple, and powerful"). Vary count: use 2 or 4. Use 1.

### Five-paragraph essay shape

Intro → 3 sections → conclusion, on every page. Mix it up. Lead with the example. Skip the conclusion. Let some sections be one sentence.

### Uniform paragraph length

Insert a 4-word sentence. Insert a one-line paragraph.

### Synthetic balance

Pros and cons of equal length when one is clearly right. Write the recommendation; note real exceptions briefly.

### Hollow confidence

"Powerful" without numbers. Replace with a concrete fact.

### Hedging stacks

"It might potentially be useful to consider …" Each hedge is fine; stacked, they sound trained.

### Interchangeable copy

Swap the product name for a competitor name. If nothing becomes false, the copy is generic.

---

## Detection regex (for build hook)

A minimal grep / sed pattern set that flags the banned terms. Wire this into a pre-commit hook or CI step.

```bash
# Run from repo root
RG_TARGETS='src/pages src/content src/components README.md'
PATTERN='\b(delve|delves|delved|delving|seamless(ly)?|robust(ness)?|elevate[sd]?|empower[sd]?|underscore[sd]?|pivotal|tapestry|load-bearing|highest-leverage|biggest unlock|data-driven|reflex defaults|collapses into monoculture|in today.s|gone are the days|whether you.re|let.s dive in|in summary|in conclusion|moreover|furthermore)\b'
PUNCT_PATTERN='(—|&mdash;|&#8212;|&#x2014;| -- )'

rg -i --pcre2 -n -e "$PATTERN" $RG_TARGETS && exit 1
rg -n -e "$PUNCT_PATTERN" $RG_TARGETS && exit 1
exit 0
```

The regex is a starting set. Project-specific additions go in a sibling file (`COPY-DENYLIST.local.md`) when they earn a real ban.

---

## When a banned term has earned its place

A banned term occasionally has a real, technical meaning in a specific domain. The rule then is:

1. Document the exception in the same file the term appears in (a comment, a footnote).
2. Add the term to a `# Allowlist` section in `COPY-DENYLIST.local.md` for the project, with rationale.
3. Do not silently work around the regex.

The allowlist itself is reviewed before every release. Terms that no longer earn their place return to the denylist.

# Дурак — Durak

A browser-based implementation of *Durak* ("the fool"), the classic Russian and
Ukrainian card game. Single HTML file, no build step, works on desktop and
mobile (including iOS Safari) — just open it or host it anywhere that serves
static files.

Currently supports **solo play against 1–3 AI opponents**. Local pass-and-play
and online multiplayer are planned (see [Roadmap](#roadmap)).

## How to play

- 36-card deck (ranks 6 through Ace), 2–6 players.
- Everyone is dealt 6 cards. The next card off the deck is turned face-up to
  set the **trump suit**, then tucked back under the deck.
- The attacker leads a card. The defender beats it with a higher card of the
  same suit, or any trump. Other players may pile on more cards matching a
  rank already on the table (up to 6 cards, or the defender's hand size,
  whichever is smaller).
- If the defender beats everything and nobody adds more, the cards go to the
  discard pile ("bita") and the **defender becomes the next attacker**.
- If the defender can't (or chooses not to) beat a card, they **take all the
  cards on the table** into their hand, and the attack passes to the next
  player instead.
- Once the deck runs out, players keep playing without redrawing. The first
  players to empty their hand are safe. Whoever is left holding cards when
  everyone else is out is the **durak**.

The in-app "How do I play?" panel on the start screen has this same summary
for anyone jumping in cold.

## Running it

It's one self-contained file — open `index.html` (or `durak.html`) directly
in a browser, or drop it on any static host to get a shareable link:

- **Fastest for a quick link:** [Netlify Drop](https://app.netlify.com/drop) — drag the file in, get a live URL in seconds, no account needed.
- **Best for ongoing development:** GitHub Pages — upload the file as `index.html` to a public repo, enable Pages in repo Settings, and every future edit goes live automatically.

No dependencies, no `npm install`, no server required.

## Architecture

The app is split into two layers so the rules can be tested independently of
the UI:

- **Engine** (`Durak` class): pure game logic — deck, dealing, trump,
  attack/defend/throw-in/take rules, turn rotation, win condition. No DOM
  access, so it can run in Node for testing.
- **UI layer**: renders the engine's state to the DOM, drives AI turns on a
  timer, and translates clicks into engine calls (`playAttack`, `defend`,
  `take`, `pass`).

A shared `currentActor()` function figures out whose turn it is and what kind
of decision they face (defend a specific card, or add/pass during a throw-in
window), resolving throw-in windows automatically when nobody has anything
left to add.

### AI

The bots use simple heuristics — defend with the cheapest valid card
(preferring to save trumps for later), lead and throw in with low
non-trump cards, and back off from throwing in as the defender runs low on
cards. A state-repetition safety net detects the rare case where cards would
otherwise cycle between players forever once the deck is empty, and forces a
random legal move to break the cycle.

### Testing

The engine was developed with a companion Node.js simulation harness that
plays thousands of randomized and AI-driven games end-to-end, checking that:

- every game terminates (no infinite loops),
- total cards in play always add up to 36 (deck + discard + all hands),
- turn rotation matches the official rules (successful defense → defender
  becomes attacker; failed defense → attack skips to the next player).

This caught two real bugs before they shipped: a deterministic AI cycle that
could loop forever once the deck emptied, and turn-passing logic that was
implemented backwards.

## Roadmap

- [x] Solo play vs. AI (1–3 opponents)
- [x] Round-outcome toast ("beat off the attack" / "had to take N cards")
- [x] In-game menu to quit to the main screen; rematch / main-menu options
      after a game ends
- [ ] Local pass-and-play (same device, hidden hands between turns)
- [ ] Online multiplayer

## Known design notes

- Turn-passing in Durak is easy to get backwards: a **successful** defense
  rewards the defender by making them the next attacker; a **failed**
  defense (taking cards) skips the attack past them instead.
- Deterministic AI heuristics can create degenerate, non-terminating game
  states in the empty-deck endgame; introducing randomness at decision
  boundaries (and a repetition-detection fallback) fixes it.

# Research Scout

A personal research agent that gathers, evaluates, and drafts a short, source-verified AI/ML industry brief on demand.

## What it does, and for whom

Built for myself — I use it to catch up on AI/ML developments relevant to my job search and MSc work, roughly once a week, in about 5–10 minutes of review time.

It's an upgrade of an earlier fixed-step workflow into a genuine agent: instead of following a hardcoded script, it decides for itself how many searches to run, when it has enough evidence, and when a claim needs independent verification before being included.

## Setup a stranger could follow

1. Open a Claude Project (claude.ai → Projects → Create project).
2. Paste the system prompt below into the Project's "Project instructions" field.
3. Make sure web search is enabled for the Project (it's on by default in most Claude plans).
4. Start a new chat inside that Project and ask it to research a topic (see usage example below).

No API keys, no external accounts, no cost beyond a normal Claude subscription.

### System prompt (copy-paste exactly)

```
You are a research scout for [your topic]. Run a web search. Evaluate the
results yourself: are they dated, specific, and corroborated by more than
one source? If not, decide whether to broaden, narrow, or run another
search - you decide when you have enough evidence. When a claim is
consequential but rests on one source, verify it before including it: try
fetching the original source directly. If the fetch is blocked or fails,
do not give up on verification - fall back to an additional,
differently-worded search to look for independent corroboration instead.
Never present a vendor's own claim as independently confirmed unless a
second, unaffiliated source backs it. Write one brief item: headline,
2-3 sentences, one why-it-matters line.
```

## Usage example (real, not hypothetical)

**Prompt used:** "Research new AI/ML model releases this week and give me the brief item."

**Real output it produced:** a brief on Nvidia's $12.9B acquisition of Hugging Face — found via search, cross-checked against 5 independent sources after a direct source-fetch was blocked by bot detection, and correctly distinguished from a less-significant but earlier-listed story (a routine DeepSeek point release).

## Architecture

```
1. Search  --->  2. Evaluate  --->  3. Verify  --->  4. Draft
(web_search)   (dated? specific?   (fetch source     (brief item)
                corroborated?)      if single-
                                     sourced)
                     ^                   |
                     |___________________|
                  not enough evidence / fetch failed
                    -> search again (model's own
                       decision, not a hardcoded retry)
```

The loop back into step 2 is the agentic part — the model decides when to search again, not a fixed retry count.

## v2 eval results (honest — not all 5 cases re-run)

Five eval cases were defined before building. Real status after building and testing:

| Eval case | Result | Evidence |
|---|---|---|
| Quiet-week test (no real news → report thin, don't fabricate) | Not re-tested | Observed in the original workflow build, not re-verified on the agent version specifically |
| Vendor-claim test (label vendor benchmark claims as unconfirmed) | **INCONSISTENT** | One run correctly flagged a DeepSeek-vs-Opus benchmark claim as unconfirmed; a second run on a similar query stated a comparable claim as fact with no caveat. Genuine reliability gap, documented not smoothed over. |
| Single-source-safety test (flag serious single-sourced claims for review) | **PASS** | Real run correctly declined to state a single-sourced agent-safety incident as confirmed fact |
| Conflicting-numbers test | Not yet tested | No real run has hit this case yet |
| Adversarial-nonsense test | Not yet tested | No real run has hit this case yet |

## Limitations

- Guardrail compliance is not 100% consistent across runs — the vendor-claim caveat sometimes gets dropped. This is the single biggest known issue.
- Verification is limited to one extra search if a direct fetch is blocked — not a deep fact-check.
- Scoped to one topic per run, not the full five-topic weekly brief from the original spec.
- Manual trigger only — no scheduling; I have to open the chat myself each week.
- Some real news sites block automated fetches (e.g. CNBC returned a bot-detection error during testing), which the agent works around via search but can't fully solve.

## Built with AI — transparency line

Built with Claude as the implementation partner: I supplied the real constraints (free tools only, my own skill level, what the output needed to look like), Claude drafted the system prompt and this README, and I personally ran and checked every real result shown above — including finding the guardrail-inconsistency bug myself by running the agent twice with the same instructions and comparing outputs. Nothing in the eval results table was invented; each row traces to an actual run.

# Open voice-AI benchmark (Italian, Spanish, Portuguese)

A reproducible test of AI phone-agent platforms on **real phone calls** in non-English languages.

This repository is a standalone copy of the method, prompts, scenarios and measurement scripts published in [`Callin-io/callin-agents`](https://github.com/Callin-io/callin-agents/tree/main/benchmark). Run by [Callin.io](https://callin.io). If Callin.io loses a metric, the result is published anyway.

Homepage: [callin.io](https://callin.io)

## Platforms

Callin.io, Retell AI, Vapi, Bland AI, ElevenLabs Agents (optional: Synthflow, Air).

Same conditions for all: the shared prompt in `agent_prompt.md` translated per language, the same LLM where the platform allows it (Gemini 2.5 Flash or GPT-4o mini), the best native voice available per language, a local phone number, vendor-recommended defaults otherwise.

## Languages and accents

- it-IT (Milan, Naples)
- es-ES (Madrid, Andalusia)
- es-MX
- pt-BR (São Paulo)

6 accents × 5 scenarios × 5 platforms = 150 calls, repeated 3 times = 450 calls.

## Scenarios (`scenarios/*.json`)

1. `booking` — restaurant table: date, time, party size, name
2. `reschedule` — move a clinic appointment, confirm identity data
3. `faq` — answer from a short document (opening hours, prices)
4. `interruption` — caller changes their mind mid-sentence
5. `handoff` — caller asks for a human: transfer

Each scenario has fixed caller lines, recorded once by native speakers (real voices, not TTS), so every platform hears exactly the same audio. Caller audio files are **not** included in this repository.

## Metrics

| Metric | How | Unit |
|---|---|---|
| Response latency | end of caller turn → start of agent speech, from the stereo recording (`scripts/measure_latency.py`) | ms, median and p90 |
| Understanding | word error rate of the agent's transcript on fixed test phrases with names, numbers, addresses (`scripts/score_wer.py`) | % |
| Task success | scenario closed with the right data, checked by hand against `expected` | % of calls |
| Naturalness | blind listening by 5 native speakers per language | 1–5 |
| Interruption handling | agent stops and resumes correctly | % of calls |
| Cost | public all-inclusive price (telephony + LLM + voice) | $ / min |

## Reproduce

1. Install dependencies:

   ```bash
   python -m pip install -r requirements.txt
   ```

2. Configure each platform with `agent_prompt.md` and the scenario's `knowledge` text.

3. The caller bot dials each agent and plays the caller lines in order, waiting for the agent to finish speaking (silence > 700 ms) before the next line. Record the call in stereo: left = caller, right = agent.

4. Save recordings as `recordings/<platform>/<lang>/<scenario>_<run>.wav`.

5. Measure latency (CSV columns: `platform,lang,scenario,run,turn,latency_ms`):

   ```bash
   python scripts/measure_latency.py recordings/ > results/latency.csv
   ```

6. Export each platform's transcript as `transcripts/<platform>/<lang>/<scenario>_<run>.txt` (the agent-side transcript of the caller's words). Score WER against the scenario's `test_phrases` (CSV columns: `platform,lang,scenario,run,phrase,wer`):

   ```bash
   python scripts/score_wer.py transcripts/ > results/wer.csv
   ```

7. Blind rating: shuffle clips, hide platform names, collect 1–5 scores.

8. Before publishing, send each vendor its own results; 7 days to flag configuration errors.

Re-run every quarter.

## License

MIT. © Replacing Humans LLC (Callin.io).

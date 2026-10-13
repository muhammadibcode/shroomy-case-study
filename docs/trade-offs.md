# Trade-offs

The decisions below are the ones I would defend in an interview, each with what was traded and what the evidence was. Everything here is backed by the production codebase, its commit history or the product's own analytics.

## Voice and audio

**1. Stream chat replies, fetch guided sessions whole.**
Chat replies are streamed from ElevenLabs through an on-device loopback proxy that supports range requests, so speech starts on the first bytes. Five-minute guided sessions are fetched as one file, cached by script hash and de-duplicated across screens.
*Traded:* a short wait before a session starts, for reliable scrubbing, lock-screen controls and no mid-session stalls.
*Evidence:* two distinct client paths and two distinct edge functions; cache and de-duplication logic; an iOS background-execution hook so a download survives the phone locking.

**2. Move from OpenAI text-to-speech to ElevenLabs (June 2025).**
The first audio path returned a single base64 blob from OpenAI's TTS through a Firebase function. I moved both voice surfaces to ElevenLabs behind new edge functions.
*Traded:* a second vendor relationship and a migration, for expressiveness, pause control and faster generation.
*Evidence:* the legacy Firebase function still exists in the repository; the ElevenLabs edge functions date from 24 June 2025; the loading animation is tuned to the Turbo model's generation time.

**3. Keep one brain for text and voice.**
Voice chat transcribes, reasons with the same Claude call, memory, exercise tool and safety router as text, then speaks. A real-time speech-to-speech relay was prototyped in spring 2025 and is not in the shipped app.
*Traded:* some turn latency, for a single source of truth about the user and a single place to enforce safety.
*Evidence:* the voice screen routes every transcript through the safety router and the shared chat pipeline; the relay function exists in the backend history but has no callers in the app.

**4. Two breathing patterns only.**
Generated breathwork may use physiological sigh or box breathing. The 4-7-8 pattern is banned in the prompt and in the product specification.
*Traded:* variety, for safety in an anxious population.
*Evidence:* the constraint appears in the audio prompt and the product context document.

## Personalisation versus safety

**5. Validation mode for reframing.**
Users in real grief or mistreatment reported that cognitive reframing told them they were wrong. The reframing exercise gained a runtime mode decision made by a first model pass on the user's own words; the screen hides all mode-dependent copy behind a neutral loading state until the mode is known; validation mode swaps distortion labels for emotion labels, makes input optional and replaces corrective feedback with reflective mirroring.
*Traded:* a slower first card and a second prompt branch to maintain, for not harming the people the default flow got wrong.
*Evidence:* the mode is persisted on the task; roughly one in six reframing sessions route to validation in production.

**6. A verbatim rule instead of paraphrase.**
Every reframing card must quote a thought the user actually wrote, and the model skips a card rather than fabricating one when nothing meets a 90% certainty bar. The second-phase call may return fewer cards than requested.
*Traded:* occasional "only one card" sessions, for never putting words in a user's mouth.

**7. A bounded adjustment surface for day-one feedback.**
The feedback conversation that regenerates the first week enumerates what the system can change (exercise mix, audio style, tone, depth, focus) and what it cannot (any UI change, any format that does not exist, the mechanics of a flow), with worked examples drawn from real production failures. The model must translate an impossible request into the nearest achievable one rather than agree to it.
*Traded:* a longer prompt, for no trust-breaking promises.

**8. A safety router that stays off until it has earned it.**
Built behind a remote flag, off by default, with a deterministic 80-plus-fixture gate in CI and a manual live evaluation before any release that touches it. For physical-symptom and health-anxiety turns it runs a short user-led self-assessment and escalates only on concrete observable indicators, because in this population both under- and over-escalation cause harm. Acute text never re-enters a model request. Copy is tested for certainty claims and emergency vocabulary.
*Traded:* shipping velocity on a safety feature, for a measurable floor before anyone sees it.

## Cost and reliability

**9. Constrain the model to six interaction formats.**
Every exercise must be one of six fixed formats with per-format required fields, enforced in the tool schema and re-validated in the app.
*Traded:* the model's freedom to invent, for a product that is renderable, testable, instrumentable and safe to change.

**10. One combined call for the whole week.**
Week planning and the 14 exercises were originally two calls; they became one structured call, with the two-stage path kept as a fallback.
*Traded:* a longer single response, for one round trip and one failure surface per week.

**11. Model tiering, and explicit downgrades.**
Sonnet-class models for weekly generation, chat and analysis; a Haiku-class model for the lightweight safety classifier; prompt caching on the long-running chat surface. A dated commit in February 2026 downgrades model tiers on specific calls to cut cost, alongside another that condenses the largest prompt.
*Traded:* some quality headroom, for unit economics that work at consumer subscription prices.

**12. Deterministic notifications over LLM-written ones.**
Notification copy used to be generated per user from their profile and memory. I replaced it with a rotation of goal-keyed templates chosen by a pure function of day, slot and the user's goals, scheduled locally fourteen days ahead.
*Traded:* personalised copy, for reliability, offline correctness and zero cost on a surface where a bad output cannot be recalled.

**13. Localise the display, store in English.**
Spanish shipped across the app in February 2026, but every stored onboarding answer is reverse-mapped to its canonical English value before saving.
*Traded:* a small translation layer, for prompts that never degrade with locale.

## Growth and monetisation

**14. Ask for notifications after the first win, not at launch.**
The permission prompt moved to the moment after a completed exercise, and the OS prompt is skipped entirely when reminders are off, so a declined permission is never burned by accident.
*Evidence:* 81% opt-in at the in-app prompt (internal analytics, 2026).

**15. Run the pricing test, read it honestly, do not ship the loser.**
A two-month feature-flagged test of a weekly price point against the default annual and monthly paywall reduced trial starts slightly. The variant was not shipped. The sample was small, so the decision was the point, not the delta.

**16. Treat churn as a habit problem.**
A bilingual in-app churn survey found "life got in the way / I forgot" to be the dominant stated reason by a wide margin. Investment went into notification reliability, timing and the permission flow rather than more content.

**17. Say no to popular requests.**
Lifetime pricing, data export, cross-device sync, dark mode and a referral link were each requested repeatedly and deliberately not built, in favour of the programme, the reframing redesign and reliability. A small product has to pick.

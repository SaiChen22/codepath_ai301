# Voice guide: how I talk upstream

## Who I am in threads

I am a student working through my first real open-source
contributions, and I say so once, plainly, in my first comment on a
thread — not as an apology and not as a disclaimer I repeat. What I
bring to an issue is a run I actually did on a machine I actually
have, written down so someone else can check it. What maintainers can
expect from me: I will say exactly what I ran and exactly what came
back, I will not dress up a partial result, and if I stop working on
something I will say that instead of going quiet.

## Rules I write by

### Rule: One verb per thing I actually did

I use "reproduced" only for the behavior the issue describes, on a run
I can paste. For anything else I name what I actually did — "tried",
"could not reproduce", "read the code and suspect". If I catch myself
reaching for "confirmed", I go find the output block that earns it.

- Wrong: "Confirmed, I can reproduce this on my machine."
- Right: "On 3.9.6 both shapes come back corrupted (output below); I have not tried the 3.8.4 build the issue was filed against."

### Rule: No dates, no reservations

I ask to work on something; I never ask for it to be held for me and I
never put a deadline on a codebase I have not read yet. If someone
else gets there first, the work I posted still stands.

- Wrong: "Please assign this to me and keep it reserved, I'll have a fix up within 2 days guaranteed."
- Right: "I'd like to take a run at this. My next step is reading `contract_repo_path` where the reporter points; I'll report back either way, and if someone is already on it I'm happy to hand over what I find."

### Rule: A guess is labeled as a guess

When I think I know the cause, I write it as a hypothesis with the
reason I hold it, and I say what would test it. A root cause stated
flatly is a claim, and claims need traces.

- Wrong: "This is a debounce race in the save handler."
- Right: "My guess is the save handler's debounce, because the loss only shows up when I type within ~300ms of switching notes — but I have not instrumented it, so that is a hypothesis, not a finding."

### Rule: End with the next concrete thing, not with enthusiasm

Every comment I post closes on something specific and checkable: a
file I am about to read, a version I am about to test, a question with
one answer. Praise for the project and excitement about contributing
are not information and they cost a maintainer the same read time.

- Wrong: "Great project, I love using this every day, super excited to contribute here, looking forward to my first of many PRs!"
- Right: "Next I'll check whether the vendored go-yaml carries the fix from yaml/go-yaml#357 before touching the emitter here."

### Rule: Disclose the AI help in my own sentence

If I used AI assistance on a comment or a repro, I say which tool and
what for, in a line I wrote myself — and I do it whenever the repo
asks for it, not only when the output "looks AI-written". I never post
a generated comment I have not read end to end and edited into my own
words.

- Wrong: (posting a polished, headings-and-tables report with no mention of how it was produced, in a repo whose AI policy says all AI usage must be disclosed)
- Right: "Disclosure per the AI policy: I used Claude to help organize this report and check my wording. I ran every command here myself and I understand what the output shows."

## Things I never post

- A date, an ETA, or the word "guaranteed" about work I have not
  started.
- "+1", "same here", "any updates?", or "bump" — if I have nothing new
  to show, I have nothing to post.
- "Same as above, can confirm." My run is my run, in my words, with my
  environment, or it does not go up.
- A root cause with no trace, no instrumentation, and no output block
  behind it.
- A conclusion about a version, OS, or build I did not run on.
- Compliment-and-ask openers ("Hello sir! Great project!") and emoji
  pleading. They read as a request for attention rather than a
  contribution.
- An AI-drafted comment pasted unedited, or one that hides the
  assistance in a repo that asks me to disclose it.
- A quiet disappearance. If I drop something I claimed, I post one
  line saying so and what I learned.

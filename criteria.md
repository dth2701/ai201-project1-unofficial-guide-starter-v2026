# Acceptance criteria — The Unofficial Guide

Five criteria that say what "working" means for this system, written in unit 1
**before** any results existed.

An acceptance criterion names a target: a number, a count, a rate, or something
a person could plainly observe. *"Retrieval works"* is an opinion. *"For at
least 4 of my 5 test questions, the top results include a chunk containing the
answer"* is a criterion.

Under each one, write a sentence or two on **why that target** and not a
stricter or looser one. A reason that says something about your corpus or your
pipeline earns credit; *"80% seemed reasonable"* does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## 1. Retrieved chunks contain the answer

For at least 4 of my 5 test questions, the retrieved chunks include one that
contains the answer.

**Why this target:** Four of my questions have an obvious matching document.
"Kestrel Commons" is in `dining_kestrel_commons.txt`, "Morrow" is in
`housing_morrow_house.txt`, and so on. The words in the question are the words
in the file.

The fifth one is not like that. I ask about keeping a textbook from the
library's reserve shelf, and the answer ("two-hour reserve") is in
`money_textbooks.txt`, which is a post about saving money on books. The two
files that actually say "library" — `admin_library_holds.txt` and
`study_library_hours.txt` — never mention two hours. I think search will return
those two and miss the right one. So I allow one miss, and I am saying now which
one I expect it to be.

---

## 2. Every answer names a source

Every answer the system produces names at least one source document.

**Why this target:** All five, because nothing about my corpus makes this hard.
`build_prompt` in `generate.py` already puts `[from <filename>]` above every
chunk it sends, so the filename is right there in the prompt every time. If an
answer comes back with no source in it, that is the model ignoring the
instruction, not a gap in my documents. There is no reason to accept four.

---

## 3. The relevance gate stops out-of-corpus questions

When I ask a question my documents clearly don't cover, the relevance gate
stops it and the system returns "I don't have enough information about that" —
in at least 4 of 5 tries.

<!-- The five questions are the ones in `OUT_OF_SCOPE` at the bottom of
     `questions.py`, and `run_eval.py` puts them through the gate and writes
     what happened into your run log. Swap them for your own if you'd rather —
     just keep five of them, or the "4 of 5" above has nothing to be 4 of. -->

**Why this target:** I have not measured my distances yet. `THRESHOLD` in
`config.py` is still the starter's 0.6, and I set my own number in Milestone 4.
This target is a guess based on the questions, not on numbers I have seen.

Four of my five out-of-scope questions use words my corpus never uses: Mongolia,
diesel engines, the World Cup, ibuprofen. Those should be easy to refuse. The
fifth is "How do I write a for loop in Rust?". I have three CS 340 files, and
they use words like "project", "week" and "course" that a programming question
also uses. If one question gets past the gate, it is that one.

---

## 4. Every chunk says what it is about

I check these ten chunks, the same ten every time:

    housing_old_brewhouse.txt          housing_innisfree_hall.txt
    housing_morrow_house.txt           course_hist_118_exams.txt
    course_cs_340_exams.txt            dining_kestrel_commons.txt
    dining_the_ridgeway_cafe_followup.txt   admin_pass_fail_option.txt
    money_textbooks.txt                admin_add_drop_deadline.txt

In all 10, the chunk text names the thing it is about — the dorm, the dining
hall, the course, or the rule — inside the chunk itself. I am not allowed to
look at the filename to work out what the chunk is about.

**Why this target:** My first version of this criterion used a size range, 150
to 900 characters. I threw it out. Every file in `campus_life` is between 183
and 554 characters and `CHUNK_SIZE` is 800, so the fixed-size chunker never
splits anything and never produces a chunk outside that range. The criterion
could not fail no matter what I did. A criterion that cannot fail tests nothing.

This version can fail. `housing_innisfree_hall.txt` is one title line and four
paragraphs. If my Milestone 3 chunker splits on paragraph breaks, one of those
chunks is "Laundry costs $1.75 wash, $1.75 dry, app-based" with no dorm name
anywhere in it. Search might still return it for a laundry question, and the
answer would have no way to say which dorm. This is not a one-off: 16 of my 88
documents have four or more paragraphs, and 7 housing files end with a laundry
paragraph that never repeats the dorm name.

I want 10 of 10 and not 8 of 10 because one anonymous chunk is enough to produce
a confident answer about the wrong dorm. There is no acceptable number of those.

I picked those ten files on purpose, not at random: the longest file, the
shortest file, a followup, and the files behind my test questions. The same ten
every run means a change in the score is a change in my chunker, not a change in
which chunks I happened to draw.

**How I check it:** `python app.py chunks` prints chunks with their source file.
I read the ten and ask the question your handout asks — could someone answer a
question using only this? For the part I can automate, I put the ten topic words
("Old Brewhouse", "Innisfree", "Morrow", and so on) in a list, loop over the
chunks with a `for` loop, and use `if word.lower() in chunk.text.lower():` to
print which ones are missing their own name. `in` on a string is a substring
check, so that is the whole test.


---

## 5. The answer says the right thing

For at least 4 of my 5 test questions, the answer text contains the `expects`
string I wrote for that question in `questions.py`, ignoring capital letters.
Those strings are "20 to 25", "Ridgeway", "Morrow", "week eight" and "two-hour".

**Why this target:** My other four criteria can all pass while the system hands
me a wrong answer. Criterion 1 checks the right chunk came back. Criterion 2
checks a filename is printed. Neither one reads the answer. A fluent, well-cited,
wrong answer scores four out of four. That is the thing that would embarrass me,
so it is the thing I am testing.

I picked 4 of 5 and not 5 of 5 because this criterion cannot beat criterion 1.
If the textbook reserve question never retrieves `money_textbooks.txt`, the
model has no way to say "two-hour", because the words are not in front of it.
That is the same question failing twice, for the same reason, and I said in
criterion 1 that I expect it to fail. Setting 5 of 5 here would mean claiming
the model can answer from material it was never given.

I am not going higher than 4 for the same reason, and not lower because the
other four questions each have a document that states the answer in one
sentence. If the model is handed "Wait times: 20 to 25 minutes between 12:15 and
1:00" and does not say 20 to 25, something is wrong that is worth finding.

**Known risk with this one:** I am checking for an exact string. If the model
writes "20-25 minutes" or "week 8", my check says fail when the answer was
right. If that happens I will fix the check, not the target — that is a broken
measurement, not a missed number, and the unit 2 rules say it gets revised with
the original left in place.

**How I check it:** `run_eval.py` writes each answer into `results/`. For each
question I take the answer text and the `expects` string and use
`if expects.lower() in answer.lower():` — `in` on a string checks whether one
string appears inside another, and `.lower()` makes both lowercase so "Ridgeway"
and "ridgeway" both count. A `for` loop over the five questions and a counter
gives me the score. That is the whole scorer, about six lines.


---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 2 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath it, like this:

         ## 1. Retrieved chunks contain the answer

         For at least 4 of my 5 test questions, the retrieved chunks include
         one that contains the answer.

         **Why this target:** ...

         > **Revised in unit 2:** For at least 4 of 5 questions, the top three
         > results contain the answer.
         >
         > **Why revised:** I couldn't judge "the chunks include one that
         > contains the answer" the same way twice — I scored two questions
         > differently on Monday than on Wednesday. The new version is
         > something I can actually check.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "I said 4 of 5 but got 2 of 5, so 2 of 5 is more realistic."

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.

     The whole reason the originals stay visible is so someone can see what you
     said before you knew the answer.
     ───────────────────────────────────────────────────────────────────────── -->

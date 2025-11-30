---
title: deciphering tunic
date: 2025-07-06
tags: [games]
---

I bought [Tunic](https://tunicgame.com/) a while ago when it was on special and
I finally got around to playing it last week.

{{< callout type="warning" text="While this post doesn't explicitly contain any spoilers, it reveals several pages of the instruction manual, which could be considered light spoilers if you want to experience the game fully for yourself." >}}

A core mechanic of the game is that instead of explicitly teaching you how to
play, pages of the game's "instruction manual" is scattered throughout the
world for you to discover. This lets the game reveal the more advanced
mechanics of the game to the player over time.

The catch? The instruction manual only has a smattering of English words! A
large part of the manual is written in a unique script created by the game
designers.

{{< img src="images/tunic/page-10.jpg" caption="One of the first pages you find in the game." >}}

That might sound counterintuitive, but the idea from the developers was to
evoke that feeling of picking up a game as a kid and not being able to fully
understand what's going on, while still giving just enough information for you
to figure out the game for yourself. The player isn't expected to actually read
the text.

## But is it text?

At first, I didn't pay too much attention to the script. There's enough
variation in the symbols to give the impression that it could plausibly be
text, but it could also just be the work of a really talented designer.
Eventually, the mystery got the better of me and I decided to give deciphering
it a shot.

With absolutely zero background information, the only way to proceed was to
make some educated guesses and cross-check them with other text to see if they
hold. I started with some basic assumptions:

* The text is English.
* Each group of symbols connected by the centre line denotes a word.
* Given the amount of glyphs and how short each word appears, the system is some kind of
  [syllabary](https://en.wikipedia.org/wiki/Syllabary) based on [English
  phonemes](https://en.wikipedia.org/wiki/English_phonology#Phonemes), not
  letters.

## Making educated guesses

With that in mind, I skimmed through the manual looking for text next to
pictures that could give a hint to its meaning. This text caught my eye:

{{< img src="images/tunic/lost.png" >}}

Based on the picture, and the fact that page 28 was a map of one of the main
areas of the game, I thought there was a chance that this text reads, `LOST?
SEE p.28`. I quickly jotted down in my notebook my first guesses:

{{< img src="images/tunic/lost-see.png" width="80%" >}}

At this point, I had no idea *how* those glyphs mapped to those sounds, but at
least I had a starting point. My next target was the heading on this page:

{{< img src="images/tunic/page-12.jpg" >}}

A very natural guess for this word is, of course, `CONTROLS`. There was no way
to validate this, but with a close look I saw that this could plausibly line up
with my previous guess:

{{< img src="images/tunic/controls.png" caption="I got slightly lucky here, because I didn't notice at the time that the final glyph here is actually a flipped version of the S from earlier." >}}

Next, I looked at the text next to the shield, which has the potential `L`
glyph in the middle.

{{< img src="images/tunic/shield2.png" >}}

I initially thought this might be the word `BLOCK`, but it didn't feel right.
Instead, I made a wild guess and pencilled in `SHIELD` for this word instead.
That gave me a potential glyph for `D` (which eventually turned out to be
right!).

{{< img src="images/tunic/shield.png" width="60%" >}}

Then, I tackled the page explaining the space button, in particular the two
actions on the right:

{{< img src="images/tunic/page-13.jpg" >}}

From playing, I knew that pressing space does a dive roll, while pressing and
holding spacing lets me run.  Given that, my guesses here were `ROLL` and
`RUN` respectively:

{{< img src="images/tunic/roll-run.png" width="80%" >}}

This would line up perfectly with the guesses for `N` and `ROL` in `CONTROLS`
from earlier, which gave me confidence that I was on the right track. Though my
understanding of the system was far from complete, I felt it was time to
try deciphering a major piece of text.

Where else but the first page to start? I went back and scribbled down a gloss
for the first couple of words:

{{< img src="images/tunic/a-secret-legend.png" >}}

Treating this like a fill-in-the-blanks puzzle, I thought the second word could
be `SECRET` and the third word could be `LEGEND`. This gave me the following
guesses to add to my list of glyphs:

{{< img src="images/tunic/more-glyphs.png" width="80%" >}}

## Consonants and vowels

All the pieces fit together well so far, so I had some confidence that my
starting assumptions were correct. Treating the text as English worked,
grouping the glyphs into words based on the centre line worked, and treating
the glyphs as syllables also yielded plausible results. Still, I thought maybe
there was more to the system: I felt that there was enough evidence to suggest
that the syllables could be further deconstructed into consonants and vowels.

Grouping the syllables by consonant, you can see that they seem to share parts
of their structure:

{{< img src="images/tunic/comparison.png" width="60%" caption="Some of these transcriptions aren't correct and still needed refinement, but they were what I had to work with at this stage." >}}

Sharing similar structures implied that further deconstruction is possible. For
example, I had potential glyphs for `L`, `LE` and `JE`. If we assumed that the
common parts of each glyph represented the consonant or vowel part of the
syllable, we could make a deduction like this:

{{< img src="images/tunic/le-je.png" >}}

This was the final clue that unlocked the rest of the script. With this, each
glyph could be deconstructed into a consonant part (made up of the inside
strokes of each glyph) and a vowel part (made up of the top, left and bottom
strokes of the hexagon). With this principle in hand, the rest of the script
becomes relatively straightforward to decipher. This is left as an exercise for
the reader :)

## Phonemes vs. letters

A core concept of this type of script is that it doesn't use regular spelling
for its words but instead it uses each word's phonetic pronunciation as the
basis of its spelling.

For example, take the words `BAIT` and `BITE`. Although `BAIT` is *spelt* with
"-ai-", its phonetic value is `EI` (as in `BEIGE`). Conversely, while `BITE` is
spelt "-ite", its phonetic value is `AI` (as in `CHAI`). This principle leads
to the following spellings:

{{< img src="images/tunic/bait-bite.png" >}}

This principle also means that homophones that are spelt differently in regular
spelling become "homoglyphs" in the game script, such as in this example with
`MEAT`, `MEET` and `METE`:

{{< img src="images/tunic/meat-meet-mete.png" >}}

## One final piece of the puzzle

What we have so far deciphers most of the script, except for one feature:
sometimes, the script includes a small circle under a glyph. Fortunately, the
developers were kind enough to provide a hint on page 21 of the instruction
manual. Notice the speech bubble with the circle under the first glyph, and
the pen markings that seem to imply some kind of swapping or reordering:

{{< img src="images/tunic/page-21.jpg" >}}

Applying what we know so far to translate this speech bubble, we get the phrase
`MY STUCK`, which doesn't really make sense:

{{< img src="images/tunic/my-stuck.png" width="60%" >}}

Using the hint, we could try swapping the order of the first syllable from `M + AI` to `AI + M`:

{{< img src="images/tunic/im-stuck.png" width="60%" >}}

This gives a much more plausible (and correct) reading of `I'M STUCK`. In other
words, a circle reorders a syllable from consonant + vowel to vowel +
consonant. This gives words like `ITEM`, which is correctly written

{{< img src="images/tunic/item.png" >}}

instead of the incorrect

{{< img src="images/tunic/item-wrong.png" caption="This is the wrong spelling." >}}

## Do you speak North American?

Given the variety of dialects of English in the world today, a phonetic script
for English has the interesting consequence of forcing a particular accent on
the reader.

For example, native speakers of English tend to pronounce the word `THE`
differently depending on whether the word that follows it begins with a vowel:
they will say **/ðə/ banana** ("the") but **/ði:/ apple** (rhyming with
"thee"). The designers chose to enforce this distinction in the script, using
both spellings for the word `THE` depending on context:

{{< img src="images/tunic/the-thee-runes.png" width="80%" >}}

You can see this distinction in sentences like the following:

{{< img src="images/tunic/the-thee.png" caption="Inscription reads: /ði:/ (\"thee\") heir hungers for reminders of /ðə/ (\"the\") corporeal world." >}}

A variation like this is common to many English dialects, but as a native
speaker of New Zealand English, one variation that stood out to me is that the
script only has one glyph for both the `LOT` and `THOUGHT` vowels. For example,
I (and probably Australian and UK speakers in general) pronounce `COT` as /kɔt/
and `CAUGHT` as /ko:t/, but there is no glyph available for the /o:/ sound.  In
other words, the [cot-caught
merger](https://en.wikipedia.org/wiki/Cot%E2%80%93caught_merger) is required by
the script and gives the following homoglyph, even though in my native dialect
these words are *not* homophones:

{{< img src="images/tunic/cot-caught.png" >}}

A further quirk of the script is with the spelling of the schwa (/ə/). Though
the schwa is a common sound in English [due to vowel
reduction](https://www.youtube.com/watch?v=EaXYas58_kc), there is no dedicated
glyph for it. This meant that the developers had to repurpose other vowels for
this case, sometimes with the `STRUT` vowel /ʌ/, sometimes with the `KIT` vowel
/ɪ/, and sometimes even with the `DRESS` vowel /ɛ/! To me, it seems to be an
interesting design decision to enforce minor distinctions like the
pronunciation of `THE` while leaving out this major feature of English.
Consequently, we see words like `CONTROL` /k**ə**ntroʊl/ spelt /k**ʌ**troʊl/,
`BUTTON` /bʌt**ə**n/ spelt /bʌt**ɪ**n/, and `ITEM` /aɪt**ə**m/ spelt
/aɪt**ɛ**m/.

## Final thoughts

In short - I love this script! It's a unique idea, it gives a special flavour to
the feel of the entire game and it makes a great side quest when you want a
break from adventuring.

I wonder if the design might have been influenced by the [Shavian
alphabet](https://en.wikipedia.org/wiki/Shavian_alphabet) where voiced-unvoiced
consonant pairs are also flipped versions of each other – though in the game
script they are vertically flipped, whereas in the Shavian alphabet they're
rotated 180º. I liked this design decision, because it meant that you could
make a good educated guess for the sound value of a glyph while deciphering
it. For example, once I had /ʒ/ from `TREASURE` I could pencil the flipped
glyph as a guess for /ʃ/.

I also loved how the designers chose to style the writing by separately
connecting the top and bottom parts, obfuscating how the letters work and
making deciphering the script that much more fun. Consider the word button from
the heading of page 13:

{{< img src="images/tunic/button.png" width="60%" >}}

The way the border is styled implies that the top and bottom parts are somehow
separate, when the glyphs are actually split like this:

{{< img src="images/tunic/button-split.png" >}}

A prominent example of this is in the logo of the game, where they fully
separate the top and bottom parts of the phrase `SECRET LEGEND` by inserting
the English word TUNIC in-between the top and bottom parts!

{{< img src="images/tunic/title.png" width="80%" >}}

This was a fun side quest and I think a great way to build up the game lore.
With that done, time to get back to adventuring :D

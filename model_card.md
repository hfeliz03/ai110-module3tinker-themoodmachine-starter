# Model Card: Mood Machine

This project includes two mood classifiers for short informal text:

1. A rule-based model in `mood_analyzer.py`
2. A small ML model in `ml_experiments.py`

## 1. Model Overview

**Model type:**  
I compared both models, but most of the design work was done in the rule-based version.

**Intended purpose:**  
The goal is to label short posts or messages as `positive`, `negative`, `neutral`, or `mixed`.

**How it works (brief):**  
The rule-based model preprocesses text into tokens, scores positive and negative cues, and converts the final score plus cue balance into a label. The ML version uses bag-of-words features with `CountVectorizer` and trains a logistic regression classifier on the labeled dataset in `dataset.py`.

## 2. Data

**Dataset description:**  
The final dataset contains 17 posts in `SAMPLE_POSTS` and 17 matching labels in `TRUE_LABELS`. I expanded the starter set with realistic short posts that included slang, emojis, sarcasm, and mixed emotions.

Added examples included:

- `Lowkey proud of myself today :)`
- `lol that meeting was awkward but whatever`
- `No cap, I'm stressed and excited at the same time`
- `I absolutely love waiting in traffic for an hour 🙃`
- `ngl I'm kinda sad but the playlist is good`
- `That surprise quiz had me sick 😭`
- `Highkey chilling tonight, finally`
- `I love when my phone dies at 2 percent 🙃`
- `This playlist is fire`
- `I'm exhausted but proud I finished it`
- `Sick of this app crashing again`

**Labeling process:**  
I assigned labels manually based on the overall mood I thought a human reader would infer, not just the presence of positive or negative words. Some posts were easy, like `Today was a terrible day -> negative`. Others were ambiguous:

- `lol that meeting was awkward but whatever` was labeled `neutral` because the speaker sounds mildly annoyed but also dismissive rather than strongly upset.
- `ngl I'm kinda sad but the playlist is good` was labeled `mixed` because it contains genuine negative and positive feelings.
- `I absolutely love waiting in traffic for an hour 🙃` was labeled `negative` because the intended meaning is sarcastic frustration.

**Important characteristics of the dataset:**

- It contains slang such as `lowkey`, `highkey`, `ngl`, `no cap`, and `fire`.
- It includes emojis and emoticons such as `:)`, `😭`, and `🙃`.
- It includes sarcasm, especially around traffic and phone battery examples.
- It includes mixed-emotion posts such as being tired but hopeful or exhausted but proud.
- Most examples are very short, which makes them sensitive to one or two tokens.

**Possible issues with the dataset:**  
The dataset is tiny, subjective, and not balanced across all language styles. Some labels depend on cultural familiarity with slang or sarcasm. There are also very few truly neutral examples, and there is no held-out test set.

## 3. How the Rule Based Model Works

**Your scoring rules:**  
The rule-based model uses three main steps:

1. `preprocess()` lowercases text, removes most punctuation, normalizes a few contractions such as `I'm -> im`, and extracts tokens including simple emoji tokens like `:)`, `😭`, and `🙃`.
2. `score_text()` loops over tokens and starts from score `0`.
3. Positive cues add `+1` and negative cues add `-1`, with a simple negation rule that flips the meaning when the previous token is one of `not`, `never`, `no`, `cant`, or `wont`.

I kept the base positive and negative lexicons from `dataset.py`, then added lightweight extra signals directly in `mood_analyzer.py`.

Positive signals:

- `hopeful`
- `proud`
- `finally`
- `chilling`
- `chill`
- `good`
- `:)`

Negative signals:

- `awkward`
- `traffic`
- `sick`
- `sad`
- `stressed`
- `tired`
- `exhausted`
- `:(`
- `😭`
- `🙃`

For label prediction, the model does not use the score alone. If it sees both positive and negative cues in the same text, it returns `mixed`. Otherwise it returns `positive` for score `> 0`, `negative` for score `< 0`, and `neutral` for score `== 0`.

**Strengths of this approach:**  
The model behaves predictably on direct statements with obvious mood words. It handles simple negation correctly in examples like `I am not happy about this`. It also works reasonably well on mixed-emotion examples when both sides are explicit, such as `Feeling tired but kind of hopeful` and `No cap, I'm stressed and excited at the same time`.

**Weaknesses of this approach:**  
The rules only know what I manually encoded. If a word is missing from the lexicon, it is ignored. The model also treats conflicting literal cues as `mixed`, even when a human would read the whole sentence as sarcastically negative.

## 4. How the ML Model Works

**Features used:**  
The ML model uses bag-of-words features through `CountVectorizer`.

**Training data:**  
It trains directly on `SAMPLE_POSTS` and `TRUE_LABELS` from `dataset.py`.

**Training behavior:**  
The ML model changed a lot when I added more examples. After I added extra sarcasm, slang, and mixed-emotion posts, the training-set accuracy reached `1.00`. That does not mean the model truly generalized well; it mostly shows that with a tiny dataset, it can memorize the exact examples and tokens I labeled.

**Strengths and weaknesses:**  
The ML model learned some patterns the rule-based model missed without me adding manual rules. For example, after I added `This playlist is fire` as a positive example, the ML model predicted it correctly while the rule-based model still returned `neutral`. It also matched my labels on the sarcastic traffic and phone-battery examples because those token combinations appeared in the training data. The downside is that it is extremely sensitive to my labels and to the small size of the dataset, so its perfect score is likely overfitting.

## 5. Evaluation

**How you evaluated the model:**  
I ran `python main.py` for the rule-based system and `python ml_experiments.py` for the ML system. Both evaluated on the labeled posts from `dataset.py`.

Observed results on the final 17-post dataset:

- Rule-based accuracy: `0.76`
- ML training accuracy on the same dataset: `1.00`

**Examples of correct predictions:**

- `I am not happy about this` -> `negative`  
  The negation rule flipped `happy` into a negative signal.
- `No cap, I'm stressed and excited at the same time` -> `mixed`  
  The rule-based system detected both negative (`stressed`) and positive (`excited`) cues.
- `Sick of this app crashing again` -> `negative`  
  The rule-based system treated `sick` as negative in this context, which matched the label.

**Examples of incorrect predictions:**

- `I absolutely love waiting in traffic for an hour 🙃`  
  Rule-based prediction: `mixed`  
  True label: `negative`  
  The model saw `love` as positive and `traffic` plus `🙃` as negative, then reduced the whole sentence to competing cues instead of understanding the sarcasm.
- `I love when my phone dies at 2 percent 🙃`  
  Rule-based prediction: `mixed`  
  True label: `negative`  
  This failed for the same reason: literal positive word plus clearly negative event plus sarcasm emoji.
- `This playlist is fire`  
  Rule-based prediction: `neutral`  
  True label: `positive`  
  The word `fire` was not in the rule lists, so every important cue was ignored.
- `lol that meeting was awkward but whatever`  
  Rule-based prediction: `negative`  
  True label: `neutral`  
  The model over-weighted `awkward` and did not understand that `whatever` softened the tone.

The ML model behaved differently on these same examples. It matched my labels on all four, probably because the training data contained those exact phrases or closely related ones.

## 6. Limitations

The most important limitation is that the rule-based system cannot reliably interpret meaning beyond token-level cues.

Specific examples:

- `I absolutely love waiting in traffic for an hour 🙃` was misclassified as `mixed` instead of `negative`. The model counted `love` literally and could not infer sarcasm from the full sentence.
- `I love when my phone dies at 2 percent 🙃` had the same issue. The negative situation did not override the positive keyword strongly enough.
- `This playlist is fire` was misclassified as `neutral` because `fire` was missing from the lexicon. A human reader familiar with the slang would read it as positive immediately.
- `lol that meeting was awkward but whatever` was labeled `neutral` by me but predicted `negative`. The model understood `awkward` but ignored the dismissive tone of `whatever`.

The ML model has a different limitation: it can appear much better than it really is because it is evaluated on the same tiny dataset it trained on. A perfect score in this setup does not show robust generalization.

## 7. Ethical Considerations

Mood detection can be misleading when used on real people’s messages. A wrong label could flatten mixed feelings, miss sarcasm, or misread distress. For example, a system that treats `I'm fine 🙂` as neutral or positive without context could overlook discomfort or masking.

This dataset is also narrow in scope. It is optimized for short English-language social-style posts with internet slang that I happened to include, like `ngl`, `lowkey`, `no cap`, and `fire`. It may misinterpret people who use different dialects, different age-coded slang, non-English expressions, or emoji conventions that differ from mine. That means the model has bias toward the language style represented in this small hand-built dataset.

There are also privacy concerns if a mood classifier were used on private texts, DMs, or journal-like content without clear consent.

## 8. Ideas for Improvement

- Add many more labeled examples, especially neutral examples and slang from more communities.
- Create a real test split instead of reporting training accuracy for the ML model.
- Expand the rule-based lexicon with context-sensitive slang such as `fire`, `wicked`, and multiple meanings of `sick`.
- Improve sarcasm handling, possibly with pattern-based rules rather than only token counts.
- Make `explain()` reflect the actual custom scoring logic instead of only the starter positive and negative word lists.
- Try TF-IDF or a slightly richer model for the ML version once there is enough data to justify it.

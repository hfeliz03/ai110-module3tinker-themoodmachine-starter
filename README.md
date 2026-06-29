# The Mood Machine

The Mood Machine is a simple text classifier that begins with a rule based approach and can optionally be extended with a small machine learning model. It tries to guess whether a short piece of text sounds **positive**, **negative**, **neutral**, or even **mixed** based on patterns in your data.

This lab gives you hands on experience with how basic systems work, where they break, and how different modeling choices affect fairness and accuracy. You will edit code, add data, run experiments, and write a short model card reflection.

---

## Repo Structure

```plaintext
├── dataset.py         # Starter word lists and example posts (you will expand these)
├── mood_analyzer.py   # Rule based classifier with TODOs to improve
├── main.py            # Runs the rule based model and interactive demo
├── ml_experiments.py  # (New) A tiny ML classifier using scikit-learn
├── model_card.md      # Template to fill out after experimenting
└── requirements.txt   # Dependencies for optional ML exploration
```

---

## Getting Started

1. Open this folder in VS Code.
2. Make sure your Python environment is active.
3. Install dependencies:

    ```bash
    pip install -r requirements.txt
    ```

4. Run the rule-based starter:

    ```bash
    python main.py
    ```

If pieces of the analyzer are not implemented yet, you will see helpful errors that guide you to the TODOs.

To try the ML model later, run:

```bash
python ml_experiments.py
```

---

## What You Will Do

During this lab you will:

- Implement the missing parts of the rule based `MoodAnalyzer`.
- Add new positive and negative words.
- Expand the dataset with more posts, including slang, emojis, sarcasm, or mixed emotions.
- Observe unusual or incorrect predictions and think about why they happen.
- Train a tiny machine learning model and compare its behavior to your rule based system.
- Complete the model card with your findings about data, behavior, limitations, and improvements.
- The goal is to help you reason about how models behave, how data shapes them, and why even small design choices matter.

---

## Tips

- Start with preprocessing before updating scoring rules.
- When debugging, print tokens, scores, or intermediate choices.
- Ask an AI assistant to help create edge case posts or unusual wording.
- Try examples that mislead or confuse your model. Failure cases teach you the most.

---

## TF Notes

This lab is a nice fit for TF because students can see the full loop without getting buried in math or tooling. They edit a few rules, add a few examples, and immediately see how those choices change predictions. That makes it easier to talk about concepts like labeling, ambiguity, bias, and overfitting in a way that feels concrete instead of abstract.

The biggest thing to watch for is that students may assume the model is "smart" when it is really just reacting to tokens and labels. That is actually the teaching opportunity. The rule-based version helps them notice how brittle hand-written logic can be, and the ML version helps them see how quickly a model can appear accurate when it is trained and tested on the same tiny dataset.

What tends to help most:

- Encourage students to make realistic posts, not obviously positive or negative ones only.
- Have them inspect wrong predictions out loud and explain what the model noticed.
- Remind them that disagreement in labels is part of the lesson, especially with sarcasm, slang, and mixed emotions.
- Frame the ML result carefully: a high score here does not mean the model truly understands language.

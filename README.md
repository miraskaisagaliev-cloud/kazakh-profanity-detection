# Kazakh Profanity Detection

This project trains a binary classifier for Kazakh reviews and comments.
The model takes one comment and returns a predicted label and a score for class `1`.

- `0`: No Kazakh profanity.
- `1`: Kazakh profanity is present.

A negative review is not automatically profane.

## Dataset

I used `kazakh_profanity_share.xlsx`, a labeled dataset prepared by
[add the dataset contributor's name or agreed credit].

- **Dataset access:** [Download the exact Excel file used in this project](kazakh_profanity_share.xlsx).
- **Text source and selection:** [add the confirmed source and explain how the comments were selected].
- **Original labeling method:** [add the confirmed method, or state that it was not documented].
- **Synthetic examples:** [state whether any were included; if unknown, say so].

The file contains comment IDs, text, binary labels, and predefined train,
validation, and test splits. The examples I inspected include reviews of games
and apps. The test results do not establish performance specifically on reviews
of physical products.

### Labeling rules and limitations

The task targets Kazakh profanity, not negative sentiment or insults in general.
A mixed-language comment qualifies for label `1` if it contains Kazakh profanity.
Russian profanity alone does not qualify. A masked or misspelled expression
qualifies if the Kazakh profanity is recognizable from context. Ambiguous cases
should be reviewed manually.

These are the intended task rules, not a verified description of how the
supplied labels were assigned. I kept the original labels and did not
independently reannotate the full dataset. At least one inspected comment
with Russian profanity but no clear Kazakh profanity was labeled `1`.
The evaluation therefore measures agreement with the supplied labels.

Examples from the dataset:

| Comment or excerpt | Supplied label | Note |
|---|---:|---|
| `Жамаан ойын` | 0 | Negative review without clear profanity. |
| `ҚОТАҚБАСТАР БТС НАХУ ҚОСТЫҢДАР …` | 1 | Mixed-language excerpt containing Kazakh profanity. |

### Data preparation and splits

The original file contained 3,061 rows. I removed two duplicate rows
found after normalizing Unicode, case, and whitespace. I then screened
for similar texts across splits using character n-grams and a cosine
similarity threshold of 0.85. I manually reviewed 16 candidate pairs,
confirmed one duplicate pair, and removed one copy. I did not reassign
any remaining comments to another split.

I used the predefined splits. Their original random seed was not available
to me. After duplicate removal, the final dataset contained 3,058 comments.

| Split | Label 0 | Label 1 | Total |
|---|---:|---:|---:|
| Train | 1,300 | 1,220 | 2,520 |
| Validation | 110 | 158 | 268 |
| Test | 112 | 158 | 270 |
| Total | 1,522 | 1,536 | 3,058 |

## Model

I trained a logistic regression classifier on character TF-IDF features.
The vectorizer converts text to lowercase, uses character n-grams of
length 2–5, and keeps fragments that appear in at least two training
comments. These fragments can help the model handle spelling variations
without requiring an exact match of whole words.

Comments are not truncated, and punctuation is not manually removed.
Character fragments outside the training vocabulary are ignored.
The classifier uses `C=1.0`, `solver="lbfgs"`, and `max_iter=1000`.

I fitted both the vectorizer and classifier on the training set only.
I selected the decision threshold by maximizing class-1 F1 on the
validation set. The selected threshold was `0.50`.

## Reproduce training in Google Colab

1. Download [the training notebook](train.ipynb) and
   [requirements.txt](requirements.txt).
2. Open the notebook in a fresh Colab runtime and upload `requirements.txt`
   to `/content/`. In a code cell, run `%pip install -r /content/requirements.txt`.
3. Download [kazakh_profanity_share.xlsx](kazakh_profanity_share.xlsx)
   and upload it to `/content/` in Colab. Keep the original filename.
4. Run the notebook from top to bottom to prepare the data, train the model,
   select the threshold, and evaluate the saved model.

## Predict a new comment

1. Download [the prediction notebook](predict_demo.ipynb) and
   [the trained model](kazakh_profanity_model.joblib).
2. Open the notebook in a fresh Colab runtime and install the pinned
   dependencies from `requirements.txt`.
3. Run the model-loading cell and upload `kazakh_profanity_model.joblib`
   when prompted. Then run the prediction cell and enter a comment.

The saved file contains the fitted TF-IDF vectorizer, the classifier,
and the selected threshold. No dataset, separate tokenizer, or retraining
is required for prediction.

Example input: `Тауар сапалы, маған ұнады`

```text
Predicted label: 0
Score for class 1: 0.1243
```

## Test results

I evaluated the saved model on 270 held-out comments at the threshold
selected on validation (`0.50`).

| Metric for class 1 | Score |
|---|---:|
| Precision | 0.9189 |
| Recall | 0.8608 |
| F1 | 0.8889 |

| Supplied label | Predicted 0 | Predicted 1 |
|---|---:|---:|
| 0 | 100 | 12 |
| 1 | 22 | 136 |

The model flagged 12 comments labeled `0` that could have been sent for
unnecessary review. It missed 22 comments labeled `1` that could have
gone unreviewed. These counts are relative to the supplied labels.

Files: [Test metrics](test_metrics.json) ·
[Confusion matrix](confusion_matrix.png) ·
[Validation threshold results](validation_thresholds.csv).

### Error analysis

- `Бұл праграма ұнадыма…` — supplied label `0`, predicted `1`,
  score `0.5177`. The comment contains non-standard spelling and repeated
  emojis, but I did not verify which features drove the prediction.
- `Күшті амы шыккан ойын` — supplied label `1`, predicted `0`,
  score `0.4807`. The expression is worth reviewing in context before
  deciding whether this is a model error or a labeling issue.
- `Онша емес қосылмаид` — supplied label `0`, predicted `1`,
  score `0.5205`. This is a short negative review with no clear Kazakh
  profanity. It illustrates why negative sentiment should not be treated
  as profanity.

A practical improvement would be to add manually verified negative
reviews without profanity and review ambiguous and mixed-language
examples in the training data. I did not change the model or threshold
after inspecting the test predictions.

## Acknowledgments and contributions

A friend prepared the labeled dataset. I checked the data for duplicates,
trained and evaluated the model, and verified that the saved model could
make predictions in Colab without retraining. I used AI to help plan the
project, draft code, and edit the README.

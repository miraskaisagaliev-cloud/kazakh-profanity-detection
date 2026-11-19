# Kazakh Profanity Detection

This project trains a binary classifier for Kazakh reviews and comments.
The model takes one comment and returns a label and a score for class 1.

- `0`: No Kazakh profanity.
- `1`: Kazakh profanity is present.

A negative review is not automatically profane.

## Dataset

I used `kazakh_profanity_share.xlsx`, a labeled dataset prepared by
[credit the person who prepared the file, with their permission].
Dataset access: [add the link or exact access instructions].
Text source and selection method: [add the confirmed source and explain
how comments were selected].
Original labeling method: [add the confirmed method, or state that
it was not documented].
Synthetic examples: [state what is confirmed about their presence].

The file contains comment IDs, raw text, binary labels, and predefined
train, validation, and test splits. The examples I inspected include
reviews about games and apps. The results do not establish performance
specifically on reviews of physical products.

### Labeling rules and limitations

The task targets Kazakh profanity, not negative sentiment or insults
in general. A mixed-language comment qualifies for label `1` if it
contains Kazakh profanity. Russian profanity alone does not qualify.
A masked or misspelled expression qualifies if the Kazakh profane
expression is recognizable from context. Ambiguous cases should be
reviewed manually.

These are the task rules, not a verified description of how every
original label was assigned. I kept the supplied labels and did not
independently reannotate the full dataset. At least one inspected
comment containing Russian profanity but no clear Kazakh profanity
was labeled `1` in the supplied file. The evaluation therefore measures
agreement with the supplied labels.

### Data preparation and splits

The original file contained 3,061 rows. I removed two duplicate rows
found after Unicode, case, and whitespace normalization. I then screened
for similar texts across splits using character n-grams and a cosine
similarity threshold of 0.85. Of 16 candidate pairs, one was confirmed
as a duplicate, and one copy was removed. No remaining rows were
reassigned to another split.

I used the predefined splits. Their original random seed
[was not provided / was ...].

After duplicate removal, the final dataset contained 3,058 comments.

| Split | Label 0 | Label 1 | Total |
|---|---:|---:|---:|
| Train | 1,300 | 1,220 | 2,520 |
| Validation | 110 | 158 | 268 |
| Test | 112 | 158 | 270 |
| Total | 1,522 | 1,536 | 3,058 |

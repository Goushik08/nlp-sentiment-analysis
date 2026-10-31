# nlp-sentiment-analysis
Sentiment analysis of WhatsApp group chats in English, Tamil and Tanglish using two Hugging Face models. Cleans the export, classifies each message, and summarises results overall and per sender.
# WhatsApp Chat Sentiment Analysis (English + Tamil + Tanglish)

Clean an exported WhatsApp group chat and estimate the sentiment of each message (Positive / Neutral / Negative), overall and per sender. Works on English, Tamil and Tanglish (Tamil written in English letters) by combining two pretrained Hugging Face models.

> ⚠️ **Privacy first.** A chat export contains private conversations. This repository contains **code only**. Never commit your `group.txt` or the result CSVs (they contain full messages and names). The included `.gitignore` blocks them. If you share screenshots, blur or rename the people. Only analyse chats where the participants are fine with it.

## What it does

1. **Parses** the WhatsApp export (`date, time - Name: message`), including multi-line messages.
2. **Cleans** the data: separates system notices ("X added you"), media-only messages (`<Media omitted>`) and real text messages.
3. **Classifies sentiment** with two models:
   - Tamil-mix sentiment model for Tamil and Tanglish
   - Multilingual sentiment model as a fallback for messages the first model marks as "not Tamil" (English, emojis and so on)
4. **Summarises** results overall and per sender, with the model's average confidence.
5. **Exports** everything to CSV files.

## Models

| Role | Model (Hugging Face) |
|---|---|
| Tamil / Tanglish | `vishnun/bert-base-cased-tamil-mix-sentiment` |
| Fallback (multilingual) | `Horizon-Labs/multilingual-sentiment-small` |

The Tamil model's classes (positive, negative, mixed feelings, unknown, not-Tamil) are mapped to three labels: mixed feelings and unknown become **Neutral**.

## Example run (one family group chat)

| Step | Count |
|---|---|
| Lines in the export | 1,627 |
| Cleaned messages | 1,065 |
| System notifications | 12 |
| Media-only messages | 690 |
| Messages analysed | 363 |
| Sent to multilingual fallback | 144 |

Overall sentiment: **77.4% Positive, 15.7% Neutral, 6.9% Negative**.

These numbers come from one private chat and are only an example. Many messages were short emojis or greetings, which models tend to score as positive, so read the result as "very friendly chat" rather than a precise measurement.

## Project structure

```
whatsapp-sentiment-analysis/
├── whatsapp_sentiment.ipynb     # rename to match your notebook
├── requirements.txt
├── .gitignore                   # keeps private chat data out of Git
└── README.md
```

## Quick start

```bash
git clone <your-repo-url>
cd <your-repo>/whatsapp-sentiment-analysis
python -m venv .venv
# Windows: .venv\Scripts\activate     macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
jupyter notebook
```

1. In WhatsApp: open the chat → **Export chat → Without media**. You get a `.txt` file.
2. Put it next to the notebook as `group.txt` (it is git-ignored).
3. In the configuration cell, set:
   ```python
   FILE_PATH = Path("group.txt")
   ```
   Do not leave a personal path such as `C:\Users\<name>\Downloads\...` in code you publish.
4. Run the cells in order. The first run downloads the two models (internet needed).

## Output files

Saved to `whatsapp_analysis_output/` (git-ignored):

| File | Content |
|---|---|
| `cleaned_all_messages.csv` | All parsed messages |
| `system_notifications.csv` | Join/leave/icon/encryption notices |
| `media_messages.csv` | Media-only messages |
| `sentiment_results.csv` | Each message with sentiment, confidence and model used |
| `overall_sentiment_summary.csv` | Positive / Neutral / Negative counts and percentages |
| `sender_summary.csv` | Per-sender message counts, sentiment split, average confidence |

## Configuration

| Setting | Meaning |
|---|---|
| `FILE_PATH` | Path to the exported chat |
| `OUTPUT_DIR` | Where CSV results are saved |
| `TAMIL_MODEL`, `MULTILINGUAL_MODEL` | Hugging Face model names |
| `BATCH_SIZE` | Messages per batch (lower it if memory is tight) |
| `DEVICE` | Uses the GPU automatically if available, otherwise CPU |

## Limitations

- **Sarcasm, slang and context** are hard for sentiment models, especially in Tanglish. Check a sample of the results by hand.
- **Emojis and short replies** are usually scored positive, which can inflate the positive share.
- Mixed-feelings and unknown labels are folded into Neutral, which hides some nuance.
- Model confidence is not the same as correctness.
- Date/time parsing expects exports like `14/07/19, 12:50 pm - Name: message`. Other regional formats may need a regex change.
- Media-only messages are skipped, so photo-heavy chats have far fewer analysed messages than total messages.

## Troubleshooting

| Problem | Fix |
|---|---|
| `File not found` | Check `FILE_PATH` and the file name |
| No messages parsed | Your export format differs; adjust `message_pattern` |
| Hugging Face warnings about symlinks or `HF_TOKEN` | Harmless; set `HF_TOKEN` for faster downloads if you want |
| Out of memory | Lower `BATCH_SIZE` |
| `ModuleNotFoundError` | `pip install -r requirements.txt` in the same environment as Jupyter |

## Roadmap

- [ ] Charts: sentiment over time, per sender, by hour of day
- [ ] Most active days and busiest hours
- [ ] Word clouds and most-used emojis
- [ ] Anonymise sender names automatically

## Built with

Python, pandas, PyTorch, Hugging Face Transformers.


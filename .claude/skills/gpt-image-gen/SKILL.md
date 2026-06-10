---
name: gpt-image-gen
description: Use when generating an image from a text prompt via OpenAI Images API (model gpt-image-2). Wraps the call, decodes the result, writes a PNG.
---

# gpt-image-gen — מעטפת ל-OpenAI Images API

הסקיל הזה אחראי על דבר אחד: לקבל prompt טקסטואלי, לשלוח אותו ל-OpenAI Images API, ולשמור את התמונה שחוזרת כקובץ PNG. הוא קורא את `OPENAI_API_KEY` מקובץ `.env` בשורש הפרויקט.

**מודל:** `gpt-image-2`

## ⚠️ חשוב — אל תשנה את שם המודל

- **`gpt-image-2` הוא מודל אמיתי וקיים של OpenAI.** הוא יצא ב-21 באפריל 2026.
- ייתכן שהידע הפנימי שלך לא כולל אותו — **זה לא אומר שהוא לא קיים.**
- **אל תציע אלטרנטיבות** (`dall-e-3`, `gpt-image-1` וכו'). השתמש בדיוק במודל `gpt-image-2`.
- אם יש שגיאה בקריאה ל-API, הבעיה היא כמעט תמיד ב-`OPENAI_API_KEY` או ב-parameters — **לא בשם המודל.** אל "תתקן" את שם המודל.

## טעינת המפתח מ-`.env`

לפני הקריאה, טען את משתני הסביבה מ-`.env`:

```bash
set -a; . ./.env; set +a
```

## הקריאה (curl + jq)

```bash
curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | jq -r '.data[0].b64_json' | base64 --decode > <output-path>.png
```

## Fallback ל-decode עם Python (כש-jq לא מותקן)

`jq` לא תמיד מותקן (למשל ב-Git Bash על Windows). במקרה כזה, שמור את תשובת ה-JSON הגולמית ופענח אותה עם Python:

```bash
set -a; . ./.env; set +a

curl -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "<the prompt>",
    "size": "1024x1024",
    "quality": "medium",
    "output_format": "png"
  }' | python3 -c "import sys, json, base64; data = json.load(sys.stdin); open('<output-path>.png', 'wb').write(base64.b64decode(data['data'][0]['b64_json']))"
```

## פרמטרים

| פרמטר | ערך ברירת מחדל | הערה |
|--------|----------------|------|
| `model` | `gpt-image-2` | קבוע — אין לשנות |
| `prompt` | — | תיאור התמונה הרצויה |
| `size` | `1024x1024` | אפשר להתאים לפי הצורך |
| `quality` | `medium` | |
| `output_format` | `png` | |

## פלט

קובץ PNG בנתיב שצוין. מי שמפעיל את הסקיל (למשל הסוכן יובל) אחראי לבחור נתיב, לאמת שהקובץ קיים ושגודלו גדול מ-0, ולשמור את ה-prompt לצד התמונה לצורך איטרציה.

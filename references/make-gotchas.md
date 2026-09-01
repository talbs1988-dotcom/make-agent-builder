# מלכודות Make — כולן עלו בבנייה אמיתית, 1.9.2026

## אימות לפני יצירה — הכלל שמונע את רוב הכאב

`validate_module_configuration` מחזיר את השדות החסרים במדויק. תרחיש שנוצר בלי אימות מוקדם
נראה תקין (`isinvalid: false`) ונופל רק כשמנסים להריץ:

```
BlueprintValidationError: Scenario validation failed - N problem(s) found.
```

**שים לב:** מספר הבעיות מוכפל במספר הענפים. שלוש שגיאות בראוטר עם שני מסלולים = 6 בעיות.

## שדות חובה שלא מופיעים בטופס

| מודול | מה חסר | הערך |
| --- | --- | --- |
| `google-calendar:createAnEvent` במצב `detail` | `transparency` | `"opaque"` |
| `google-calendar:createAnEvent` במצב `detail` | `visibility` | `"default"` |

שניהם מסומנים "advanced" בסכימה, ובכל זאת **mandatory**. בלעדיהם התרחיש לא יעלה.

## סוגי נתונים

`google-calendar:searchEvents` → `limit` חייב להיות **מספר** ולא מחרוזת. `40` ולא `"40"`.
זה נכון לכל שדה שמוגדר `type: number` — Make לא ממיר.

## Google Calendar — quick מול detail

`select: "quick"` מקבל **שדה טקסט אחד בלבד**. אין משך, אין משתתפים, אין תיאור, אין וידאו,
וברירת המחדל היא **שעה שלמה**. סוכן שמונחה "צור פגישה של 20 דקות עם זום" יקבל שעה בלי כלום,
ויספר ללקוח שהזימון נשלח.

`select: "detail"` נותן: `summary` · `start` · `duration` · `attendees` · `description` ·
`conferenceDate` (מוסיף **Google Meet**, לא זום).

## Webhooks

- **webhook שייך לתרחיש אחד.** ניסיון לשייך אותו לתרחיש שני מחזיר
  `The hook already has a scenario assigned`. יוצרים webhook חדש עם `hooks_create` על אותו חיבור.
- ל-WhatsApp: `typeName: "whatsapp-business-cloud"`, `data: {"__IMTCONN__": <id>, "events": ["messages"]}`
- **`queueCount` בווב-הוק מראה כמה הודעות אמיתיות ממתינות.** אם הוא גדול מאפס — הודעות כן הגיעו,
  התרחיש פשוט היה כבוי.

## חיבורים

- **אי אפשר ליצור חיבור מבחוץ.** המשתמש חייב ללחוץ "התחבר" בעצמו. זו מגבלה של Make.
- כל מודול דורש **סוג** חיבור מסוים. `google-email:ActionSendEmail` דורש `google-restricted` —
  חיבור Gmail רגיל יידחה עם `is not compatible with ... module`.
- חיבור מהסוג הנכון עדיין יכול להיכשל על הרשאות: `403 insufficientPermissions`. לבדוק באימות.
- `connections_list` מחזיר את המזהים האמיתיים. **לא להמציא מזהי חיבור.**

## WhatsApp Business Cloud

- `fromId` הוא **מזהה המספר**, לא המספר. מקבלים אותו מ-`rpc_execute` עם `listPhoneNumbers`.
- `contacts[].wa_id` שמגיע מהטריגר **כבר כולל קידומת מדינה**. הוספת `+972` לפניו יוצרת
  `+972972...` וההודעה לא תגיע.
- לכל WABA יש `fromId` משלו וחיבור משלו. מספר שני = webhook חדש + חיבור חדש + fromId חדש.

## Airtable

- `airtable:ActionSearchRecords` · `ActionCreateRecord` · `ActionUpdateRecords` (שים לב לרבים ב-Update).
- `maxRecords` — מספר. `formula` — נוסחת Airtable אמיתית, למשל `FIND("972501234567", {נייד} & "")`.
- **`typecast: false` יפיל כתיבה לשדה singleSelect עם ערך שלא קיים ברשימה.** `typecast: true`
  יוצר את האפשרות אוטומטית — הפתרון כששדה סטטוס ריק מאפשרויות.
- שדה singleSelect יכול להיות קיים ועדיין `choices: []`. לבדוק ב-`get_table_schema`.

## סוכן ה-AI (`ai-local-agent:RunLocalAIAgent`)

- הכלים יושבים במערך `tools` **בתוך מודול הסוכן**, כל אחד `{name, description, flow:[module]}`.
- `{{<agentId>.<שם>}}` בתוך מיפוי של כלי = פרמטר שהסוכן ממלא בזמן ריצה.
- `threadId` = מזהה השיחה. מספר טלפון שם נותן **זיכרון נפרד לכל ליד** — זה נכון ורצוי.
- הפניה למשתנה שלא קיים (למשל שם בעברית שלא הוגדר ב-SetVariables) עוברת אימות ומחזירה **ריק**.

## Google Sheets כמקור ידע

גיליון שבנוי כמסמך — שורות ממוזגות מעל הכותרות, שתי טבלאות בלשונית אחת — יחזיר את שורות
הכותרת כאילו הן נתונים. **לשונית אחת, כותרות בשורה הראשונה, שורה אחת לכל פריט.**

## Gemini

- `gemini-ai:createACompletionGeminiPro` → הפלט הוא `{{<id>.candidates[].content.parts[].text}}`.
- תמלול הקלטה: `getMedia` → `uploadAFile` → completion. אם המשתנה אחרי התמלול עדיין קורא
  מ-`text.body` — התמלול נזרק לפח והסוכן מקבל הודעה ריקה.
- המסלול החינמי: 15 בקשות בדקה, 1,000 ביום, בלי כרטיס אשראי — אבל גוגל רשאית להשתמש בתוכן.
  לפני לקוחות אמיתיים: לחבר חיוב.

## גודל בלופרינט

בלופרינט מלא מ-`scenarios_get` מגיע ל-140KB בגלל `metadata` של כל מודול. לשמור רק
`metadata.designer` — זה מוריד אותו לרבע ומאפשר להעביר אותו ב-API.

# 🔄 מדריך להגדרת סנכרון דו־כיווני בין שני מאגרי GitHub (ללא לולאות אינסופיות)

## שלב 1: יצירת מפתח הרשאה (Token)

1. ב־GitHub לחצו על תמונת הפרופיל שלכם → **Settings**
2. גללו ל־**Developer settings**
3. היכנסו ל־**Personal access tokens**
4. בחרו **Tokens (classic)**
5. צרו Token חדש
6. סמנו את ההרשאות:
   - `repo`
   - `workflow`
7. העתיקו את ה־Token (תצטרכו אותו מיד)

---

# שלב 2: הוספת ה־Token לשני המאגרים

בכל אחד משני המאגרים (Lovable + AI Studio):

1. היכנסו ל־Repository
2. עברו ל־**Settings**
3. היכנסו ל־**Secrets and variables**
4. לחצו על **Actions**
5. צרו Secret חדש:
   - Name:
     ```txt
     SYNC_TOKEN
     ```
   - Value:
     הדביקו את ה־Token שיצרתם

---

# שלב 3: הסקריפט למאגר הראשון (Lovable)

צרו קובץ:

```txt
.github/workflows/sync-to-studio.yml
```

והדביקו:

```yaml
name: Sync to AI Studio Repo

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest

    # הגנה קריטית מפני לולאת עדכונים אינסופית
    if: github.actor != 'github-actions[bot]' && !contains(github.event.head_commit.message, 'Auto-sync updates')

    steps:
      - name: Checkout source repo
        uses: actions/checkout@v4
        with:
          path: source-repo
          token: ${{ secrets.SYNC_TOKEN }}

      - name: Checkout target repo
        uses: actions/checkout@v4
        with:
          # 👇 החליפו לשם המשתמש והמאגר של AI Studio
          repository: USERNAME/AI-STUDIO-REPO
          token: ${{ secrets.SYNC_TOKEN }}
          path: target-repo

      - name: Sync files (excluding workflows)
        run: |
          rsync -av --delete \
            --exclude='.git/' \
            --exclude='.github/workflows/' \
            source-repo/ target-repo/

      - name: Commit and push changes
        run: |
          cd target-repo

          git config user.name "GitHub Sync Bot"
          git config user.email "action@github.com"

          git add .

          if ! git diff-index --quiet HEAD; then
            git commit -m "Auto-sync updates from Lovable"
            git push
          else
            echo "No changes to sync."
          fi
```

---

# שלב 4: הסקריפט למאגר השני (AI Studio)

צרו קובץ:

```txt
.github/workflows/sync-to-lovable.yml
```

והדביקו:

```yaml
name: Sync to Lovable Repo

on:
  push:
    branches: [main]

jobs:
  sync:
    runs-on: ubuntu-latest

    if: github.actor != 'github-actions[bot]' && !contains(github.event.head_commit.message, 'Auto-sync updates')

    steps:
      - name: Checkout source repo
        uses: actions/checkout@v4
        with:
          path: source-repo
          token: ${{ secrets.SYNC_TOKEN }}

      - name: Checkout target repo
        uses: actions/checkout@v4
        with:
          # 👇 החליפו לשם המשתמש והמאגר של Lovable
          repository: USERNAME/LOVABLE-REPO
          token: ${{ secrets.SYNC_TOKEN }}
          path: target-repo

      - name: Sync files (excluding workflows)
        run: |
          rsync -av --delete \
            --exclude='.git/' \
            --exclude='.github/workflows/' \
            source-repo/ target-repo/

      - name: Commit and push changes
        run: |
          cd target-repo

          git config user.name "GitHub Sync Bot"
          git config user.email "action@github.com"

          git add .

          if ! git diff-index --quiet HEAD; then
            git commit -m "Auto-sync updates from AI Studio"
            git push
          else
            echo "No changes to sync."
          fi
```

---

# ✅ סיימתם!

מעכשיו שני המאגרים יתעדכנו אוטומטית אחד מהשני ברקע — בלי לולאות אינסופיות 🚀

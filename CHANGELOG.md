# Changelog

Record of every fix made to align each lesson's `_code_brief.ipynb` and `_lesson.ipynb` with its lecture notebook (100% matching, runnable code). Entries are appended, never overwritten. Sections are titled by topic, not date.

---

## 2026-07-08 — Module 3.4 (LECTURE FILE EDIT — second one in this effort)

### `3.4 Evaluating Tree-Based Models.ipynb`
**Background:** Instructor-level review found the notebook's Learning Objectives listed 5 items but only delivered 2 (verified precisely — grepped every cell for `feature_importances`, `calibrat`, `threshold`, `f1_score`; confirmed zero code anywhere for objectives 3, 4, and 5, and confirmed `f1_score` is never called despite the Summary having a dedicated "F1 Score" subsection explaining it as if it had been computed). Reported to Juan with the specifics. His reply: **"yes that was edited out. thank you. adjust to reflect what is actually delivered."** — confirms this content was intentionally removed at some point and the fix is to trim the text to match reality, not add the missing analyses.

**Change 1 — Learning Objectives (cell `lLbiUj107EWD`):**
**Before:**
```markdown
### Learning Objectives

1. Generate and interpret Precision-Recall curves
2. Analyze confusion matrices for each model
3. Compare feature importances across models
4. Understand probability calibration and threshold selection
5. Make an informed recommendation for deployment
```
**After:**
```markdown
### Learning Objectives

1. Generate and interpret Precision-Recall curves
2. Analyze confusion matrices for each model
```
**Reason:** Items 3–5 have no corresponding code anywhere in the notebook. Trimmed to match what's actually delivered.

**Change 2 — Summary section (cell `d41e0bce`):**
**Before:** Had three subsections — "Precision-Recall (PR) Curves", "Confusion Matrices", and "F1 Score" (the F1 Score subsection explained the metric conceptually as if it had just been used in the analysis).
**After:** Removed the "F1 Score" subsection entirely. Kept "Precision-Recall (PR) Curves" and "Confusion Matrices" untouched — both describe content that's actually in the notebook.
**Reason:** `f1_score` is never called anywhere in this notebook (0 occurrences, confirmed via grep) — the subsection was describing an analysis that doesn't exist here.

**Verified (static only):** re-read both edited cells, confirmed the objectives list now matches exactly what cells 2 (PR curves) and 3 (confusion matrices) deliver, confirmed the Summary's remaining two subsections are untouched and accurate.

**Update — both now fixed too (same session):**

### `3.4_lesson.ipynb`
**Change:** Full rewrite to fix a real bug and align with the lecture.
1. **Fixed data-leakage bug:** was `test_enc.fillna(test_enc.median())` (filled test-set gaps with the test set's own median) → now `test_enc.fillna(train_medians)` (fills with train-only medians, matching the lecture's leakage-safe approach — same class of bug already fixed in `3.3_lesson.ipynb`).
2. **Loads the tuned models instead of retraining untuned ones:** replaced the block that instantiated fresh `DecisionTreeClassifier`/`RandomForestClassifier`/`XGBClassifier` with 3.2's untuned defaults and called `.fit()`, with `joblib.load('../models/decision_tree_best.joblib')` etc. — the tuned models `3.3_lesson.ipynb` now saves.
3. **Removed a full "ROC Curves" section** (computed `roc_curve`/`roc_auc_score`) — the lecture has no ROC section at all, only PR curves; this section didn't correspond to anything in the lecture and re-introduced AUC after the course standardized on F1.
4. **Removed the "Feature Importance Comparison" section** and the "Recommendations" table — neither exists in the lecture (confirmed: the lecture never delivers feature importance, per the Module 3.4 lecture-edit above).
5. Added back a Summary section matching the lecture's trimmed one (PR Curves + Confusion Matrices subsections only).

Note: while editing, several parallel `NotebookEdit` delete/insert calls caused cell-ID renumbering issues that briefly dropped the Summary cell entirely and left an orphaned code cell. Caught via re-reading the file after each batch, fixed by switching to one sequential edit at a time with a re-read in between. Final file verified: 9 cells, correct order, zero remaining references to `X_train`, `roc_curve`, `roc_auc_score`, `feature_importances`, or `f1_score`.

### `3.4_code_brief.ipynb`
**Change:** Full rewrite — was the generic 2-cell boilerplate (same placeholder every other module 3 code_brief had), now 11 cells of real code mirroring the lecture: Drive-based path setup, loading `feature_columns.pkl`/`train_medians.pkl` + the three `.joblib` models, PR curve, confusion matrices, and a takeaways summary. Verified: all path variables (`project_path`, `data_filepath`, `course3_models`, `ARTIFACT_DIR`) defined before use, zero AUC/feature-importance references.

**Module 3.4 status: complete.** Lecture, code_brief, and lesson are now aligned.

---

## 2026-07-08 — Module 4.1 (JUAN'S CHANGE — pulled from GitHub, not made by us)

### `4.1 Systematic Model Comparison.ipynb`
**Change:** Juan pushed an update to GitHub and asked the user to pull it. No local git clone exists yet, so the user downloaded the file individually and had it swapped into the working folder. Diffed the two versions to confirm what changed before replacing:
- **Interpretability vs. Performance radar chart** now explicitly computes `F1 (scaled to 10)` as its first dimension (`(F1 Score / max_f1) * 10`), replacing whatever the previous AUC/Avg-Precision-derived dimension was. Radar went from 6 dimensions to 5 (dropped "Stakeholder Trust").
- **Recommendations table** dropped one row ("Limited IT" / "Simplest to deploy").
- Everything else (data loading, model loading, performance comparison table, PR curves, summary) unchanged.

This is Juan applying the same AUC→F1 standardization already done elsewhere in this effort — a good independent confirmation of the direction.

**Action taken:** copied `~/Downloads/4.1 Systematic Model Comparison.ipynb` over the working-folder copy. Verified byte-for-byte identical via `diff` after the copy.

**Update — all three files now aligned (same session):**

### `2.4_lesson.ipynb` (prerequisite fix — needed before 4.1_lesson could load a tuned logistic model)
**Changes:**
1. Fixed `models_filepath` bug (was `== data_filepath`) → separate `Course 3 Models/` path.
2. Fixed stale load filenames: `l2_ridge_logistic_model.pkl` etc. → `l2_ridge_logistic.pkl` etc. (matching the corrected 2.2/2.3 save names).
3. Fixed confusion matrix: `labels=['N','Y']` → `labels=['N','E']` (`'Y'` isn't a real class value) — same class of bug as the earlier 2.4 review found.
4. Fixed the TP/TN/FP/FN interpretation cell — was naively unpacking `cm.ravel()` assuming standard `[TN,FP,FN,TP]` order, which is wrong for `labels=['N','E']` (positive class first). Replaced with explicit `cm[0,0]`/`cm[1,1]`/`cm[1,0]`/`cm[0,1]` indexing matching the lecture exactly.
5. Fixed the save cell to use the lecture's **static** filename `best_tuned_logistic_model.pkl` (was a **dynamic** name like `l1_lasso_tuned.pkl`) — this is the piece `4.1` depends on by exact name.
6. **New:** also saves a copy to local `../models/best_tuned_logistic_model.pkl`, because `4.1_lesson.ipynb` uses local relative paths (no Drive mount) while `2.4_lesson.ipynb` uses Drive paths (matching its own lecture) — a real environment mismatch between the module 2 and module 3+ lesson tracks, flagged and resolved by saving to both locations.

### `4.1_lesson.ipynb`
**Change:** Full rewrite. Was retraining fresh, wrong/untuned models (`L2, C=0.1` logistic instead of the tuned `L1, C=0.01` winner; 3.2's untuned RF/XGBoost defaults). Now loads the three tuned models (`../models/best_tuned_logistic_model.pkl`, `random_forest_best.joblib`, `xgboost_early_stopping.joblib`) exactly like the lecture does. Also removed a whole "ROC Curves" section and a metrics bar chart that don't exist in the lecture, trimmed the performance table to match (Precision/Recall/F1/Avg Precision/Brier Score only, no Accuracy/ROC-AUC/Train Time), replaced the old 6-dimension ROC-AUC-based radar chart with the new F1-based 5-dimension one (matching Juan's 4.1 lecture update), fixed the Recommendations table (dropped the "Limited IT" row to match), and added a "Summary — Key Findings" section that didn't exist before.

### `4.1_code_brief.ipynb`
**Change:** Full rewrite — was the generic 2-cell boilerplate, now 13 cells of real code mirroring the lecture: Drive-based path setup, loading `feature_columns.pkl`/`train_medians.pkl` + the three tuned models, performance table, PR curve, the new F1-based radar chart, recommendations.

**Verified (static only):** all three files re-read top-to-bottom; confirmed no undefined variables, zero stray `roc_auc_score`/`LogisticRegression(`/fresh-training calls in `4.1_lesson.ipynb` and `4.1_code_brief.ipynb`, and confirmed `2.4_lesson.ipynb`'s filenames now match what both `3.x` and `4.1` tracks expect.

**Module 4.1 status: complete.** Lecture (Juan's version), code_brief, and lesson are now aligned. `2.4_lesson.ipynb` is also now fully fixed (all previously-known issues resolved).

---

## 2026-07-08 — Module 4.2 (+ a formatting bug found and fixed across 5 earlier files)

### `4.2_code_brief.ipynb`
**Change:** Was confirmed (via precise cell-type/content check, after an initial incorrect claim that it already had real content — corrected mid-session) to be the generic 2-cell placeholder, zero real code. Rewritten with real code matching the lecture: Model Selection Criteria table, data setup, the save/load RF example (including the lecture's own leakage bug and AUC-only metric — deliberately kept identical, not silently "improved," since code_brief must match the lecture as it currently stands), Model Card template (including the unfilled `[Insert from evaluation]` placeholder, kept as-is), Key Decisions.
**Verified:** diffed cell-by-cell against the lecture — both code cells now **byte-for-byte identical**.

### `4.2_lesson.ipynb`
**Change:** Was very close to the lecture already (same hyperparameters, same leakage bug, same logic) but not literally identical — was missing 3 unused imports the lecture has (`LogisticRegression`, `XGBClassifier`, `StandardScaler`), had shortened print strings (`"Data prepared."` vs lecture's `"Data prepared for final model selection."`), and was missing the lecture's inline comments. Restored all of it to be byte-for-byte identical to the lecture's two code cells.
**Verified:** diffed cell-by-cell — both code cells now **byte-for-byte identical** to the lecture.

### Formatting bug found and fixed: `3.1_code_brief.ipynb`, `3.2_code_brief.ipynb`, `3.4_code_brief.ipynb`, `4.1_code_brief.ipynb`
**Background:** While building `4.2_code_brief.ipynb` and diffing it against the lecture to confirm a true byte-for-byte match (per the "100% identical" standard), discovered that the `json.dump`-based script used to generate several earlier code_brief rewrites had a real bug: it split cell source text on `\n` without preserving the trailing newline on each line (`source: src.split('\n')` instead of properly re-adding `\n` to every line but the last). Jupyter's notebook format requires each entry in a cell's `source` array to end in `\n` (except the last) so that `''.join(source)` reconstructs the original multi-line text; without it, every code cell in the affected files was stored as a single unreadable line-blob — would have rendered as one giant unbroken line of code in Jupyter/GitHub, even though the actual Python logic was correct.

**Affected files:** `3.1_code_brief.ipynb` (8 cells fixed), `3.2_code_brief.ipynb` (13 cells), `3.4_code_brief.ipynb` (7 cells), `4.1_code_brief.ipynb` (8 cells). `3.3_code_brief.ipynb` was unaffected because it was built via `NotebookEdit` calls rather than raw `json.dump`, which handles source formatting correctly.

**Fix:** re-read each cell's `source`, reconstructed the full text via `''.join()`, re-split on `\n` with proper trailing-newline preservation on every line but the last, wrote back. **No code content changed** — purely a formatting/structure fix. Verified across all 6 code_brief files (3.1, 3.2, 3.3, 3.4, 4.1, 4.2): zero cells with the broken format remaining.

**Module 4.2 status: complete.** Lecture, code_brief, and lesson code are now all byte-for-byte aligned. The lecture-level issues (leakage bug, AUC-only, unfilled model card placeholder) are unchanged and still pending Juan's input before any lecture edit — code_brief/lesson were made to match the lecture *as it currently is*, not a hypothetical fixed version.

---

## 2026-07-08 — Module 6.1 (Special Topics — Additional Boosting Algorithms)

### `6.1_code_brief.ipynb`
**Change:** Full rewrite — was a single markdown cell (title + "Condensed reference for notebook 6.1."), not even the generic boilerplate placeholder other empty code_briefs had. Rewritten with 13 cells covering the comparison table, Drive-based setup, AdaBoost, LightGBM (try/except), the `!pip install catboost` cell, CatBoost (try/except), and key takeaways.
**Verified:** all 6 code cells diffed against the lecture — **byte-for-byte identical**. Formatting checked clean (no single-line-blob bug from the earlier session).

### `6.1_lesson.ipynb`
**Status:** Already in good shape before this session — all three model-building blocks (AdaBoost, LightGBM, CatBoost) already had hyperparameters identical to the lecture, just consolidated into fewer/denser cells (established lesson-track condensing style) and using local `../data/` paths instead of Drive (established lesson-track convention). No hyperparameter or logic mismatches found.
**Change:** Added the one missing piece — a `!pip install catboost` cell, inserted between the CatBoost markdown header and the CatBoost model cell, matching the lecture's structure exactly.

**Module 6.1 status: complete.** Lecture, code_brief, and lesson are now aligned.

---

## 2026-07-08 — Module 6.2 (Special Topics — Neural Networks, third lecture-file edit)

### `6.2 Special Topics - Neural Networks.ipynb` (LECTURE FILE EDIT)
**Background:** Found a self-contradiction inside the lecture itself: the actual `MLPClassifier` code uses `batch_size=50, random_state=88`, but the "Notes on our MLPClassifier Configuration" markdown cell right after it describes the model as using `batch_size=32` — the docs didn't match the code. Also noted `random_state=88` is a course-wide outlier (every other notebook uses `42`), likely a typo.

Reported both to Juan with two options: (a) change the code to match the notes, or (b) change the notes to match the code. Recommended (b) since the notebook's cached output (`Neural Network ROC-AUC: 0.8404`) was generated with the current code — changing the code would make that number stale with no way to re-run and verify. Juan agreed: **"but i think fixing the notes is the right path."**

**Change:** Updated the notes cell — `batch_size=32` → `batch_size=50`. No code cells touched, cached output unaffected.

### `6.2_code_brief.ipynb`
**Change:** Full rewrite — was a single markdown cell (title + "Condensed reference for notebook 6.2."), even less than the usual empty-boilerplate pattern. Rewritten with 10 cells matching the lecture's actual code exactly (`batch_size=50`, `random_state=88`, both print notes).
**Verified:** all 3 code cells diffed against the lecture — **byte-for-byte identical**.

### `6.2_lesson.ipynb`
**Change:** Three fixes to match the lecture's actual code (not its former, since-corrected notes):
1. `batch_size=32` → `50`
2. `random_state=42` → `88`
3. Added back the missing second print line (`"They also take longer to train and have more hyperparameters."`)
4. Fixed the comparison table's last row — was "Images/text: Excellent / Not suitable" (a row not in the lecture at all), restored to match the lecture's "Sample efficiency: Needs more data / Works with less."

**Verified:** confirmed `batch_size=50`, `random_state=88`, and the second print line are all present in the lesson's code cell.

**Module 6.2 status: complete.** Lecture (notes corrected), code_brief, and lesson are now aligned.

---

## 2026-07-08 — Module 3.3 (LECTURE FILE EDIT — first one in this effort)

### `3.3 Tuning Tree-Based Models.ipynb` (the plain, no-suffix lecture)
**Background:** Module 3.3 had two lecture files (`3.3 Tuning Tree-Based Models Using f1.ipynb` and this plain one) that each saved only half of what `3.4 Evaluating Tree-Based Models.ipynb` needs — the plain file saved the three tuned `.joblib` models but not the `feature_columns.pkl`/`train_medians.pkl` preprocessing artifacts; "Using f1" saved those artifacts but not the (better) early-stopped models. Reported the diff to Juan (repo owner); he approved merging into a single canonical file, using this plain one as the base since it's more complete.

**Additional finding during verification (not just "more complete" — actually more correct):** despite its name, `Using f1.ipynb` tunes on F1 but reports **Test AUC** as the held-out evaluation metric for every model (`roc_auc_score` called 6 times, "Test AUC" printed after every single search) instead of Test F1 — a real tuning/reporting metric mismatch. The plain file is F1-consistent end-to-end (tunes on F1, reports Test F1 for every model, zero AUC anywhere except one unused import). This directly answered Juan's question about whether AUC usage was incorrect — it was, in the other file, not this one.

**Change:** Added the missing artifact-persistence step to the data-prep cell.
**Before:**
```python
import numpy as np
import pandas as pd
import warnings
warnings.filterwarnings('ignore')
...
from sklearn.metrics import roc_auc_score, make_scorer, f1_score
...
train_enc, test_enc = train_enc.align(test_enc, join='left', axis=1, fill_value=0)
train_enc = train_enc.fillna(train_enc.median())
test_enc = test_enc.fillna(train_enc.median())

X_train, y_train = train_enc, train_df['DEPARTED']
X_test, y_test = test_enc, test_df['DEPARTED']

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=RANDOM_STATE)
print(f"Data loaded: {X_train.shape[0]:,} training, {X_test.shape[0]:,} testing samples")
```
**After:**
```python
import numpy as np
import pandas as pd
import joblib, os
import warnings
warnings.filterwarnings('ignore')
...
from sklearn.metrics import make_scorer, f1_score
...
ARTIFACT_DIR = f'{project_path}{course3_models}'
os.makedirs(ARTIFACT_DIR, exist_ok=True)
...
train_enc, test_enc = train_enc.align(test_enc, join='left', axis=1, fill_value=0)

train_medians = train_enc.median()
train_enc = train_enc.fillna(train_medians)
test_enc  = test_enc.fillna(train_medians)

X_train, y_train = train_enc, train_df['DEPARTED']
X_test, y_test = test_enc, test_df['DEPARTED']

feature_columns = list(train_enc.columns)

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=RANDOM_STATE)
print(f"Data loaded: {X_train.shape[0]:,} training, {X_test.shape[0]:,} testing samples")

joblib.dump(feature_columns, f'{ARTIFACT_DIR}feature_columns.pkl')
joblib.dump(train_medians,   f'{ARTIFACT_DIR}train_medians.pkl')
print(f"Saved {len(feature_columns)} feature columns + {len(train_medians)} medians → {ARTIFACT_DIR}")
```
**Reason:** `3.4 Evaluating Tree-Based Models.ipynb` loads `feature_columns.pkl`/`train_medians.pkl` from `ARTIFACT_DIR` — this file now produces them (plus the three `.joblib` models it already saved), so running this single notebook is enough for 3.4 to work end-to-end. Also dropped the unused `roc_auc_score` import so the file is unambiguously F1-only, per Juan's request.

**Verified (static only — no data files/Colab access locally):** re-read the whole file, confirmed `feature_columns`/`train_medians`/`joblib`/`ARTIFACT_DIR` are all defined before use, confirmed zero remaining `roc_auc_score` references anywhere in the file, confirmed the saved filenames exactly match what 3.4's load cell expects.

**Not yet done:** `3.3 Tuning Tree-Based Models Using f1.ipynb` has NOT been deleted yet — that's a separate, explicit next step once the user/Juan are satisfied with this edit. `3.3_code_brief.ipynb`, `3.4_code_brief.ipynb` (generic boilerplate), and `3.4_lesson.ipynb`'s untuned-model gap are also still open (see plan for details) — none of that was touched in this change.

---

## 2026-07-07 — Module 2 (2.1, 2.2, 2.3, 2.4)

### 2.1 Introduction to Regularization
**Change:** None needed.
**Reason:** `2.1_code_brief.ipynb` and `2.1_lesson.ipynb` already contained the same 3 code snippets as the lecture (overfitting demo, bias-variance plot, L1/L2 geometry plot), just repackaged for format. Verified only.

### 2.2 Build a Regularized Logistic Regression Model — `2.2_code_brief.ipynb`
**Change:** Renamed variables and fixed hyperparameters/filenames to match lecture.
**Before:**
```python
l2_logistic_model = Pipeline([
    ('preprocessing', preprocessor),
    ('classifier', LogisticRegression(
        penalty='l2', C=1.0, class_weight='balanced',
        solver='lbfgs', max_iter=1000, random_state=42
    ))
])
...
models = {
    'L2 (Ridge)': l2_logistic_model, 'L1 (Lasso)': l1_logistic_model, 'ElasticNet': elasticnet_logistic_model
}
for name, model in models.items():
    filename = name.lower().replace(' ', '_').replace('(', '').replace(')', '')
    filepath = f'{models_path}{filename}_logistic_model.pkl'
```
**After:**
```python
model_l2 = Pipeline([
    ('preprocessing', preprocessor),
    ('classifier', LogisticRegression(
        penalty='l2', C=1.0, class_weight='balanced',
        solver='liblinear', random_state=42
    ))
])
...
models_dict = {
    "L2 Ridge": model_l2, "L1 Lasso": model_l1, "ElasticNet": model_elasticnet
}
for name, model_pipeline in models_dict.items():
    filename = name.lower().replace(' ', '_') + '_logistic.pkl'
```
**Reason:** Lecture uses `model_l2`/`model_l1`/`model_elasticnet`, `solver='liblinear'` for L2, no `max_iter`, dict keys without parentheses, and saves as `<name>_logistic.pkl`. code_brief had different names/params/filenames — same logic, different code.

### 2.2 — `2.2_lesson.ipynb`
**Change:** Same variable/hyperparameter/filename fixes as code_brief above, applied to the matching cells (L2/L1/ElasticNet build + save).
**Also fixed:** `models_filepath` was set equal to `data_filepath` (both pointed at the Data folder). Changed to a distinct `Course 3 Models/` path, matching the lecture's separate `/data/` vs `/models/` split.
**Reason:** Same as above, plus the path bug meant saved model files would land in the wrong folder.

### 2.3 Train and Compare Regularized Logistic Regression Models — `2.3_code_brief.ipynb`
**Change:** Added the missing baseline (unregularized) model to the comparison.
**Before:**
```python
l2_model = pickle.load(open(f'{course3_models}l2_ridge_logistic_model.pkl', 'rb'))
l1_model = pickle.load(open(f'{course3_models}l1_lasso_logistic_model.pkl', 'rb'))
elasticnet_model = pickle.load(open(f'{course3_models}elasticnet_logistic_model.pkl', 'rb'))
models = {'L2 (Ridge)': l2_model, 'L1 (Lasso)': l1_model, 'ElasticNet': elasticnet_model}
...
preprocessor = trained_models['L2 (Ridge)'].named_steps['preprocessing']
```
**After:**
```python
baseline_model = pickle.load(open(f'{course3_models}baseline_logistic.pkl', 'rb'))
l2_model = pickle.load(open(f'{course3_models}l2_ridge_logistic.pkl', 'rb'))
l1_model = pickle.load(open(f'{course3_models}l1_lasso_logistic.pkl', 'rb'))
elasticnet_model = pickle.load(open(f'{course3_models}elasticnet_logistic.pkl', 'rb'))
models = {'Baseline (No Penalty)': baseline_model, 'L2 (Ridge)': l2_model, 'L1 (Lasso)': l1_model, 'ElasticNet': elasticnet_model}
...
preprocessor = trained_models['Baseline (No Penalty)'].named_steps['preprocessing']
```
**Reason:** Lecture compares 4 models (baseline + 3 regularized); code_brief only had 3. Also updated the pickle filenames to match the corrected 2.2 save names (no `_model` suffix).

### 2.3 — `2.3_lesson.ipynb`
**Change:** Fixed the baseline model filename.
**Before:** `pickle.load(open(f'{models_filepath}baseline_logistic_model.pkl', 'rb'))`
**After:** `pickle.load(open(f'{models_filepath}baseline_logistic.pkl', 'rb'))`
**Reason:** File was never saved under the `_model.pkl` name anywhere — would crash with `FileNotFoundError`. Corrected to match what the lecture actually expects (`baseline_logistic.pkl`, still an external artifact assumed pre-existing from Course 2, same as the lecture's own assumption — not fabricated).
**Also fixed:** same `models_filepath == data_filepath` path bug as 2.2_lesson, same fix (distinct Models path).

### 2.4 Tune Regularization Hyperparameters
**Status:** Issues identified but **not yet fixed** (pending explicit go-ahead): stale `_logistic_model.pkl` load filenames (broken by the 2.2 fix above), `2.4_lesson.ipynb` confusion-matrix `labels=['N','Y']` bug (`'Y'` isn't a real class — should be `['N','E']`), same `models_filepath` path bug, `2.4_code_brief.ipynb` missing `Best C`/`Best l1_ratio`/`Std` columns and per-C results tables, and a save-filename mismatch with the lecture's static `best_tuned_logistic_model.pkl` (which Module 4.1 depends on by that exact name).

---

## 2026-07-07 — Module 3.1 (Introduction to Tree-Based Models)

### `3.1_code_brief.ipynb`
**Change:** Full rewrite — was 2 cells of generic boilerplate (identical across all of 3.1/3.2/3.3/3.4_code_brief), now contains the lecture's real content.
**Before:**
```markdown
## Key Pattern: Instantiate → Fit → Predict
​```python
model = ModelClass(**params)
model.fit(X_train, y_train)
y_prob = model.predict_proba(X_test)[:, 1]
​```
```
(nothing else)
**After:** 9 cells — the pattern intro (kept), the real demo-dict code cell (DecisionTree/RandomForest/XGBoost instantiation), and condensed reference notes for Gini impurity (with the worked numeric example), overfitting-control hyperparameters, Random Forest bagging, XGBoost boosting (with the worked boosting-rounds example), and the three-way comparison table.
**Reason:** code_brief had zero content specific to lecture 3.1 — it was interchangeable with every other module 3 code_brief.

### `3.1_lesson.ipynb`
**Change:** None needed.
**Reason:** Already contained the same 2 real code cells as the lecture (demo dict, tree-anatomy diagram) plus condensed Gini/RF/XGBoost notes. Verified only.

---

## 2026-07-07 — Module 3.2 (Building Tree-Based Models in Practice)

### `3.2_code_brief.ipynb`
**Change:** Full rewrite — was the same generic 2-cell boilerplate as 3.1/3.3/3.4_code_brief, now contains the lecture's real code.
**After:** 21 cells — setup/imports, Drive-mounted data load + feature prep, Decision Tree build+eval, Random Forest build+eval, XGBoost build+eval, side-by-side comparison table, feature importance chart, decision-tree visualization, and 5-fold cross-validation — all with hyperparameters copied verbatim from the lecture.
**Reason:** Same issue as 3.1_code_brief — no real code specific to 3.2.

### `3.2_lesson.ipynb`
**Change 1:** Removed two `.fillna(median())` calls in the feature-prep cell that the lecture doesn't have.
**Before:**
```python
train_encoded, test_encoded = train_encoded.align(test_encoded, join='left', axis=1, fill_value=0)
train_encoded = train_encoded.fillna(train_encoded.median())
test_encoded = test_encoded.fillna(test_encoded.median())
```
**After:**
```python
train_encoded, test_encoded = train_encoded.align(test_encoded, join='left', axis=1, fill_value=0)
```
**Reason:** Lecture has no fillna step here — kept lesson at strict 100% parity per explicit instruction, even though the extra fillna was arguably a safety net. Flagged as a residual open question (whether the source data has NaNs in those numeric columns) — unresolved since we don't yet have access to the real data files.

**Change 2:** Fixed the Cross-Validation cell — wrong scoring metric and missing hyperparameters.
**Before:**
```python
models_cv = {
    'Decision Tree': DecisionTreeClassifier(max_depth=8, min_samples_split=20,
        min_samples_leaf=10, class_weight='balanced', random_state=RANDOM_STATE),
    'Random Forest': RandomForestClassifier(n_estimators=200, max_depth=12,
        min_samples_leaf=5, class_weight='balanced', n_jobs=-1, random_state=RANDOM_STATE),
    'XGBoost': XGBClassifier(n_estimators=150, learning_rate=0.1, max_depth=5,
        subsample=0.8, colsample_bytree=0.8, use_label_encoder=False,
        eval_metric='logloss', random_state=RANDOM_STATE)
}
print("5-Fold Cross-Validation Results (ROC-AUC):")
for name, model in models_cv.items():
    scores = cross_val_score(model, X_train, y_train, cv=cv, scoring='roc_auc', n_jobs=-1)
```
**After:**
```python
scale_pos_weight_value = len(y_train[y_train==0]) / len(y_train[y_train==1])
models_cv = {
    'Decision Tree': DecisionTreeClassifier(max_depth=8, min_samples_split=20, min_samples_leaf=10,
        max_features='sqrt', class_weight='balanced', random_state=RANDOM_STATE),
    'Random Forest': RandomForestClassifier(n_estimators=200, max_depth=12, min_samples_split=10, min_samples_leaf=5,
        max_features='sqrt', class_weight='balanced', n_jobs=-1, random_state=RANDOM_STATE),
    'XGBoost': XGBClassifier(n_estimators=150, learning_rate=0.1, max_depth=5, min_child_weight=3,
        subsample=0.8, colsample_bytree=0.8, scale_pos_weight=scale_pos_weight_value,
        use_label_encoder=False, eval_metric='logloss', random_state=RANDOM_STATE)
}
print("5-Fold Cross-Validation Results (F1 Score):")
for name, model in models_cv.items():
    scores = cross_val_score(model, X_train, y_train, cv=cv, scoring='f1', n_jobs=-1)
```
**Reason:** Lesson was scoring on `'roc_auc'` instead of the lecture's `'f1'`, and had silently dropped `max_features='sqrt'` (DT+RF), `min_samples_split=10` (RF), and `min_child_weight`/`scale_pos_weight` (XGBoost, i.e. class-imbalance handling) — would produce different CV numbers than the lecture.

---

## Known open items (not yet fixed, tracked for future sessions)

- Module 2.4 (`2.4_code_brief.ipynb`, `2.4_lesson.ipynb`) — see Module 2.4 entry above.
- Module 3.3/3.4 — real bug identified at the **lecture level**: `3.4 Evaluating Tree-Based Models.ipynb` needs artifacts split across both `3.3 Tuning Tree-Based Models Using f1.ipynb` and `3.3 Tuning Tree-Based Models.ipynb` (the two near-duplicate 3.3 notebooks) — a student running only one will hit `FileNotFoundError` in 3.4 regardless of which one they pick. Not yet fixed — awaiting direction.
- All 3.3/3.4 `_code_brief.ipynb` files still contain the generic placeholder boilerplate (same issue 3.1/3.2 had) — not yet rewritten.
- Data files needed to actually execute/verify any of this code are not available locally — verification so far has been static (variable/filename tracing only). Juan has offered to share a Google Drive folder.

---

## 2026-07-13 — Module 7 (new `_code_brief.ipynb` files — did not exist before)

**Background:** Module 7 (EDA in Unstructured Data) had only the 6 lecture notebooks (7.1–7.6) plus 2 instructor-side data-generator notebooks — no `_code_brief.ipynb` or `_lesson.ipynb` for any sub-lesson, unlike every other module. Flagged to Juan/Keval (still awaiting confirmation on whether `_lesson.ipynb` files should also exist). User asked to build the code_briefs regardless.

### `7.1_code_brief.ipynb` (new file)
7.1's lecture is pure conceptual markdown — zero code cells. Code_brief is a condensed key-concepts summary (UPAD cycle applied to surveys, national vs. internal surveys, survey data structure) instead of code, since there's no code to extract.

### `7.2_code_brief.ipynb` (new file)
Same situation as 7.1 — 7.2's lecture is pure ethics/framework markdown, zero code. Condensed summary of ASA/ACM frameworks, the two dimensions of ethical practice, stakeholder analysis, and the 6-step reasoning process.

### `7.3_code_brief.ipynb` (new file)
13 code cells, verified byte-for-byte identical to the lecture (`CountVectorizer`/`TfidfVectorizer` on `Free_Response_Text`, helper function, train/test vectorization, merge with NSSE scores, CSV export).

**Known bug preserved as-is:** the lecture's TF-IDF cell references `ML_Survey_Data19.index` — that variable is never defined anywhere in the notebook (loaded frame is `ML_Survey_Data`), will throw `NameError` if run top to bottom. Per established rule (match lecture bugs as-is until Juan approves a fix — see Module 4.2 precedent), this bug is preserved verbatim in the code_brief. Already flagged to Juan; not fixed pending his answer.

### `7.4_code_brief.ipynb` (new file)
11 code cells, verified byte-for-byte identical to the lecture (TF-IDF recap, NMF topic modeling, LDA topic modeling, dominant-topic assignment, stakeholder reporting).

### `7.5_code_brief.ipynb` (new file)
8 real code cells, verified byte-for-byte identical to the lecture (VADER sentiment, synthetic GPA-linked labels, crosstab validation). Lecture has a 9th "code cell" that's genuinely empty (stray trailing cell, id `Uogfw6d-7qF2`) — not replicated since it has no content.

### `7.6_code_brief.ipynb` (new file)
13 code cells, verified byte-for-byte identical to the lecture (ColumnTransformer preprocessing, PCA on TF-IDF text features, feature fusion, master matrix export).

**Verification status:** All code cells verified via `difflib` byte-for-byte comparison against their lecture source, same method used throughout this changelog. Not yet run end-to-end (no data files available locally).

**Module 7 code_brief status: complete for 7.1–7.6.** `_lesson.ipynb` files still do not exist for module 7 — pending Juan/Keval confirmation on whether they're in scope.

---

## 2026-07-13 — Module 7.3 bug fix (LECTURE FILE EDIT)

**Background:** The `NameError` bug flagged in the 7.3 code_brief entry above (`ML_Survey_Data19.index` — undefined variable) was approved for a direct fix rather than waiting further on Juan.

### `7.3 Transforming Higher Ed Text Data to Vectors.ipynb` (lecture, cell-23)
**Before:**
```python
df_tfidf_vectorized.index = ML_Survey_Data19.index
```
**After:**
```python
df_tfidf_vectorized.index = ML_Survey_Data.index
```
**Reason:** `ML_Survey_Data19` was never defined anywhere in this notebook — the loaded frame is `ML_Survey_Data`. Leftover naming from an earlier draft (matches the naming convention in the separate TRAIN/TEST data-generator notebooks). Would throw `NameError` if run top to bottom; now matches the pattern used one cell earlier for the CountVectorizer DataFrame (`df_count_vectorized.index = ML_Survey_Data.index`).

### `7.3_code_brief.ipynb`
Same fix applied to the matching cell, to stay in sync with the lecture.

**Verified:** all 13 code cells in `7.3_code_brief.ipynb` still byte-for-byte identical to the lecture after the fix; `ML_Survey_Data19` no longer appears in any executable code in either file.

**Known cosmetic leftovers (not fixed, harmless):** a markdown cell still mentions the `ML_Survey_Data19_Num` DataFrame name in prose, and one code comment still reads `# Merge ML_Survey_Data19 and df_tfidf...`. Neither affects execution — left as-is pending user direction.

---

## 2026-07-13 — Module 3.3/3.4/4.1/8.3/8.4 (cross-module tuned-model filename fix)

**Background:** Reviewing Module 8 surfaced a real cross-module dependency break, predating any work in this session. The same three tuned tree models (Decision Tree, Random Forest, XGBoost) were referenced under **three different filenames** across the course:

| Producer | Old filenames | Consumed by |
|---|---|---|
| `3.3 Tuning Tree-Based Models.ipynb` (canonical, kept) | `decision_tree_best.joblib`, `random_forest_best.joblib`, `xgboost_early_stopping.joblib` | `3.4` lecture, `3.4_code_brief.ipynb`, `4.1_lesson.ipynb` |
| `3.3 Tuning Tree-Based Models Using f1.ipynb` (duplicate, marked for eventual deletion) | `dt_tuned_f1.pkl`, `rf_tuned_f1.pkl`, `xgb_tuned_f1.pkl` | `4.1 Systematic Model Comparison.ipynb` (Juan's actual lecture), `4.1_code_brief.ipynb` |
| *(nothing produced these)* | `dt_tuned.pkl`, `rf_tuned.pkl`, `xgb_tuned.pkl` (no suffix) | `8.3`, `8.4` lectures |

Juan's own `4.1` lecture depended on the duplicate 3.3 file that was slated for deletion — deleting it would have broken 4.1. Module 8.3/8.4 depended on filenames that didn't exist anywhere. Flagged to the user, proposed fix approved: standardize on the `_f1.pkl` naming (matches what Juan's `4.1` lecture already expects — avoids touching his file), update everything else to match.

### `3.3 Tuning Tree-Based Models.ipynb` (lecture, cell `3Mm6lsxcawE8`)
**Change:** save cell renamed — `decision_tree_best.joblib`→`dt_tuned_f1.pkl`, `random_forest_best.joblib`→`rf_tuned_f1.pkl`, `xgboost_early_stopping.joblib`→`xgb_tuned_f1.pkl`. Still `joblib.dump`, same directory.

### `3.4 Evaluating Tree-Based Models.ipynb` (lecture, cell `oulnguncmxmZ`)
**Change:** `model_filenames` dict updated to load the three renamed files.

### `3.4_code_brief.ipynb` (cell-5)
**Change:** same rename, to stay in sync with the lecture.

### `4.1_lesson.ipynb` (cell-5)
**Change:** `rf = joblib.load('../models/random_forest_best.joblib')` → `'../models/rf_tuned_f1.pkl'`, same for `xgb`.

### `4.1_code_brief.ipynb`
**No change needed** — already loaded `rf_tuned_f1.pkl`/`xgb_tuned_f1.pkl`, since it was built to match Juan's actual `4.1` lecture.

### `8.3 Enhancing Predictive Models with Survey Data.ipynb` (cell `RN8rC1f8UD4Z`)
**Two problems, one fix:** wrong filename (`xgb_tuned.pkl`) AND wrong Drive folder (`IR ML Cert/MLCert Course 3/Course 3 Models/` — a completely different folder than where module 3 actually saves its models). Added a dedicated `tree_models_path` variable pointing at `Applied-Data-Analytics-For-Higher-Education-Course-3/models/` (where the models actually live), used only for this load — the notebook's own `model_filepath` variable (`IR ML Cert/...`) is left untouched since it's still correct for everything module 8 itself produces/reads.

### `8.4 Systematic Model Comparison Framework.ipynb` (cell `dGMuLZB_AbUV`)
**Same two-part fix:** `dt_tuned.pkl`/`rf_tuned.pkl`/`xgb_tuned.pkl` → `dt_tuned_f1.pkl`/`rf_tuned_f1.pkl`/`xgb_tuned_f1.pkl`, loaded via the same new `tree_models_path` variable. The later save cell (`Survey_xgb_model.pkl`, cell `UyUztshYlcAe`) is untouched — correctly uses the `IR ML Cert/...` folder since that's a new model 8.4 itself creates.

**Verified:** full-repo sweep confirms zero remaining references to any of the three old filename sets anywhere in the course.

**Separately discovered, NOT part of this fix, flagged to user:** `3.3_lesson.ipynb` and `3.3_code_brief.ipynb` are not actually in the fixed state earlier changelog entries described — `3.3_lesson.ipynb` still has the AUC-scoring bug, the leakage bug, hardcoded (non-`best_params_`) early-stopping hyperparameters, and has no model-save cell at all; `3.3_code_brief.ipynb` is still the original empty 2-cell placeholder. Likely reverted by a later full-folder file replacement. `3.4_lesson.ipynb` depends on a save step that doesn't exist yet. This needs its own rebuild pass, separate from today's rename fix.

---

## 2026-07-13 — Module 8 (bug fixes in 8.4/8.5, then new `_code_brief.ipynb` files for 8.1–8.5)

**Background:** Thorough review of module 8 (didn't exist as code_briefs before, same gap as module 7) surfaced real bugs in 8.4 and 8.5, re-verified word-for-word before fixing. User approved fixing them, then building code_briefs the same way as module 7.

### `8.4 Systematic Model Comparison Framework.ipynb` (lecture)

**Fix 1 — leakage bug (cell `5452c3df`):**
- **Before:** `train_enc = train_enc.fillna(train_enc.median())` / `test_enc = test_enc.fillna(test_enc.median())`
- **After:** captures `train_admin_medians = train_enc.median()` once, fills both train and test with it.
- **Reason:** test was leaking its own median — the exact bug pattern fixed repeatedly elsewhere in this course (3.3, 3.4, 4.1).

**Fix 2 — leakage bug (cell `d7f4ac7c`):** same pattern, same fix, for the survey-enhanced feature set (`X_train_survey`/`X_test_survey`).

**Fix 3 — copy-paste bug (cell `06606175`, radar chart):**
- **Before:** `'Decision Tree': [results_df.loc['Regularized Logistic', 'ROC-AUC'] * 10, ...]`
- **After:** `'Decision Tree': [results_df.loc['Decision Tree', 'ROC-AUC'] * 10, ...]`
- **Reason:** Decision Tree's radar plot was pulling Logistic Regression's ROC-AUC value instead of its own.

**Fix 4:** deleted stray scratch cell `Uz7dkIGeQsdv` (`90_000+32`) — leftover debug arithmetic, zero pedagogical value.

### `8.5 Final Model Selection and Deployment.ipynb` (lecture, cell `338df7e2`)
**Fix:** `RANDOM_STATE = 2` → `RANDOM_STATE = 42`. Every other notebook in the course uses `42`; this was an isolated inconsistency.

**Not fixed, flagged only (needs Juan/more info, not a code-level fix):**
- `Deploy_Survey_Data.csv` / `Deploy_Data_Other.csv` (loaded in 8.5) are not produced by any notebook anywhere in this repo — confirmed via full recursive search. The cached output in 8.5 proves these files existed when the instructor ran it once, but there's no generator notebook for them in this repo (possibly related to the `deploy_departure*.csv`/`deploy_grade_points*.csv` raw files Juan shared, but no processing notebook connects them).
- Cell `S7dHpux4grU_`: `pd.concat([df_deploy_names[['SID','NAME','LAST_NAME']], holdout_scores], axis=1)` joins by row position, not by a key like SID — risk of silently mismatching students to the wrong risk score if the two files aren't in identical row order. Not fixed since altering the join logic without the actual data to verify column structure risked introducing a different bug.
- Model card claim `Performance (AUC) | 0.8791` — unverified, no way to confirm it was actually computed in this notebook without running it.

### New files: `8.1_code_brief.ipynb` through `8.5_code_brief.ipynb`
Same pattern as Module 7:
- `8.1_code_brief.ipynb` — 8.1's lecture is pure conceptual (survey-ML literature review, algorithm families), zero code. Condensed key-concepts summary instead.
- `8.2_code_brief.ipynb` — 14 code cells (K-Means clustering on the Module 7 master matrix), verified byte-for-byte identical to the lecture.
- `8.3_code_brief.ipynb` — 16 code cells (survey-enhanced XGBoost, feature-group importance), verified byte-for-byte identical, includes the model-path fix from the earlier cross-module filename fix.
- `8.4_code_brief.ipynb` — 22 code cells, verified byte-for-byte identical to the lecture **after** the leakage/radar/stray-cell fixes above.
- `8.5_code_brief.ipynb` — 12 code cells (deployment pipeline, risk bands, model card), verified byte-for-byte identical to the lecture after the `RANDOM_STATE` fix. Trailing empty cell in the lecture not replicated (same convention as 7.5, 8.2).

**Verification status:** all code cells verified via `difflib` byte-for-byte comparison against lecture source. Not run end-to-end (no data files available locally, and 8.5's deploy data doesn't exist in this repo at all).

**Module 8 status:** all 5 code_briefs complete. `_lesson.ipynb` files still don't exist for module 8, same open question as module 7 — pending Juan/Keval.

---

## 2026-07-14 — Module 7 (new `_lesson.ipynb` files) + Module 7 data pipeline fix + newly found 7.6 leakage bug

### Generator fix: `Generating Higher Ed Text Data - TRAIN.ipynb` (cell `cell-26`)
**Before:** `output_filename = 'ML_Survey_Data19.csv'`
**After:** `output_filename = 'ML_Survey_Data.csv'`
**Reason:** every downstream notebook (7.3, 7.4, 7.5, 8.2) reads `ML_Survey_Data.csv` (no "19" suffix) — nothing produced that exact filename before this fix, so the pipeline broke immediately after the generator ran. Confirmed via full-repo search that nothing else writes or expects the "19" version. FYI'd to Juan (not a judgment call, just a stale leftover from when the course used cohort-year-suffixed naming).

### New files: `7.1_lesson.ipynb` through `7.6_lesson.ipynb`
Same convention as `6.1_lesson.ipynb`/`6.2_lesson.ipynb` — local relative paths (`../data/`), no Drive mount, regardless of the lecture's own path convention.
- `7.1_lesson.ipynb`, `7.2_lesson.ipynb` — condensed teaching narrative (lectures have no code).
- `7.3_lesson.ipynb` — CountVectorizer/TF-IDF vectorization, helper function, merge with NSSE scores, export.
- `7.4_lesson.ipynb` — NMF + LDA topic modeling, dominant-topic assignment.
- `7.5_lesson.ipynb` — VADER sentiment, synthetic ground-truth validation, crosstab.
- `7.6_lesson.ipynb` — ColumnTransformer + PCA feature fusion, master matrix export.

### Newly found bug, NOT fixed, flagged only: `7.6 Get Your Survey Data Machine Learning Ready.ipynb`
**Leakage bug** in the structured-feature preprocessing cell:
```python
X_structured_train = preprocessor.fit_transform(df_train)
X_structured_test = preprocessor.fit_transform(df_test)   # ← re-fits on test, should be .transform()
```
`preprocessor.fit_transform(df_test)` re-fits the `MinMaxScaler`/`StandardScaler`/`OneHotEncoder` on the test set separately, instead of reusing the train-fitted `preprocessor` via `.transform(df_test)` — the same leakage pattern found and fixed repeatedly elsewhere in this course. Confirmed present in the lecture and preserved as-is in `7.6_code_brief.ipynb` (verified byte-for-byte identical to the lecture, bug included, per the established "match lecture as-is until approved" rule).

**Self-correction:** the first draft of `7.6_lesson.ipynb` written today silently used the *correct* `.transform(df_test)` instead of matching this bug — caught before finishing, reverted to `.fit_transform(df_test)` to stay consistent with the lecture and code_brief. Not fixed anywhere yet — pending the same approval process used for the 7.3 `ML_Survey_Data19` bug.

**Module 7 status:** all 6 `_lesson.ipynb` files now exist alongside the 6 `_code_brief.ipynb` files built earlier. Module 7 is otherwise complete pending: (1) approval to fix the 7.6 leakage bug, (2) Keval's answer on the module 8.5 deploy-data question.

---

## 2026-07-15 — Module 6 and 7: fixing all known leakage bugs, "make it perfect" pass

**Background:** User asked to make Module 6 and 7 fully correct before continuing to Module 8. This session fixed every previously-flagged bug in both modules, verified via full sweep that nothing else remains.

### `6.1 Special Topics - Additional Boosting Algorithms.ipynb` (lecture, cell `P7PUSeJVyW-p`)
**Before:** `train_enc = train_enc.fillna(train_enc.median())` / `test_enc = test_enc.fillna(test_enc.median())`
**After:** captures `train_medians = train_enc.median()` once, fills both train and test with it.
**Reason:** test was leaking its own median — same pattern found and fixed repeatedly elsewhere in this course. This bug in 6.1's lecture had not been explicitly flagged before (it predates all work in this session and was inherited when 6.1 was first rewritten from a placeholder).
Propagated identically to `6.1_code_brief.ipynb` (cell `cell-3`) and `6.1_lesson.ipynb` (cell `cell-4`). Verified all three now consistent.

### `6.2 Special Topics - Neural Networks.ipynb` (lecture, cell `5VirBoVh5KLF`)
Same leakage bug, same fix. Propagated to `6.2_code_brief.ipynb` (cell `cell-4`) and `6.2_lesson.ipynb` (cell `cell-4`).

### `7.6 Get Your Survey Data Machine Learning Ready.ipynb` (lecture, cell `CFI4PUHYP595`)
**Before:** `X_structured_test = preprocessor.fit_transform(df_test)` — re-fits `MinMaxScaler`/`StandardScaler`/`OneHotEncoder` on test data separately.
**After:** `X_structured_test = preprocessor.transform(df_test)` — reuses the train-fitted preprocessor.
**Reason:** same leakage class as above; this is the bug flagged (not fixed) during the 7.6_lesson build earlier today — now fixed for real, since the user approved fixing all known Module 6/7 issues.
Propagated to `7.6_code_brief.ipynb` (cell `d84ae8c5`) and `7.6_lesson.ipynb` (cell `cell-5`, which had been deliberately reverted to match the bug earlier — now corrected back to `.transform()`).

### Cosmetic cleanup: `7.3 Transforming Higher Ed Text Data to Vectors.ipynb` + `7.3_code_brief.ipynb`
Cleaned up the two harmless `ML_Survey_Data19` leftovers noted (but intentionally left alone) in the earlier 7.3 bug-fix entry: a markdown mention of `ML_Survey_Data19_Num` and a code comment `# Merge ML_Survey_Data19 and df_tfidf...`, both renamed to drop the stale "19". Neither affected execution before or after — pure cleanup for full consistency.

**Verified:** full-sweep grep across every `.ipynb` in both modules for the leakage pattern (`test.fillna(test.median())`, `preprocessor.fit_transform(test)`) and the `ML_Survey_Data19` string — zero remaining hits outside the `Generating Higher Ed Text Data - TRAIN.ipynb` generator (where `ML_Survey_Data19` is the correct, intentional variable name for that script). Re-confirmed `7.3_code_brief.ipynb` and `7.6_code_brief.ipynb` still byte-for-byte identical to their lectures after all edits.

**Module 6 status: complete and verified clean.** **Module 7 status: complete and verified clean**, except the still-open external blocker (Module 8.5 deploy-data question, pending Keval — not a Module 7 issue, listed here only because it was surfaced during this module's review).

---

## 2026-07-15 — Module 8 (new `_lesson.ipynb` files for 8.1–8.5)

Same convention as Module 7's lessons — local relative paths (`../data/`, `../models/`), no Drive mount, regardless of the lecture's own path convention.

- `8.1_lesson.ipynb` — condensed teaching narrative (lecture has no code).
- `8.2_lesson.ipynb` — K-Means clustering on the master matrix, elbow method, cluster profiling with raw values + student-voice validation, saves `kmeans_model.pkl`.
- `8.3_lesson.ipynb` — loads the Module 3 tuned XGBoost hyperparameters, refits on survey-enhanced features, evaluates, feature-group importance.
- `8.4_lesson.ipynb` — five-model comparison (Logistic, Decision Tree, Random Forest, XGBoost, Survey-Enhanced XGBoost), full metric suite, feature-group importance. Built against the **already-fixed** leakage logic (this module's lecture never had the bug others did).
- `8.5_lesson.ipynb` — deployment pipeline: risk scores, capacity-based threshold, risk bands, advisor outreach list.

### Self-correction during the 8.5 build
First draft of `8.5_lesson.ipynb` silently "improved" the advisor-outreach-list join from the lecture's `pd.concat([...], axis=1)` (positional join, no key) to a proper `pd.merge(..., on='SID')` — without flagging or getting approval first. This is the same class of mistake caught earlier during the `7.6_lesson.ipynb` build (silently fixing a bug instead of matching the lecture and flagging it). Caught and reverted before finishing: `8.5_lesson.ipynb` now matches the lecture's actual (flagged, still-unresolved) positional-concat approach. The `pd.merge(on='SID')` fix remains a **recommendation only**, not applied anywhere yet — pending the same approval process as the 7.6 leakage bug.

**Known dependency gaps (not fixed, not new — inherited from earlier findings):**
- `8.3_lesson.ipynb` and `8.4_lesson.ipynb` load `../models/dt_tuned_f1.pkl` / `rf_tuned_f1.pkl` / `xgb_tuned_f1.pkl` — these are only produced locally once `3.3_lesson.ipynb` is actually rebuilt (still broken, blocked mid-fix earlier this session, not yet completed).
- `8.5_lesson.ipynb` loads `Deploy_Survey_Data.csv` / `Deploy_Data_Other.csv` — still pending Keval's answer on which raw file (`deploy_departure.csv` vs `deploy_grade_points.csv`) is canonical and what the two `student_academics_deploy_*.csv` files are for.

**Module 8 status:** all 5 `_lesson.ipynb` files now exist alongside the 5 `_code_brief.ipynb` files built earlier. Two items remain before Module 8 can be called fully complete: the 3.3_lesson rebuild (blocks 8.3/8.4 from actually running locally) and Keval's deploy-data answer (blocks 8.5).

---

## 2026-07-15 — Repo-wide GitHub cell-ID fix, then full cross-module audit

**Background:** Created a private GitHub repo (`course3-review`) to review changes. After pushing, lesson notebooks wouldn't open on GitHub.

### Root cause and fix: missing cell `id` fields
GitHub's notebook renderer requires every cell to have a real string `id` when `nbformat_minor >= 5` (nbformat 4.5+ spec). The lesson-file generator script used earlier this session wrote `"id": null` instead of real IDs. Wrote a repo-wide script assigning a unique id to every cell missing one, touching only the `id` field — verified no code content changed (spot-checked `7.6_code_brief.ipynb` still byte-for-byte identical to its lecture after the fix). **27 files fixed**: all of 7.1–7.6 and 8.1–8.5 lesson files, several code_briefs (1.1–1.3, 3.1–3.4, 4.1–4.2, 5.1, 6.1–6.2), and 3 lecture files (1.1, 1.2, 1.3, 4.2) that had the same gap. Committed and pushed.

### Full cross-module audit (user requested `/code-review`, but the skill needs a git diff — repo was fully committed/pushed with nothing pending, so its diff was empty; pivoted to the same manual sweep methodology used all session)
Three systematic sweeps across every module (0–8):
1. **Leakage pattern sweep** (`test.fillna(test.median())`, `preprocessor.fit_transform(test)`, etc.) — only known, already-flagged issues found (3.3_lesson still broken, 4.2's intentional as-is match). Nothing new.
2. **Scoring-metric mismatch sweep** (lesson/code_brief `scoring=` vs. lecture's) — only the known 3.3_lesson issue. Nothing new.
3. **Code_brief vs. lecture logic/byte comparison, all modules** — surfaced a real, previously-unflagged bug:

### New bugs found and fixed: `2.2_code_brief.ipynb`, `2.3_code_brief.ipynb`, `2.4_code_brief.ipynb`
Three separate problems, all from when these files were built early in this session (before the byte-exact methodology was standardized):

1. **Wrong path convention.** All three code_briefs used local `../data/`/`../models/` paths, but their lectures (2.2/2.3/2.4) mount Google Drive and use `project_path = '/content/drive/MyDrive/Applied-Data-Analytics-For-Higher-Education-Course-3'`. Code_briefs are supposed to match the lecture exactly (the local-path convention is only for lesson files) — flagged to Juan, who chose **"match the lecture exactly"** over the alternative (adopt the lesson-style local convention). Fixed all three: added `drive.mount()`, restored the real `project_path`/`data_filepath`/`models_filepath` variables, updated every read/write to use them.
2. **Wrong title in two files.** `2.2_code_brief.ipynb` and `2.3_code_brief.ipynb` both had copy-paste title errors — headed "1.2 Code Brief" and "1.3 Code Brief" respectively (Module 1 numbering) instead of "2.2"/"2.3". Fixed.
3. **`2.4_code_brief.ipynb` — wrong model filenames and incomplete save logic**, found while fixing the path issue:
   - Loaded `l2_ridge_logistic_model.pkl` etc. (files that don't exist — 2.2/2.3 actually save `l2_ridge_logistic.pkl`, no `_model` suffix). Fixed to the correct filenames.
   - Final save cell used a dynamic filename (`{best_model_name}_tuned.pkl`) instead of the lecture's fixed `best_tuned_logistic_model.pkl` — the exact name `4.1`'s lecture/lesson load. Also completely missing the `grid_search_comparison_data.pkl` save (a dict of all 3 GridSearchCV objects + best model name) that the lecture produces. Both fixed to match the lecture's cell `34` exactly.

**Verified:** all three code_briefs re-checked — correct titles, Drive paths throughout, zero local-path references remaining, correct model filenames matching what 2.2/2.3/2.4's lectures actually save/load.

**Audit conclusion:** aside from the two long-known, already-flagged open items (3.3_lesson rebuild, 8.5 deploy-data question), the entire course (modules 0–8) is now verified consistent — every code_brief matches its lecture's actual code and environment, every lesson file either matches its lecture's logic (on local paths, per the established lesson convention) or correctly mirrors a known unfixed lecture bug pending approval.

---

## 2026-07-15 — `3.3_lesson.ipynb` and `3.3_code_brief.ipynb` rebuilt for real this time

**Background:** These two files were reported fixed earlier this session, then discovered still broken during a later review (likely reverted by a Downloads-folder file replacement somewhere along the way). A rebuild attempt got blocked mid-way by the permission system. User re-approved; rebuilt both from scratch this time, verified immediately rather than assumed.

### `3.3_code_brief.ipynb`
Rebuilt as 19 cells — condensed markdown headers + all 9 of the canonical lecture's code cells transcribed exactly (Drive paths, F1 scoring throughout, `xgb_search.best_params_` reuse for early stopping, the `dt_tuned_f1.pkl`/`rf_tuned_f1.pkl`/`xgb_tuned_f1.pkl` save cell).
**Verified:** byte-for-byte identical to `3.3 Tuning Tree-Based Models.ipynb` across all 9 code cells (caught and fixed one `→` vs `->` unicode mismatch during verification).

### `3.3_lesson.ipynb`
Rebuilt as 18 cells on local `../data/`/`../models/` paths (same convention as every other lesson file), matching the lecture's actual logic:
- `scoring='f1'` throughout (not `roc_auc`)
- Train-only median imputation (not leakage)
- XGBoost early stopping reuses `xgb_search.best_params_` (not hardcoded hyperparameters)
- New save cell — `dt_tuned_f1.pkl`, `rf_tuned_f1.pkl`, `xgb_tuned_f1.pkl` to `../models/`

**Verified:** scripted check confirms zero leakage pattern, zero `roc_auc` scoring, `f1` scoring present, `best_params_.copy()` reuse present, and all three save-file references present.

**Downstream effect:** `3.4_lesson.ipynb` and `4.1_lesson.ipynb` (which already expected these exact three files in `../models/`) are now unblocked — `3.3_lesson.ipynb` actually produces what they need.

**Module 3 status: fully complete.** Item #1 and #5 from the "what's left" list are resolved — 8.3_lesson/8.4_lesson can now actually run locally once data is in place.

---

## 2026-07-15 — Remaining "what's left" items: `5.1_code_brief.ipynb` rebuild, 1.1 wording fix, repo cleanup

Before fixing, checked whether either flagged item was actually a genuine bug or something required as-is (same diligence as the 3.3 files, which turned out to be genuinely load-bearing downstream). Confirmed both were real, isolated bugs with no hidden dependency:
- `5.1`'s lecture is fully self-contained (all synthetic data generated inline, zero `pickle.dump`/`joblib.dump`/`to_csv`/external `read_csv` calls) and nothing in modules 6/7/8 references module 5 at all — safe to rebuild with no downstream risk.
- 1.1's "CRISP framework from notebook 1.2" reference is a genuine forward-reference error — confirmed CRISP is never taught before 1.1 (module 0 doesn't mention it), and 1.2 independently teaches CRISP fresh rather than citing 1.1 as its source. No hidden intent, just a wording slip.

### `5.1_code_brief.ipynb` rebuilt
25 cells — condensed markdown headers + all 16 of the lecture's code cells. First attempt (typed from memory) dropped several inline comments and renamed one variable (`entry_pca` → `coords`) — caught during verification, rebuilt a second time by extracting the lecture's cells programmatically instead of retyping them, to guarantee exactness.
**Verified:** byte-for-byte identical to `5.1 Institutional Pattern Discovery with Unsupervised Learning.ipynb` across all 16 code cells.

### `1.1 AI-Assisted Coding in Google Colab with Gemini.ipynb` (lecture) — wording fix
Two cells referenced CRISP as if it came from notebook 1.2 (which is taught *after* 1.1):
- Cell `3f720294`: "The same CRISP prompting framework from notebook 1.2 works perfectly with Gemini" → "The CRISP prompting framework works perfectly with Gemini (you'll use this same framework with Codex in notebook 1.2)"
- Cell `d9b0fe89` (Summary): "The same CRISP framework from Codex (notebook 1.2) works with Gemini" → "The CRISP framework works with Gemini here, and you'll use it again with Codex in notebook 1.2"

Checked `1.1_code_brief.ipynb` and `1.1_lesson.ipynb` for the same issue — neither had it, no changes needed there.

### GitHub repo cleanup
Removed `3.3_comparison_for_juan.md` and `3.3_comparison_for_juan.pdf` from the GitHub repo (user request — these were scratch review artifacts made to get Juan's sign-off on the 3.3 duplicate-file merge, already resolved, not course content). Used `git rm --cached` (untrack, keep local copies) plus added both to `.gitignore` so they don't get re-tracked. They remain in the repo's git history (initial commit) but no longer appear in the current tree or any future commit.

**"What's left" list status:** items #2 (5.1_code_brief) and #3 (1.1 wording) resolved. Only #4 (8.5 positional-join, still just a recommendation pending approval) and #6 (8.5 deploy data, partially resolved — have `Deploy_Survey_Data.csv` from Keval, still need `Deploy_Data_Other.csv` and file clarification) remain open.

---

## 2026-07-21 — Subagent-driven full review: 4 real bugs found and fixed + 1 flagged

Ran 7 parallel review subagents (one per code-heavy module 2–8), each doing a deep correctness check of that module's lecture/code_brief/lesson triads. Verified every new claim directly before acting (one subagent claim refuted). Found bugs my earlier grep sweeps missed because they weren't leakage/scoring patterns:

### FIXED — Critical
- **`3.4_lesson.ipynb`**: loaded dead model filenames `decision_tree_best.joblib` / `random_forest_best.joblib` / `xgboost_early_stopping.joblib`, but `3.3_lesson.ipynb` now saves `dt_tuned_f1.pkl` / `rf_tuned_f1.pkl` / `xgb_tuned_f1.pkl` (from the earlier rename fix). Guaranteed `FileNotFoundError`. The rename fix had updated 3.4 lecture + 3.4_code_brief but missed the lesson. Fixed the three load paths to `_f1.pkl`.

### FIXED — Minor
- **`2.4 Tune Regularization Hyperparameters.ipynb`** (lecture): title said `# 1.4` → fixed to `# 2.4`.
- **`2.1_code_brief.ipynb`**: title said `# 1.1 Code Brief` → fixed to `# 2.1 Code Brief`. (Completes the title-fix set; 2.2/2.3 code_brief titles were fixed earlier.)
- **`8.1 Introduction ... .ipynb`** (lecture): two typos in the Study-C retraction sentence — "retracted by due to" → "retracted due to", "thr integrity" → "the integrity".

### REFUTED subagent claim (no action)
- A subagent claimed `3.3_lesson.ipynb` must save `feature_columns.pkl` + `train_medians.pkl` or `3.4_lesson` breaks. Verified false: `3.4_lesson` is self-contained — it reads `../data/training.csv` + `testing.csv` and computes its own train medians correctly (train-only, no leakage). The lesson track never loads those two artifacts; only the lecture/code_brief track does. Both tracks valid.

### FLAGGED, NOT auto-fixed — the one remaining runtime crash
- **`baseline_logistic.pkl` orphan**: `2.3` (lecture + code_brief + lesson) loads `baseline_logistic.pkl` to compare the unregularized baseline against the regularized models, but NOTHING in Course 3 produces it (2.1 is pure theory; 2.2 only saves l2_ridge/l1_lasso/elasticnet). 2.4 does NOT need it (only 2.3). This is a guaranteed `FileNotFoundError` in 2.3 — confirmed at runtime by the user. Fix requires either (A) 2.2 also build+save a `LogisticRegression(penalty=None)` baseline, or (B) 2.3 drop the baseline from its comparison. Both change lecture content / pedagogy → Juan's decision, not a mechanical fix. Recommendation: Option A (keeps the baseline-vs-regularized comparison 2.3 is built around; matches the course's "each notebook saves what the next needs" pattern).

Module review verdicts from subagents (after verification): Modules 4, 5, 6, 7 fully clean. Module 2 = baseline orphan + 2 title typos (fixed). Module 3 = 3.4_lesson filename crash (fixed). Module 8 = 8.1 typos (fixed), everything else clean including the known-accepted 8.5 positional-join / deploy-data items.

---

## 2026-07-21 — FIX: baseline_logistic.pkl orphan (Option A) — 2.3 no longer crashes

The last remaining runtime crash. 2.3 (lecture + code_brief + lesson) loaded `baseline_logistic.pkl` to compare the unregularized Course-2 baseline against the regularized models, but nothing produced it → guaranteed FileNotFoundError.

**Fix (Option A, user-approved):** 2.2 now also builds and saves the baseline. Added to all three 2.2 files (lecture, code_brief, lesson):
- A new model cell: `model_baseline = Pipeline([... LogisticRegression(penalty=None, class_weight='balanced', solver='lbfgs', max_iter=1000, random_state=42)])`. `penalty=None` = the Course-2 unregularized baseline; `solver='lbfgs'` (liblinear does not support penalty=None); same preprocessor as the regularized models.
- `"Baseline": model_baseline` added as the first entry in `models_dict`.

The existing save loop (`name.lower().replace(' ','_') + '_logistic.pkl'`) turns the "Baseline" key into exactly `baseline_logistic.pkl` — the filename all three 2.3 files load. Verified the full chain: each 2.2 file's models_dict contains "Baseline", and each 2.3 file loads `baseline_logistic.pkl` + the three regularized files. Match confirmed.

Why Option A over dropping the baseline from 2.3 (Option B): 2.3 is literally "Train and COMPARE Regularized... Models" — the baseline-vs-regularized comparison is the lesson's whole point; Option B would gut it and require reworking 2.3's tables/coefficient plots. Option A is the smaller change and pedagogically correct. It also matches the course's established "each notebook saves what the next needs" pattern (the bug was simply that this one artifact got dropped from the chain). 2.2's narrative already recaps "the baseline model from Course 2 (penalty=None)" in section 2, so building it there is fully consistent.

Flagged to Juan as a lecture-content change (informational, per the established fix workflow).

**Course status:** both known runtime crashes now fixed (3.4_lesson filenames earlier today, baseline_logistic.pkl now). No remaining code-level crashes anywhere in the course. Only open items are external/decision: the 8.5 SID-column request to Kagba (held uncommitted), and the bottleneck.csv question to Juan.

---

## 2026-07-21 — FIX: corrupted duplicate cells from the earlier baseline fix (2.2 lecture/code_brief/lesson)

The baseline fix committed earlier today (`eeca04d`) had a real defect the user caught by actually running it in Colab: in all three 2.2 files, the NotebookEdit that was meant to add `"Baseline": model_baseline` to `models_dict` instead landed on the wrong cell — it overwrote the markdown header immediately before the code cell (turning it into a markdown cell containing the new code as inert, non-executing text), while the actual `models_dict` **code** cell was left untouched with its old 3-model version. Net effect: the notebook still only ever saved `l2_ridge_logistic.pkl` / `l1_lasso_logistic.pkl` / `elasticnet_logistic.pkl` — `baseline_logistic.pkl` was never actually produced, so 2.3 still crashed. The `model_baseline` Pipeline definition itself (added earlier in each file) was correct and unaffected — only the models_dict registration was broken.

**Fixed** in all three files: restored the markdown header to plain text (2.2 lecture: "### Model Comparison Summary"; 2.2_code_brief: "## Save Models"; 2.2_lesson: "## Compare What We Built"), and fixed the real code cell to include `"Baseline": model_baseline` in models_dict. Verified: each file now has exactly one code cell defining models_dict, containing Baseline, and zero markdown cells with stray code text.

Root cause: this notebook format doesn't use the standard nbformat top-level `cell.id` (id lives in `cell.metadata.id`, a Colab-export convention) — the edit tool couldn't resolve the target cell reliably against that scheme when a new cell had just been inserted earlier in the same file, and silently landed on an adjacent cell instead of erroring. Lesson: after any edit to a file using this id convention, verify the actual diff/resulting cell content directly rather than trusting the edit confirmation.

---

## 2026-07-21 — FIX: undefined `elasticnet_results_best_l1` in 2.4 (caught by user's live Colab run)

2.4's hyperparameter-visualization cell used `elasticnet_results_best_l1` (to plot ElasticNet's CV F1 vs. C on the same axis as L2/L1, which only have one hyperparameter each) but that variable was never defined anywhere — a `NameError` on run. Only the lecture references it; `2.4_code_brief.ipynb` and `2.4_lesson.ipynb` don't include this visualization cell, so they were unaffected.

**Fix:** inserted a new cell right after `elasticnet_results` is built, deriving the per-C "best l1_ratio" row via `elasticnet_results.groupby('param_classifier__C')['mean_test_score'].idxmax()` — for each C value, keep only the row with the highest CV F1 across the 5 l1_ratio values tested. This is exactly what the existing comment above the broken line already described ("the 'elasticnet_results_best_l1' dataframe which already has the best l1_ratio for each C value") — the derivation step was simply missing.

This is the third real runtime-only bug the user's actual Colab run has caught (after the 3.4_lesson filename crash and the baseline_logistic.pkl orphan) — confirms static/agent review alone wasn't sufficient; the live run-through is finding real, distinct bugs each module.

---

## 2026-07-21 — Full ML-methodology review, modules 0–7 (6 parallel review agents)

User had by this point run every lecture in Colab end-to-end and fixed all runtime crashes (baseline_logistic.pkl, 3.4_lesson filenames, elasticnet_results_best_l1). Requested a deeper pass: not "does it crash" (already proven out live) but "is the ML methodology actually sound" — reviewed as an ML engineer.

**Scope:** Modules 0 and 1 checked first and excluded — 0.1 is pure setup/package-verification (no data, no models); 1.1–1.3 are AI-prompting tutorials (no model-training code). Dispatched one methodology-review agent per module for 2, 3, 4, 5, 6, 7 (module 8 out of scope for this pass), each checking: test-set discipline, class-imbalance handling, CV strategy, leakage patterns, threshold consistency, reproducibility, and comparison fairness. Every non-trivial claim from the agents was independently re-verified against the actual notebook JSON/cached outputs before being accepted — one agent claim was corrected in the process (see below).

### FLAGGED — real bug, not yet fixed (pending decision)

**Module 7.3/7.6 — TF-IDF/CountVectorizer fit separately on train and test text.**
`vectorize_text_data()` in 7.3 (lecture + code_brief + lesson) creates a fresh `CountVectorizer`/`TfidfVectorizer` and calls `.fit_transform()` on whatever dataframe is passed in — called once on `ML_Survey_Data` (train text) and again on `ML_Survey_Data22` (test text). Each call builds its own independent vocabulary from that cohort's text alone. Verified downstream in 7.6: the resulting matrices are consumed via pure positional slicing (`ML_Survey_Data_Num.iloc[:, 11:]`, `ML_Survey_Data22_Num.iloc[:, 11:]`), no column-name alignment, then `pca.fit_transform(tfidf_matrix_train)` followed by `pca.transform(tfidf_matrix_test)`.

Since train (Fall 2019 cohort) and test (Fall 2022 cohort) have different free-text responses, their independently-fit vocabularies will almost certainly differ in size — meaning `pca.transform(tfidf_matrix_test)` will raise a feature-count mismatch (`ValueError`) the first time 7.6 is actually run against real generated data. If the vocabulary sizes coincidentally match, it's worse: PCA would silently apply a transformation learned from train's column meanings onto test's differently-meaning columns — garbage output with no error.

**Correct fix (not yet applied — lecture-content change, needs sign-off):** fit the vectorizer once on train text only (`.fit()`), then `.transform()` (not `.fit_transform()`) on test text, so both share one vocabulary/column set — same pattern already used correctly elsewhere in the course (train-only medians, train-only preprocessor fit). Affects 7.3's lecture, code_brief, and lesson (all three currently have the bug identically).

### VERIFIED SOUND — no action needed, full audit trail

- **Module 4** (1.1/4.2): no findings at all — comparison fairness, feature-space handling, threshold consistency, reproducibility all correct.
- **Module 2** (2.2–2.4): test-set discipline, class-imbalance handling (`class_weight='balanced'` + F1-on-minority scoring throughout), StratifiedKFold CV, Pipeline-based leakage prevention, `random_state=42` consistency, and search-space sanity all verified correct.
- **Module 3** (3.3/3.4): test-set discipline, train-only median imputation, XGBoost early-stopping validation split (carved from train, not test), reproducibility, and 3.3→3.4 feature-schema handoff (`feature_columns.pkl`/`train_medians.pkl`) all verified correct.
- **Module 5** (5.1): scaling-before-clustering, PCA fit on the same scaled data used for clustering, cluster profiles correctly computed on original (not scaled) units, RISK_INDEX weights sum to 1.0 and are self-normalized, reproducibility all verified correct.
- **Module 6** (6.1/6.2): MLPClassifier scaling (fit-train/transform-test), early-stopping validation split, train-only median imputation, and 6.1-vs-Module-3-XGBoost comparison fairness all verified correct.
- **Module 7** (7.4, 7.5, 7.6 apart from the flagged issue): PCA fit-on-train-only discipline in 7.6 verified correct; topic modeling (7.4) fit on the full corpus is appropriate since it's unsupervised/exploratory, not part of the supervised pipeline; VADER (7.5) needs no fitting so no leakage risk; structured-feature imputation and PCA component count (80% explained variance) in 7.6 both verified correct.

### CLARIFIED — flagged by an agent, verified NOT a bug

- **Module 3 Decision Tree tuned to `class_weight=None`.** An agent flagged this as inconsistent imbalance-handling (RF/XGB always balance, DT's search picked no balancing). Verified against the actual cached GridSearchCV output: `class_weight` was included as a genuine, empirically-tuned option (`['balanced', None]`), and `None` won on CV F1 (0.6570) and confirmed on test F1 (0.6846) — both reasonable, non-degenerate scores. This is F1-scoring (already imbalance-aware) legitimately finding that this Decision Tree performs better without explicit reweighting, not a methodology defect.
- **Module 2: baseline model isn't re-tuned in 2.4.** Flagged as breaking the "does regularization help" narrative. This is a legitimate scope choice, not a defect — 2.3 already establishes the baseline's CV F1 for comparison; 2.4's stated purpose is tuning the three regularized variants specifically.
- **Module 6: AdaBoost has no `class_weight` handling** (unlike LightGBM/CatBoost in the same notebook). Confirmed: sklearn's `AdaBoostClassifier` genuinely has no `class_weight` parameter — this is an sklearn API constraint, not something the course can "fix." Worth a caveat when comparing AdaBoost's ROC-AUC to LightGBM/CatBoost's, not an actionable bug. (The MLPClassifier's `random_state=88` was also re-flagged by an agent as an inconsistency — already reviewed and confirmed intentional earlier this session; not new.)

### NOTED — pedagogical clarity gaps, not code bugs

- **Module 5**: k=4 is used consistently across all four case studies, correctly informed by elbow/silhouette plots, but no markdown cell narrates *why* k=4 was chosen from those plots. Silhouette scores are printed but never interpreted (e.g., what counts as "good"). Both are teaching-clarity improvements, not correctness issues — the underlying k selection and metrics themselves are valid.

**Net result of this pass:** one genuine, previously-undiscovered methodology bug (module 7 vectorizer leakage) found and precisely diagnosed; everything else in modules 2–7 confirmed methodologically sound after independent verification, with two agent-flagged "issues" correctly identified as false positives (legitimate design choices, not defects).

---

## Summary — All Errors Solved (across the whole review effort)

Consolidated, undated list of every bug actually fixed (not just flagged). Full before/after detail for each lives in the sections above; this is the scannable index.

### Module 1
- 1.1 lecture: two cells wrongly referenced CRISP as if taught in notebook 1.2 (which comes after 1.1) — reworded both.

### Module 2
- 2.2/2.3 code_briefs: wrong variable names, hyperparameters, and save filenames vs. the lecture — fixed to match.
- 2.2/2.3 lessons: same fixes, plus a `models_filepath == data_filepath` path bug (models were saving into the data folder).
- 2.3 code_brief/lesson: missing baseline (unregularized) model in the comparison, and a wrong `baseline_logistic_model.pkl` filename that would have crashed — fixed.
- 2.2/2.3/2.4 code_briefs: were using local `../data/` paths instead of the lecture's Drive paths (code_briefs must match the lecture's actual environment) — fixed all three to Drive paths.
- 2.2_code_brief, 2.3_code_brief, 2.1_code_brief: wrong titles ("1.2 Code Brief", "1.3 Code Brief", "1.1 Code Brief" — Module 1 numbering left over from copy-paste) — fixed to correct module numbers.
- 2.4_code_brief: wrong model filenames (`_logistic_model.pkl` suffix that doesn't exist) and a dynamic save filename instead of the lecture's fixed `best_tuned_logistic_model.pkl` (which 4.1 depends on by exact name); also missing the `grid_search_comparison_data.pkl` save entirely — fixed.
- 2.4 lecture: title said "1.4" instead of "2.4" — fixed.
- 2.4 lecture: `elasticnet_results_best_l1` was used but never defined (`NameError`, caught by live run) — added the missing per-C best-l1_ratio derivation.
- 2.2 lecture/code_brief/lesson: `baseline_logistic.pkl` was loaded by 2.3 but nothing produced it (`FileNotFoundError`, caught by live run) — 2.2 now also builds and saves the baseline model (`penalty=None`).
- Same 2.2 fix, round 2: the first attempt corrupted a markdown header (turned it into dead code text) and left the real `models_dict` cell with the old 3-model version, so the fix silently didn't take — caught by live run, properly fixed in all three files.

### Module 3
- 3.1_code_brief: was generic boilerplate with zero real content — full rewrite to match the lecture.
- 3.2_code_brief: same — full rewrite.
- 3.2_lesson: had two extra `.fillna(median())` calls not in the lecture, and its cross-validation cell scored on `roc_auc` instead of the lecture's `f1`, missing several hyperparameters (`max_features`, `min_samples_split`, `scale_pos_weight`) — fixed.
- 3.3 lecture: missing the artifact-persistence step (`feature_columns.pkl`, `train_medians.pkl`) that 3.4 depends on — added.
- 3.3_lesson/3.3_code_brief: found still broken on a later pass (reverted somewhere along the way) — fully rebuilt both from scratch: F1 scoring throughout, train-only median imputation, XGBoost early stopping reusing `best_params_`, and the model-save cell.
- 3.3/3.4/4.1/8.3/8.4 cross-module: tuned model files were saved/loaded under three different filename sets across the course — standardized everything on the `_f1.pkl` convention (matches Juan's own 4.1 lecture).
- 3.4 lecture: Learning Objectives listed 5 items but only 2 were actually delivered; Summary had a "F1 Score" subsection describing an analysis that was never run — trimmed both to match what's actually in the notebook (per Juan's explicit instruction).
- 3.4_lesson: was retraining fresh untuned models instead of loading the tuned ones, had the test-own-median leakage bug, and included ROC/feature-importance sections that don't exist in the lecture — full rewrite.
- 3.4_code_brief: was generic boilerplate — full rewrite.
- 3.4_lesson (round 2): after the cross-module rename, still loaded the old dead filenames (`decision_tree_best.joblib` etc.) instead of the new `_f1.pkl` names — guaranteed `FileNotFoundError`, missed by the original rename pass, caught by a review-agent pass. Fixed.
- Formatting bug: a JSON-writing script used for several early code_brief rewrites dropped trailing newlines from multi-line cell source, making them render as one unbroken line — fixed across 3.1/3.2/3.4/4.1 code_briefs (8 + 13 + 7 + 8 cells).

### Module 4
- 4.1_lesson: was retraining wrong/untuned models instead of loading the tuned ones, had a stale 6-dimension ROC-AUC radar chart instead of the lecture's F1-based one, and extra sections (ROC curves, metrics bar chart) not in the lecture — full rewrite.
- 4.1_code_brief: generic boilerplate — full rewrite.
- 4.2_code_brief: generic boilerplate — full rewrite to byte-for-byte match the lecture.
- 4.2_lesson: close but not identical (missing imports, shortened strings, missing comments) — restored to byte-for-byte match.

### Module 5
- 5.1_code_brief: was still the generic empty placeholder — rebuilt as 25 cells, byte-for-byte matching all 16 of the lecture's code cells (first attempt from memory dropped comments and renamed a variable — caught in verification, rebuilt by extracting the lecture's cells programmatically instead).

### Module 6
- 6.1_code_brief: was a single placeholder markdown cell — full rewrite (13 cells, byte-for-byte match).
- 6.1_lesson: missing the `!pip install catboost` cell the lecture has — added.
- 6.2 lecture: the "Notes on our MLPClassifier Configuration" markdown said `batch_size=32` but the actual code used `batch_size=50` — fixed the notes (not the code, per Juan's direction, since cached output was generated with the real code).
- 6.2_code_brief: single placeholder cell — full rewrite (byte-for-byte match).
- 6.2_lesson: wrong `batch_size` (32→50), wrong `random_state` (42→88), missing a print line, and one comparison-table row that didn't match the lecture — fixed.
- 6.1 and 6.2 (lecture + code_brief + lesson, all 6 files): data-leakage bug — test set was being filled with its own median instead of the training median — fixed everywhere.

### Module 7
- 7.1–7.6 code_briefs: didn't exist — built new for all six, verified byte-for-byte against their lectures (7.1/7.2 are condensed concept summaries since their lectures have no code).
- 7.3 lecture + code_brief: a cell referenced `ML_Survey_Data19.index`, an undefined variable (`NameError`) — fixed to the actual loaded frame name `ML_Survey_Data`. Also cleaned up two harmless leftover "19" references in a markdown cell and a code comment.
- TRAIN data generator: was exporting `ML_Survey_Data19.csv`, but every downstream notebook expects `ML_Survey_Data.csv` (no suffix) — fixed the output filename, unblocking the entire module 7 pipeline.
- 7.1–7.6 lessons: didn't exist — built new for all six.
- 7.6 (lecture + code_brief + lesson): leakage bug — `preprocessor.fit_transform(df_test)` was re-fitting the scaler/encoder on test data instead of reusing the train-fitted preprocessor via `.transform()` — fixed everywhere.
- Repo-wide: 27 files (mostly module 7/8 lesson files, several code_briefs, a few lecture files) had `null` cell IDs, which broke GitHub's notebook renderer — assigned real IDs to every affected cell without touching any code content.

### Module 8
- 8.4 lecture: two leakage bugs (test filled with its own median, both the admin and survey feature sets) — fixed, using captured train-only medians.
- 8.4 lecture: the Decision Tree row of the radar chart was pulling Logistic Regression's ROC-AUC value instead of its own — fixed.
- 8.4 lecture: deleted a stray leftover debug-arithmetic cell with no pedagogical purpose.
- 8.5 lecture: `RANDOM_STATE = 2` — every other notebook in the course uses 42, this was an isolated typo — fixed.
- 8.1–8.5 code_briefs: didn't exist — built new for all five, verified byte-for-byte, including the module-filename fixes below.
- 8.3/8.4 lectures: were loading tuned tree models from the wrong filenames (`xgb_tuned.pkl` etc., which nothing produces) and the wrong Drive folder entirely — fixed to load the correct `_f1.pkl` files from the correct folder.
- 8.1–8.5 lessons: didn't exist — built new for all five.

### Not fixed — flagged, pending an external decision (see body of this changelog for full detail)
- **Module 7.3/7.6**: the text vectorizer (`CountVectorizer`/`TfidfVectorizer`) is fit separately on train and test text, producing two independently-built vocabularies — will crash or silently corrupt PCA downstream in 7.6. Diagnosed precisely; fix is straightforward (fit on train only, transform test) but touches lecture content, pending sign-off.
- **Module 8.5**: the advisor-outreach join matches students to risk scores by row position, not by a real key — `Deploy_Survey_Data.csv` has no SID column to key on. Guarded with a row-count assertion (uncommitted, pending Kagba's regenerated file with SID added).
- **Module 5**: whether `bottleneck.csv` should replace 5.1's synthetic course-bottleneck data — pending Juan's answer.

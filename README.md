# String Manipulation (Filter “RPA” Sentences) in UiPath

A simple UiPath project that reads a paragraph, splits it into sentences, keeps only sentences that contain **“RPA”**, and types those sentences into an open Microsoft Word document.

> Tested with **UiPath Studio 2025.0.172 STS** (Modern experience).

---

## 1) What this workflow does

* Stores a paragraph in a string variable.
* Splits the paragraph into sentences by the period (`.`).
* Loops through each sentence.
* Checks if the sentence contains the text **RPA**.
* Types the matching sentences into an **already open** Word document (one per line).

---

## 2) Requirements

* UiPath Studio **2025.0.172 STS** (or later).
* Microsoft Word installed.
* A **blank Word document** opened before running the workflow.

---

## 3) Project setup (from scratch)

1. Open **UiPath Studio** → **Home** → **Process**.
2. Name the project: `String Manipulations – Practice 2`.
3. Choose **Language**:

   * Prefer **C#** (default in new Studio).
   * VB is also fine; use the VB expressions shown below.
4. Click **Create**.
5. In **Main.xaml**, add a **Sequence** and rename it to:
   `Sequence – Split and Filter Sentences (RPA)`.

---

## 4) Variables

Open the **Variables** panel (bottom of Designer). Create:

| Name          | Type      | Scope    | Default   |
| ------------- | --------- | -------- | --------- |
| `newText`     | String    | Sequence | *(empty)* |
| `newSentence` | String\[] | Sequence | *(empty)* |

> How to set `String[]`: Type column → **Array of \[T]** → choose **String**.

---

## 5) Add activities and expressions

### A) Assign the paragraph

Add an **Assign**:

* **To**: `newText`
* **Value** (use straight quotes `" "`):

**C#**

```csharp
"Robotic process automation (RPA) is a software technology that makes it easy to build, deploy, and manage software robots that emulate human actions interacting with digital systems and software. RPA streamlines workflows, which makes organizations more profitable, flexible, and responsive. RPA is noninvasive and can be rapidly implemented to accelerate digital transformation. It also increases employee satisfaction, engagement, and productivity by removing mundane tasks from their workdays."
```

**VB**

```vb
"Robotic process automation (RPA) is a software technology that makes it easy to build, deploy, and manage software robots that emulate human actions interacting with digital systems and software. RPA streamlines workflows, which makes organizations more profitable, flexible, and responsive. RPA is noninvasive and can be rapidly implemented to accelerate digital transformation. It also increases employee satisfaction, engagement, and productivity by removing mundane tasks from their workdays."
```

### B) Split into sentences

Add an **Assign**:

* **To**: `newSentence`
* **Value**:

**C#**

```csharp
newSentence = newText.Split('.');
```

**VB**

```vb
newSentence = newText.Split("."c)
```

### C) For Each (loop through sentences)

Add a **For Each** activity:

* **In**: `newSentence`
* **TypeArgument**: **String**
* **Item name**: `item` (or keep the default; just use the same name in the If condition)

### D) If (keep sentences that contain “RPA”)

Place an **If** activity **inside** the For Each Body.

**Recommended (robust) condition**:

**C#**

```csharp
!string.IsNullOrWhiteSpace(item) && item.Contains("RPA")
```

**VB**

```vb
Not String.IsNullOrWhiteSpace(CStr(item)) AndAlso CStr(item).Contains("RPA")
```

> If you already set **TypeArgument = String** in VB and prefer the original pattern:
>
> ```vb
> item IsNot Nothing AndAlso item.Trim.Length > 0 AndAlso item.Contains("RPA")
> ```

### E) Type Into Word (Then branch of If)

1. Add **Use Application/Browser**.

   * Click **Indicate window on screen** → select the **open Word document**.
   * In Properties: **Close** = **Never**.
2. Inside **Do**, add **Type Into**.

   * Click **Indicate target on screen** → click **inside the white page**.
   * **Text**:

**C#**

```csharp
item.Trim() + "[k(enter)]"
```

**VB**

```vb
item.Trim & "[k(enter)]"
```

3. **Type Into** properties:

   * **Activate** = On
   * **ClickBeforeTyping** = On
   * **Input Mode**: try **Simulate** first; if nothing is typed, turn **Simulate** off
   * (Optional) **DelayBetweenKeys** = 10 ms

---

## 6) Run the workflow

* Keep the **Word document open** on the desktop.
* In Studio, click **Run**.
* The sentences that contain **“RPA”** appear in Word, each on a new line.

---

## 7) Troubleshooting

**Problem:** *If condition shows an error*

* Ensure **For Each → TypeArgument = String**.
* Use **straight quotes** `"RPA"`, not curly quotes.
* Match your **language**:

  * C#: `newText.Split('.')`
  * VB: `newText.Split("."c)`
* If your loop variable is not `item` (e.g., `CurrentItem`), use that exact name in the condition.

**Problem:** *Nothing is typed in Word*

* In **Type Into**, disable **Simulate** and test again.
* Ensure **Activate** and **ClickBeforeTyping** are on.
* Re-indicate the text area inside the page (not the ribbon).
* Set **WaitForReady = Complete** on the scope and Type Into (if available).

**Problem:** *Cannot find UI element*

* Re-indicate the **Word window** and the **Type Into target**.
* Make sure the document is visible and not minimized.
* Close other Word windows if multiple are open.

**Problem:** *Extra blanks or leading spaces*

* Keep `item.Trim()` in the Type Into text.
* Use the robust If condition shown above to skip empty items.

---

## 8) Optional improvements

* **Faster structure**
  Open the Word scope **once** outside the loop:

```
Use Application/Browser (Word)
  For Each item in newSentence
    If (match)
      Type Into (item.Trim + Enter)
```

* **Case-insensitive match**
  Match `RPA`, `rpa`, etc.

**C#**

```csharp
!string.IsNullOrWhiteSpace(item) &&
item.IndexOf("RPA", StringComparison.OrdinalIgnoreCase) >= 0
```

**VB**

```vb
Not String.IsNullOrWhiteSpace(CStr(item)) AndAlso
CStr(item).IndexOf("RPA", StringComparison.OrdinalIgnoreCase) >= 0
```

* **No-UI Word approach (alternative)**
  Install **UiPath.Word.Activities**, then use **Use Word File** → **Append Text** for each matching sentence.

---

## 9) Expressions cheat sheet

**C#**

```csharp
newSentence = newText.Split('.');
!string.IsNullOrWhiteSpace(item) && item.Contains("RPA")
item.Trim() + "[k(enter)]"
```

**VB**

```vb
newSentence = newText.Split("."c)
Not String.IsNullOrWhiteSpace(CStr(item)) AndAlso CStr(item).Contains("RPA")
item.Trim & "[k(enter)]"
```

---

## 10) Notes

* Use **straight quotes** `"` in all expressions.
* `newSentence` is an **array of String** (`String[]`).
* The last array element can be empty if the paragraph ends with a period; the robust If handles this.

---

## 11) Folder / files

* No external files required.
* The Word document is created and opened by you before running.
* The workflow types directly into the open document.

---

## 12) License

Personal or educational use. Adjust as needed for your environment.


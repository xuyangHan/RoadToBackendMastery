# Solving Problems with Regular Expressions: From LeetCode to Production

## 🧠 Introduction: Why Regex Still Matters (Even in the Age of AI)

Regular expressions — or regex — are a compact and powerful way to define patterns in text. They allow developers to match, extract, and manipulate strings with surprising precision. If you've ever written a validator for email addresses, parsed logs, or searched for specific formats in a file, chances are you've bumped into regex.

### So what *is* regex?

At its core, a regex is just a string pattern. For example:

- `\d+` matches one or more digits.
- `^Hello` matches any string that starts with "Hello".
- `[a-zA-Z0-9]{6,}` matches alphanumeric strings that are at least 6 characters long.

Regex operates like a **mini-language for patterns**, and it works across many programming environments — including C#, JavaScript, Python, MySQL, and shell scripts.

### Why is it powerful?

The real magic of regex is: **write once, match many**. A single pattern can capture a huge variety of text cases. Instead of multiple lines of loops and if-statements, you can solve problems with a concise, expressive rule.

### But... isn’t everyone just using tools now?

Totally. Today, tools like ChatGPT, regex101.com, and various IDEs can generate regex patterns for you in seconds. This is especially helpful when regex is a "side task" — not your main focus.

However, **blindly copying** a regex can be risky. You might:

- Miss edge cases the tool didn’t consider
- Use an overly strict or too lenient pattern
- Introduce performance issues (e.g. catastrophic backtracking)

### Understanding regex = power

Even a basic understanding of regex can help you:

- Debug patterns that don’t behave as expected
- Customize generated regex for your specific use case
- Avoid overcomplicated or broken expressions
- Communicate clearly with teammates when reviewing code

### When regex shines ✨

Regex is best used when:

- **Validating** input (emails, phone numbers, usernames, etc.)
- **Extracting** structured data from semi-structured text (e.g., logs)
- **Filtering** large datasets by pattern (e.g., in SQL queries)
- **Replacing** patterns in a string with something else (cleaning text)

If used right, regex can be the most underrated time-saver in your toolbox.

---

## 🔧 Regex in Action: C# and MySQL Use Cases

Before diving into LeetCode problems, let’s first explore how regex is used in two practical environments: C# and MySQL.

### 📚 Basic Regex Concepts You Should Know

- `\d`: digit (0–9)
- `\w`: word character (letters, digits, underscore)
- `.`: any character (except newline)
- `*`: zero or more
- `+`: one or more
- `?`: optional
- `^`: start of string
- `$`: end of string
- `[]`: character set
- `()`: grouping
- `{n,m}`: quantity (min to max)

You don’t need to memorize everything — just knowing the basics lets you read, understand, and tweak generated patterns confidently.

---

### 🔵 Examples in C-Sharp

C# has built-in support for regex in the `System.Text.RegularExpressions` namespace.

#### 1. Email Validator

```csharp
string email = "user@example.com";
bool isValid = Regex.IsMatch(email, @"^[\w\.-]+@[\w\.-]+\.\w{2,}$");
Console.WriteLine(isValid); // true
```

#### 2. Phone Number Parser (US-style)

```csharp
string phone = "(123) 456-7890";
Match match = Regex.Match(phone, @"\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}");
Console.WriteLine(match.Success); // true
```

#### 3. Log Line Filter (find lines starting with ERROR)

```csharp
string log = "ERROR: Something failed\nINFO: All good\nERROR: Another failure";
foreach (Match m in Regex.Matches(log, @"^ERROR:.*", RegexOptions.Multiline))
{
    Console.WriteLine(m.Value);
}
// Output:
// ERROR: Something failed
// ERROR: Another failure
```

---

### 🟡 Examples in MySQL

MySQL supports regex via the `REGEXP` keyword in `WHERE` clauses (or `RLIKE`, which is a synonym).

#### 1. Find usernames starting with a capital letter

```sql
SELECT * FROM users
WHERE username REGEXP '^[A-Z]';
```

#### 2. Find emails from a specific domain

```sql
SELECT * FROM users
WHERE email REGEXP '@example\\.com$';
```

#### 3. Extract logs that mention error codes (e.g., "ERR123", "ERR500")

```sql
SELECT * FROM logs
WHERE message REGEXP 'ERR[0-9]{3}';
```

Note: MySQL’s regex engine is a bit simpler than what you'd get in C# or Python — it doesn’t support all advanced features like lookaheads/lookbehinds — but it’s still powerful for filtering text-based data.

---

## 💡 Regex Meets LeetCode: Solving Problems Elegantly

### 🧬 Problem [3475]: DNA Pattern Recognition

You're given a table `Samples` containing columns: `sample_id`, `dna_sequence`, and `species`. Your task is to identify patterns in the DNA sequences — such as whether they start with a specific codon (`ATG`), end with a stop codon (`TAA`, `TAG`, `TGA`), or contain specific substrings (`ATAT`, `GGG`).

#### 👴 Traditional SQL Solution (using `LIKE`)

```sql
SELECT 
    sample_id, 
    dna_sequence, 
    species,

    CASE
        WHEN dna_sequence LIKE 'ATG%' THEN 1
        ELSE 0
    END AS has_start,

    CASE 
        WHEN dna_sequence LIKE '%TAA' THEN 1
        WHEN dna_sequence LIKE '%TAG' THEN 1
        WHEN dna_sequence LIKE '%TGA' THEN 1
        ELSE 0
    END AS has_stop,

    CASE 
        WHEN dna_sequence LIKE '%ATAT%' THEN 1
        ELSE 0
    END AS has_atat,

    CASE 
        WHEN dna_sequence LIKE '%GGG%' THEN 1
        ELSE 0
    END AS has_ggg

FROM Samples
ORDER BY sample_id;
```

It works, but it’s verbose and repetitive. You can’t help but feel there’s a cleaner way…

#### ✨ Regex-Powered SQL One-Liner

```sql
SELECT sample_id,
       dna_sequence,
       species,
       dna_sequence REGEXP '^ATG'           AS has_start,
       dna_sequence REGEXP 'TAA$|TAG$|TGA$' AS has_stop,
       dna_sequence REGEXP 'ATAT'           AS has_atat,
       dna_sequence REGEXP 'GGG'            AS has_ggg
  FROM Samples
 ORDER BY sample_id;
```

#### 🧠 Regex Breakdown

- `^ATG`: `^` asserts the start of the string, so this matches sequences beginning with `ATG`.
- `TAA$|TAG$|TGA$`: the `|` acts as an OR operator, and `$` asserts the end — so this checks if the sequence ends in one of the stop codons.
- `ATAT`: simple substring match — if `ATAT` appears anywhere.
- `GGG`: same — looks for the triplet G anywhere in the string.

#### 💡 Key Insight

With just a few regex patterns, we simplified several `LIKE`-based conditionals into expressive, compact logic. This not only improves readability but also makes it easier to expand or tweak logic later.

---

## ⚖️ Balance: Use Tools, But Understand the Output

### 🤖 The Modern Developer Reality

Let’s be honest — regex isn’t something most of us sit down to write from scratch every day. Between:

- 🔍 [regex101.com](https://regex101.com/)
- 💬 ChatGPT-style pattern generation
- 🧠 AI-powered IDEs (like Copilot or IntelliCode)

You can get a working regex in seconds.

### 🛠️ But Understanding Matters

Here’s where things can go wrong:

- ❌ **Copy-pasted regex fails silently** — your pattern almost matches, but not quite.
- 🔍 **No one knows what the pattern means** — even you, one week later.
- 🐌 **Performance issues** — catastrophic backtracking from an over-greedy pattern.

### 🧭 Tips for Using Regex (Responsibly)

1. **Know the basics**:
   - `\d` = digit
   - `\w` = word character
   - `.` = any character
   - `^` and `$` = anchors
   - `*`, `+`, `?` = quantifiers
   - `[]` = character sets
   - `()` = grouping and capturing

2. **Test your patterns** before deploying — always.

3. **Comment complex patterns**, especially in production code:

   ```csharp
   // Match US phone number: (123) 456-7890 or 123-456-7890
   var pattern = @"\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}";
   ```

4. **Maintain a personal cheat sheet or snippets repo**. It's way faster than Googling “regex email validator” for the 30th time.

---

## ✅ Conclusion

Regex isn’t about showing off — it’s about solving problems fast and clearly.

In interviews, scripts, and even production queries, regex can save you lines of code and hours of work. You don’t need to be a regex wizard. But just like with Git or SQL, being **literate** in regex makes you more efficient and effective.

And in a world where tools are getting smarter, your regex understanding is what helps you:

- Spot flaws
- Fix edge cases
- Avoid misuse

So go ahead — use ChatGPT to generate that pattern.  
But read it. Understand it. Tweak it.  
That’s what makes you a pro.

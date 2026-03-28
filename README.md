# 🦊 Fox Lab – Individual Oral Code Defense Script

**Course:** 6DWEB & 6IMAN – Holy Angel University  
**Date:** March 27, 2026  
**Team Members:**
- Marcelino, Princess Camille (Project Lead)
- Roque, Daryl John Clark
- Bautista, Mark Anthony
- Bermas, Estella Mae
- Gamboa, Rodel Vincent

---

## 🎬 SLIDE 1: Introduction (Marcelino, Princess Camille)

**Script:**

> "Good day, everyone. I'm Princess Camille Marcelino, the project lead of Fox Lab. This project is presented as part of our partial requirements for **6DWEB** and **6IMAN** at **Holy Angel University**.
>
> Today, each member of our team will present their individual contributions to the Fox Lab Cybersecurity Awareness & Training Platform. Our presentation will cover the following:
>
> 1. **Project Overview** – What Fox Lab is and its architecture
> 2. **Individual Module Walkthroughs** – Each member will discuss their assigned PHP modules, database tables, and how they work together
> 3. **Live Modification Tasks** – Each of us will perform one live code modification to demonstrate understanding of our assigned codebase
> 4. **Deployment** – Our platform is already deployed using **ProFreeHost**, which provides an admin control panel that allows us to make file modifications directly on the live website
>
> Fox Lab is a full-stack PHP web application with a MySQL database consisting of **23 normalized tables**. It features a phishing email simulator, a password security tester, an online code compiler with Python and Java support, an 85-term cybersecurity glossary, a blog system with admin management, and a security tips page. All pages use PDO prepared statements for SQL injection prevention and the `e()` helper function for XSS protection.
>
> Before we dive into the technical details, let me give you a quick tour of the platform so you can see what Fox Lab actually looks and feels like."

---

## 🌐 SLIDE 1.5: Live Platform Walkthrough (Marcelino, Princess Camille)

**Script:**

> *(Opens the deployed Fox Lab website in the browser.)*
>
> "Alright — welcome to **Fox Lab**. This is our live deployed website hosted on **ProFreeHost**. Let's walk through it.
>
> **Starting with the Home Page** — this is the first thing users see. Right at the top we have our hero section — *'Master Cybersecurity, One Skill at a Time.'* Below that, you'll see live platform statistics pulled straight from the database — total users, total glossary terms, and total phishing scenarios. These aren't hardcoded — they update automatically as the database grows. Scroll down a bit and you'll see our **platform features overview** — each card represents a core tool. And at the bottom, our **partner organizations** — CSIA, CISCO, and GDG — rendered dynamically from the `partners` table.
>
> **Let's jump to the Blogs page.** Here you can see our featured blog displayed prominently at the top, and below it, all published articles with category filter buttons — Education, Cybersecurity, Technology, and more. These filter buttons are generated dynamically from the `categories` table. Click on any blog and you'll see the full article with **auto-linked glossary terms** — hover over the highlighted words and they link directly to our glossary. That's Daryl's feature and it works beautifully.
>
> **Now let's check out the Cybersecurity Glossary.** This is a searchable, filterable dictionary of **85 cybersecurity terms**. You can filter by category — Protocols, Threats, Concepts, Encryption, Compliance, Tools — or browse alphabetically with the A–Z navigation. Watch this — as I type in the search box, **predictive suggestions** appear instantly. Click on any term and you get the full definition, pronunciation, usage context, related terms, threats, and resources. Logged-in users can also **bookmark** their favorite terms with one click.
>
> **Next up — Security Tips.** Five detailed cybersecurity best practices, each with expandable step-by-step guidance. Think of this as Fox Lab's quick-reference survival guide for staying safe online.
>
> **Now here's where it gets fun — the Phishing Simulator.** This is an interactive email inbox that presents realistic phishing and legitimate email scenarios. Users have to decide: *Is this phishing or is it legitimate?* You get a checklist of indicators to analyze before making your call. After answering, the system reveals red flags and tells you exactly what you missed. It's like a cybersecurity escape room — but with emails. Each session gives you **4 random scenarios** out of 15 in the database, so it's different every time.
>
> **Let's move to the Password Security Tester.** Type any password and watch the **real-time analysis** — five criteria light up as you type: length, uppercase, lowercase, numbers, and symbols. The strength bar fills up and changes color. And here's the best part — it checks your password against **100 known compromised passwords** loaded from our database. All of this runs entirely in the browser — the actual password **never touches our server**. Only metadata is logged for analytics.
>
> **And finally — the crown jewel — Fox Code, our Online Compiler.** This supports **Python and Java** with real code execution. You can write code in the editor, hit Run, and see the output instantly — powered by the Piston sandboxed execution engine with a local fallback. On the left sidebar, you'll see your saved projects and global demo projects. Switch languages and the **Quick Reference panel** updates with language-specific commands. And over here — **interactive tutorials** — 8 lessons for Python, 8 for Java, loaded from our database. Click any tutorial and the code loads right into the editor ready to run.
>
> **One more thing — the Admin Dashboard.** *(Logs in as admin.)* This is the backend control center. Platform overview stats, user activity breakdown with quiz counts and password checks per user, password strength distribution charts, and the **Blog Management System** where admins can create, edit, and delete posts with thumbnail uploads, HTML content editing, and a live preview toggle.
>
> So that's Fox Lab — **six major features**, all database-driven, all secured with PDO prepared statements and XSS prevention, all deployed and running live. Now let me walk you through the technical details of what I personally built."

---
> Let me now walk through my own contribution."

---

## 🔧 SLIDE 2: Marcelino – Fox Code Online Compiler & Database Schema (Princess Camille)

**Script:**

> "I was responsible for two major components: designing the **23-table normalized relational database schema** and implementing the **Fox Code Online Compiler** with Python and Java support.
>
> Starting with the **database**, I designed all 23 tables in `database_backup.sql`. The relationship map looks like this:
>
> - `users` is the central table, referenced by `blogs`, `projects`, `pw_logs`, `quiz_results`, `bookmarks`, and `enrollments`
> - `scenarios` has child tables `red_flags` and `indicators`, both with `ON DELETE CASCADE`
> - `terms` connects to `term_links`, `threats`, and `resources`
> - `languages` connects to `tutorials`, `quick_refs`, and `projects`
> - `partners` connects to `org_features`
>
> For the **compiler**, let me walk through the execution flow. When a user opens `compiler.php`, `requireLogin()` is called at the top. The page handles three POST actions — `create`, `save`, and `delete` — all scoped to the logged-in user:

```php
// compiler.php – save action
if ($_POST['action'] === 'save' && isset($_POST['id'])) {
    $stmt = $pdo->prepare("UPDATE projects SET code = :code, filename = :fn, language = :lang WHERE id = :id AND user_id = :uid");
    $stmt->execute([
        ':code' => $_POST['code'] ?? '',
        ':fn' => trim($_POST['filename'] ?? 'Untitled.py'),
        ':lang' => trim($_POST['language'] ?? 'python'),
        ':id' => (int)$_POST['id'],
        ':uid' => $userId
    ]);
}
```

> "Notice the `WHERE id = :id AND user_id = :uid` — this ensures users can only modify their own projects.
>
> The sidebar loads projects with:

```php
$stmtProjects = $pdo->prepare("SELECT id, filename, language, is_recent FROM projects WHERE user_id = :uid OR user_id IS NULL ORDER BY is_recent ASC, updated_at DESC");
```

> "The `OR user_id IS NULL` allows global demo projects to appear for all users.
>
> The Quick Reference panel queries:

```sql
SELECT qr.command, qr.description FROM quick_refs qr
JOIN languages l ON qr.language_id = l.id
WHERE l.slug = :lang ORDER BY qr.display_order ASC
```

> "When the user clicks **Run Code**, `compiler.js` sends an async POST to `api/execute.php`. This API does several things:
>
> 1. Enforces `POST` method — returns HTTP 405 otherwise
> 2. Validates language against a **server-side whitelist**: `['python', 'java']`
> 3. Sends a cURL request to the **Piston** sandboxed execution engine
> 4. If Piston fails, falls back to **local execution** via `proc_open()`

```php
// api/execute.php – local Python execution
function executeLocally(string $pythonBin, string $code, string $stdin): array {
    $tmpDir  = sys_get_temp_dir() . DIRECTORY_SEPARATOR . 'foxlab_' . uniqid();
    mkdir($tmpDir, 0700, true);
    $tmpFile = $tmpDir . DIRECTORY_SEPARATOR . 'main.py';
    file_put_contents($tmpFile, $code);

    $desc = [
        0 => ['pipe', 'r'],  // stdin
        1 => ['pipe', 'w'],  // stdout
        2 => ['pipe', 'w'],  // stderr
    ];

    $proc = proc_open("\"$pythonBin\" \"$tmpFile\"", $desc, $pipes, $tmpDir);
    // ...
    cleanup($tmpDir);
    return makeResult($stdout, $stderr, $exit);
}
```

> "The key security point here: `proc_open()` uses an **explicit pipe array** rather than shell passthrough, preventing shell injection. And `cleanup()` runs unconditionally after every execution, removing all temp files via `unlink()` and `rmdir()`.
>
> Tutorials are loaded by `api/tutorials.php`:

```sql
SELECT l.slug AS lang_slug, t.title, t.description, t.code
FROM tutorials t JOIN languages l ON t.language_id = l.id
ORDER BY l.slug ASC, t.display_order ASC
```

> "This returns 16 lessons — 8 Python and 8 Java — grouped by language as JSON, which `compiler.js` consumes to populate the tutorial panel."

### ✅ Live Modification Task – Marcelino

> "Now for my live modification. Currently, the `execute.php` API only allows Python and Java. I will **add JavaScript to the allowed languages list** so the API accepts it."

**Task:** In `api/execute.php`, find the `$allowed` array and add `'javascript'` to it.

**Before:**
```php
$allowed = ['python', 'java'];
```

**After:**
```php
$allowed = ['python', 'java', 'javascript'];
```

> "This single-line change extends our compiler's language validation. The Piston API already supports JavaScript, so this is all that's needed on the backend to accept it. In a full implementation, we'd also update the frontend language selector and `LANG_CONFIG` in `compiler.js`, but for the API layer, this one change is sufficient."

---

## 📖 SLIDE 3: Roque – Cybersecurity Glossary & Security Tips (Daryl John Clark)

**Script:**

> "I'm Daryl John Clark Roque, and I developed the **Cybersecurity Glossary** and the **Security Tips** features.
>
> The glossary in `terms.php` is a single file that handles **three separate request types**. Let me walk through each one.
>
> **First: AJAX Bookmark Toggle.** When a logged-in user clicks the bookmark button, JavaScript sends a POST with `action=toggle_bookmark`. The PHP handler checks authentication first:

```php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['action']) && $_POST['action'] === 'toggle_bookmark') {
    header('Content-Type: application/json');
    if (!isLoggedIn()) {
        echo json_encode(['error' => 'Login required']);
        exit;
    }
    $termId = (int)($_POST['term_id'] ?? 0);
    $userId = (int)$_SESSION['user_id'];
    if ($termId <= 0) {
        echo json_encode(['error' => 'Invalid term']);
        exit;
    }
```

> "Notice: `term_id` is cast to `(int)` and checked against `<= 0`. `user_id` comes exclusively from `$_SESSION`, never from POST data. It then checks if a bookmark exists and toggles accordingly — INSERT to add, DELETE to remove.
>
> **Second: AJAX Search Suggestions.** When the user types in the glossary search box, `terms.php?action=suggest` is hit:

```sql
SELECT id, title, category FROM terms
WHERE title LIKE :q OR definition LIKE :q2
ORDER BY CASE WHEN title LIKE :q3 THEN 0 ELSE 1 END, title ASC
LIMIT 8
```

> "The `CASE WHEN` expression prioritizes **exact prefix matches** over partial matches, so if you type 'Fire', 'Firewall' appears before 'Configure Firewall Settings'.
>
> **Third: The main page render.** The term list applies dynamic WHERE clauses depending on which filters are active. Category, letter, search text, and bookmark filters all stack. For the selected term's detail view, I load **related terms** using this UNION query:

```sql
SELECT t.id, t.title FROM terms t
WHERE t.id IN (
    SELECT linked_id FROM term_links WHERE term_id = :tid1
    UNION
    SELECT term_id FROM term_links WHERE linked_id = :tid2
)
ORDER BY t.title ASC
```

> "The UNION handles bidirectional relationships — if term A links to term B, this query returns B when viewing A, and A when viewing B, without storing duplicate rows.
>
> The `term_links` table uses a `UNIQUE KEY (term_id, linked_id)` constraint to prevent duplicate cross-references, and both FKs reference `terms` with `ON DELETE CASCADE`.
>
> The `bookmarks` table also enforces `UNIQUE KEY (user_id, term_id)` at the database level, preventing duplicate bookmarks even if the JavaScript fails. Both FKs use `ON DELETE CASCADE` — so if a user or term is deleted, the bookmark records are automatically cleaned up.
>
> Another feature I built is the **auto-linking of glossary terms in blog content**. In `blog.php`, the `linkGlossaryTerms()` function scans blog HTML, matches glossary term titles, and wraps the first occurrence with a link to the glossary page — while skipping text already inside `<a>` tags.
>
> The **Security Tips** page, `tips.php`, uses a static PHP array for the five tips and their expanded step-by-step guidance. Each tip has a `why` explanation and a `steps` array rendered as an ordered list. There's currently no admin UI for tips — it's hardcoded in PHP."

### ✅ Live Modification Task – Roque

> "For my live modification, I will **change the search suggestion limit from 8 results to 5** in the AJAX suggest handler in `terms.php`."

**Task:** In `terms.php`, find the AJAX suggest query and change `LIMIT 8` to `LIMIT 5`.

**Before:**
```php
$stmt = $pdo->prepare("SELECT id, title, category FROM terms WHERE title LIKE :q OR definition LIKE :q2 ORDER BY 
    CASE WHEN title LIKE :q3 THEN 0 ELSE 1 END, title ASC LIMIT 8");
```

**After:**
```php
$stmt = $pdo->prepare("SELECT id, title, category FROM terms WHERE title LIKE :q OR definition LIKE :q2 ORDER BY 
    CASE WHEN title LIKE :q3 THEN 0 ELSE 1 END, title ASC LIMIT 5");
```

> "This reduces the suggestion dropdown from 8 to 5 results, making it more compact. The same change could also be applied to `api/search_terms.php` if we wanted to keep them consistent."

---

## 📝 SLIDE 4: Bautista – Blog Management & Admin Dashboard (Mark Anthony)

**Script:**

> "I'm Mark Anthony Bautista. I developed the **Blog Management System** in `admin-blogs.php` and the **Admin Dashboard** in `admin-dashboard.php`.
>
> Both pages start with `requireAdmin()` at the very top — this function, defined in `config/db.php`, checks the session role and redirects non-admins immediately:

```php
function requireAdmin(): void {
    if (!isAdmin()) {
        $isSubPage = (basename(dirname($_SERVER['SCRIPT_FILENAME'])) === 'pages');
        header('Location: ' . ($isSubPage ? '../index.php' : 'index.php'));
        exit;
    }
}
```

> "In `admin-blogs.php`, the request lifecycle processes **deletion first**. When `$_GET['delete']` is set, the system fetches the blog's `image_url` before running the DELETE query:

```php
$stmt = $pdo->prepare("SELECT image_url FROM blogs WHERE id = :id");
$stmt->execute([':id' => $delId]);
$row = $stmt->fetch();
if ($row && !empty($row['image_url']) && strpos($row['image_url'], 'uploads/') === 0) {
    $filePath = __DIR__ . '/../' . $row['image_url'];
    if (file_exists($filePath)) unlink($filePath);
}
$pdo->prepare("DELETE FROM blogs WHERE id = :id")->execute([':id' => $delId]);
```

> "This ensures the uploaded file is cleaned up from the filesystem before the database row is removed.
>
> For **blog creation and editing**, the form handler processes thumbnail uploads with proper security. MIME type is validated using `finfo_open()` rather than just trusting the file extension:

```php
$finfo = finfo_open(FILEINFO_MIME_TYPE);
$mimeType = finfo_file($finfo, $_FILES['thumbnail']['tmp_name']);
finfo_close($finfo);
```

> "Filenames are randomized using `bin2hex(random_bytes(4))` to prevent overwrites and directory traversal attacks.
>
> The `blogs` table has 18 fields. An important design decision is **intentional denormalization** — both `category` (text label) and `category_id` (FK to `categories`) are stored. This preserves the category label even if the `categories` row is deleted, since the FK uses `ON DELETE SET NULL`.
>
> When marking a post as featured, the system first unmarks all others:

```php
if ($isFeatured) {
    $pdo->exec("UPDATE blogs SET is_featured = 0");
}
```

> "This ensures only one featured post exists at a time.
>
> The **Admin Dashboard** aggregates data from across the entire platform using LEFT JOINs. The user activity breakdown query joins `users` with subqueries for `quiz_results`, `pw_logs`, `bookmarks`, `projects`, `enrollments`, and `blogs`:

```php
SELECT u.id, u.full_name, u.email, u.role,
    COALESCE(qr.quiz_count, 0) AS quiz_count,
    COALESCE(pw.pw_checks, 0) AS pw_checks,
    -- ... more subqueries
FROM users u
LEFT JOIN (SELECT user_id, COUNT(*) AS quiz_count ... FROM quiz_results GROUP BY user_id) qr ON u.id = qr.user_id
-- ... more LEFT JOINs
```

> "The `COALESCE` ensures zero values are shown instead of NULL for users with no activity. The password strength distribution chart queries `pw_logs` with `GROUP BY strength_level` for the bar chart visualization.
>
> The editor panel also includes JavaScript helpers — `insertTag()` wraps selected text in HTML tags, `insertImg()` prompts for an image URL and inserts an `<img>` tag, and `togglePreview()` renders the raw HTML content in a preview div."

### ✅ Live Modification Task – Bautista

> "Now, before I make a code-level modification, let me first demonstrate the admin blog management UI in action.
>
> Imagine a scenario: what if a featured partner organizationm for this demonstration the CSIA wants you to feature their Facebook post within the blogs page? With our admin blog system, this is very straightforward — you can either **create a new blog** or **edit an existing one**. For this demonstration, I will be **editing this existing featured blog** to also show the edit function working.
>
> *(Opens admin-blogs.php in the browser, clicks the Edit button on the featured blog, updates the content field to include an embedded Facebook post link or reference, and clicks Save.)*
>
> As you can see, the edit form pre-populates all 18 fields from the database using:

```php
$stmt = $pdo->prepare("SELECT * FROM blogs WHERE id = :id LIMIT 1");
```

> When I hit Save, the UPDATE query fires with all the fields including the new content, and `updated_at` refreshes automatically via `ON UPDATE CURRENT_TIMESTAMP` in the schema. The thumbnail upload, MIME validation with `finfo_open()`, and filename randomization with `bin2hex(random_bytes(4))` all work the same whether creating or editing. That's the UI side working as intended.
>
> Now for my **code-level modification**. I will **change the blog listing sort order from newest-first to oldest-first** on the admin blog table."

**Task:** In `admin-blogs.php`, find the blog list query and change `DESC` to `ASC`.

**Before:**
```php
$allBlogs = $pdo->query("SELECT id, title, category, author, image_url, is_featured, views, published_at FROM blogs ORDER BY published_at DESC")->fetchAll();
```

**After:**
```php
$allBlogs = $pdo->query("SELECT id, title, category, author, image_url, is_featured, views, published_at FROM blogs ORDER BY published_at ASC")->fetchAll();
```

> "This changes the admin table to show the oldest posts first instead of the newest. This could be useful for reviewing posts in chronological order. To revert, we simply change `ASC` back to `DESC`."
---

## 🎨 SLIDE 5: Bermas – UI/UX Design, Theme Engine & Password Tester (Estella Mae)

**Script:**

> "I'm Estella Mae Bermas. I developed the platform's **complete visual identity**, the **responsive front-end architecture**, the **dark/light theme engine**, and the **front-end UI of the Password Security Tester**.
>
> The **design system** uses **Inter** for body text and **Fira Code** for code/monospace elements. All visual tokens are defined as CSS custom properties on `:root` for light mode and overridden under `[data-theme="dark"]`, covering backgrounds, text colors, borders, shadows, and accent colors across every page.
>
> Theme toggling is stored in `localStorage`, so the user's preference persists across sessions on the same device.
>
> For the **Password Security Tester**, the page `checker.php` uses a **two-panel layout** — input on the left, analysis on the right. The entire strength evaluation runs **client-side** in `checker.js`. The five criteria are evaluated in real-time as the user types:

```javascript
// assets/js/checker.js
const criteria = {
    length:  password.length >= 8,
    upper:   /[A-Z]/.test(password),
    lower:   /[a-z]/.test(password),
    number:  /[0-9]/.test(password),
    symbol:  /[^A-Za-z0-9]/.test(password)
};
```

> "Each criterion updates a colored dot and status text — green for PASS, red for FAIL, gray for PENDING. The overall strength level is calculated from the count of passing criteria, with a bonus for length over 16 characters.
>
> The **weak password comparison** also runs entirely client-side. On page load, `checker.js` fetches the list from `api/weak_passwords.php`:

```javascript
// assets/js/checker.js
window._weakPasswords = [];
fetch('../api/weak_passwords.php')
    .then(res => res.json())
    .then(data => { window._weakPasswords = data; });
```

> "This API endpoint queries:

```php
// api/weak_passwords.php
$stmt = $pdo->query("SELECT password FROM weak_passwords ORDER BY id ASC");
$passwords = $stmt->fetchAll(PDO::FETCH_COLUMN);
echo json_encode($passwords);
```

> "The `weak_passwords` table is standalone — 100 rows of known compromised passwords. The comparison happens in the browser:

```javascript
const isCompromised = window._weakPasswords && window._weakPasswords.includes(password.toLowerCase());
```

> "**Crucially, the actual password is never transmitted to the server.** When the user clicks 'Analyze Password Strength,' only **metadata** is submitted via AJAX POST:

```javascript
// assets/js/checker.js
const formData = new FormData();
formData.append('action', 'log_strength');
formData.append('strength_level', strengthLevel);
formData.append('char_count', password.length);
formData.append('has_uppercase', criteria.upper ? 1 : 0);
// ... more metadata, but never the password itself
```

> "On the server side, `checker.php` handles this POST and inserts into `pw_logs`:

```php
// pages/checker.php
$stmt = $pdo->prepare("INSERT INTO pw_logs (user_id, strength_level, char_count, has_uppercase, has_lowercase, has_numbers, has_symbols, is_compromised, ip_address) VALUES (:uid, :sl, :cc, :hu, :hl, :hn, :hs, :ic, :ip)");
```

> "The `pw_logs` table has `user_id` as an FK to `users` with `ON DELETE SET NULL`, so logs are preserved even if the user account is deleted. This data is later consumed by the Admin Dashboard for the password strength distribution chart using `GROUP BY strength_level`.
>

### ✅ Live Modification Task – Bermas

> "For my live modification, I will **change the minimum password length criterion from 8 to 12 characters** in the real-time checker in `checker.js`."

**Task:** In `assets/js/checker.js`, find the `length` criterion in the `updateStrengthIndicators` function and change `>= 8` to `>= 12`.

**Before:**
```javascript
const criteria = {
    length:  password.length >= 8,
    upper:   /[A-Z]/.test(password),
```

**After:**
```javascript
const criteria = {
    length:  password.length >= 12,
    upper:   /[A-Z]/.test(password),
```

> "This raises the minimum length requirement to align with current industry best practices — NIST now recommends at least 12 characters. The same change should also be applied in the `analyzePassword()` function further down in the same file for consistency."

---

## 🐟 SLIDE 6: Gamboa – Phishing Simulator & Partners Page (Rodel Vincent)

**Script:**

> "I'm Rodel Vincent Gamboa. I developed the **Phishing Email Simulator** and the **Partners page**.
>
> The Phishing Simulator in `phishing.php` is the platform's core training tool. It requires authentication via `requireLogin()` at the top. Let me walk through the full flow.
>
> **Session-based quiz initialization.** On first visit or when `$_GET['restart']` is set, the system fetches all scenario IDs, shuffles them, and stores 4 randomly selected IDs in the session:

```php
// pages/phishing.php
if (!isset($_SESSION['phishing_quiz_ids']) || isset($_GET['restart'])) {
    $ids = array_column($allScenarios, 'id');
    shuffle($ids);
    $_SESSION['phishing_quiz_ids'] = array_slice($ids, 0, min(4, count($ids)));
    $_SESSION['phishing_quiz_pos'] = 0;
}
```

> "This means every time a user takes the quiz, they get a **different random subset** of the 15 pre-seeded scenarios.
>
> Each scenario's **indicators** — the checklist items users can check before submitting — are loaded via:

```php
$stmtInd = $pdo->prepare("SELECT * FROM indicators WHERE scenario_id = :sid ORDER BY id ASC");
$stmtInd->execute([':sid' => $currentScenario['id']]);
$indicators = $stmtInd->fetchAll();
```

> "Each indicator has an `is_correct` field (0 or 1) that marks whether it's a genuine phishing indicator. This is used after submission to show the user which indicators they correctly or incorrectly identified.
>
> **Response submission** is handled via AJAX. When the user clicks 'Report as Phishing' or 'Mark as Legitimate,' `phishing.js` sends a POST with `action=submit_response`. In the JavaScript, buttons are **immediately disabled** to prevent double submission:

```javascript
// assets/js/phishing.js
const buttons = document.querySelectorAll('.phishing-action-btn');
buttons.forEach(btn => {
    btn.disabled = true;
    btn.style.opacity = '0.5';
    btn.style.pointerEvents = 'none';
});
```

> "The PHP handler then validates the response server-side:

```php
// pages/phishing.php
$stmtCheck = $pdo->prepare("SELECT is_phishing FROM scenarios WHERE id = :id");
$stmtCheck->execute([':id' => $scenarioId]);
$scenario = $stmtCheck->fetch();
```

> "Correctness is computed by comparing the user's response against `is_phishing`. The result is inserted:

```php
$stmtSave = $pdo->prepare("INSERT INTO quiz_results (user_id, scenario_id, user_response, is_correct) VALUES (:uid, :sid, :resp, :correct)");
$stmtSave->execute([
    ':uid' => $_SESSION['user_id'],
    ':sid' => $scenarioId,
    ':resp' => $userResponse,
    ':correct' => $isCorrect ? 1 : 0
]);
```

> "Note that `user_id` is always taken from `$_SESSION['user_id']`, **never from POST data** — this prevents users from spoofing results for other accounts.
>
> The handler then fetches **red flags** for the scenario:

```php
$stmtRF = $pdo->prepare("SELECT flag_title, flag_description, flag_icon FROM red_flags WHERE scenario_id = :sid");
$stmtRF->execute([':sid' => $scenarioId]);
$flags = $stmtRF->fetchAll();
```

> "These are returned as JSON and rendered client-side in the analysis panel. The response also includes cumulative correct and incorrect counts.
>
> The `scenarios` table stores 15 pre-seeded entries with fields like `sender_email`, `body_html`, `is_phishing` (TINYINT boolean), and `difficulty` (ENUM: easy, medium, hard). `red_flags` and `indicators` are child tables — both use `ON DELETE CASCADE` referencing `scenarios`. `quiz_results` has FKs to both `users` and `scenarios`, both with `ON DELETE CASCADE`.
>
> For the **Partners page**, `partners.php` loads organizations and their features:

```php
$stmtPartners = $pdo->query("SELECT * FROM partners ORDER BY display_order ASC");
$partners = $stmtPartners->fetchAll();

foreach ($partners as $p) {
    $stmtFeatures = $pdo->prepare("SELECT feature, icon FROM org_features WHERE partner_id = :pid ORDER BY id ASC");
    $stmtFeatures->execute([':pid' => $p['id']]);
    $partnerFeatures[$p['id']] = $stmtFeatures->fetchAll();
}
```

> "The `org_features` table has an FK to `partners` with `ON DELETE CASCADE`, so if a partner is removed, their features are automatically cleaned up."

### ✅ Live Modification Task – Gamboa

> "For my live modification, I will **change the number of random scenarios per quiz session from 4 to 3** in `phishing.php`."

**Task:** In `pages/phishing.php`, find the line where `array_slice` selects quiz scenarios and change `4` to `3`.

**Before:**
```php
$_SESSION['phishing_quiz_ids'] = array_slice($ids, 0, min(4, count($ids)));
```

**After:**
```php
$_SESSION['phishing_quiz_ids'] = array_slice($ids, 0, min(3, count($ids)));
```

> "This shortens each quiz session from 4 scenarios to 3. This could be useful for quick assessment sessions. The progress bar and final results calculation will automatically adjust since they're based on the count of `$quizIds`."

---

## 🎬 SLIDE 7: Closing (Marcelino)

**Script:**

> "To summarize, Fox Lab is a full-stack PHP/MySQL cybersecurity training platform with:
>
> - **23 normalized database tables** with proper foreign key constraints and cascading deletes
> - **PDO prepared statements** on every database query across all files
> - **XSS prevention** via the `e()` helper on all dynamic output
> - **Session-based authentication** with role enforcement via `requireLogin()` and `requireAdmin()`
> - **Deployed on ProFreeHost**, where we use the hosting control panel to make live file modifications
>
> Each of us demonstrated a live code modification on our respective modules, showing that we understand the codebase we built. Thank you for your time."

---

## 📋 Quick Reference: Live Modification Summary

| Member | File | Modification | Line Change |
|---|---|---|---|
| **Marcelino** | `api/execute.php` | Add `'javascript'` to allowed languages array | `$allowed = ['python', 'java']` → `$allowed = ['python', 'java', 'javascript']` |
| **Roque** | `pages/terms.php` | Reduce search suggestion limit | `LIMIT 8` → `LIMIT 5` |
| **Bautista** | `pages/admin-blogs.php` | Change blog sort order | `ORDER BY published_at DESC` → `ORDER BY published_at ASC` |
| **Bermas** | `assets/js/checker.js` | Raise minimum password length | `password.length >= 8` → `password.length >= 12` |
| **Gamboa** | `pages/phishing.php` | Reduce quiz scenarios per session | `min(4, count($ids))` → `min(3, count($ids))` |

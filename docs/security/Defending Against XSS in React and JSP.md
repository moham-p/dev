Cross-Site Scripting (XSS) is one of the most common and dangerous vulnerabilities in web applications. While modern 
frontend frameworks like React provide strong built‑in protection, older technologies like JSP rely on manual escaping, 
making it easier to introduce XSS bugs.

In this post, we compare how JSP (as a legacy server-rendered frontend) and React (as modern client-side frontend) 
handle XSS.

---

## React: XSS Is Mostly Handled — If You Let It

✅ **React escapes by default**
React’s rendering engine automatically escapes all content interpolated into JSX. For example:

```jsx
const userInput = "<script>alert('XSS')</script>";
return <div>{userInput}</div>;
```

This renders as:

```html
<div>&lt;script&gt;alert('XSS')&lt;/script&gt;</div>
```

✅ No script executes — React encoded it safely.

---

❌ React provides an escape hatch called `dangerouslySetInnerHTML` — and the name itself is a warning. It reflects React’s **security-first design**: by including the word *“dangerously”*, the framework signals clearly that **you’re bypassing its built-in protections and assuming responsibility** for content safety.

**Unsafe Example:**

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

If `userInput` is `<script>alert('XSS')</script>`, it will execute in the browser.

---

✅ **If you must inject HTML, sanitize it first**
For HTML coming from a CMS, WYSIWYG editor, or any untrusted source, sanitize it with a library like [DOMPurify](https://github.com/cure53/DOMPurify):

```jsx
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(userInput);
return <div dangerouslySetInnerHTML={{ __html: clean }} />;
```

---

## JSP: XSS Prevention Is Manual — and Easy to Get Wrong

JSP is a server-rendered frontend technology used in older Java-based applications. Unlike React, it does **not** escape output by default. If you print user input directly into HTML, it will be rendered as-is — including any malicious scripts.

---

❌ **Vulnerable example:**

```jsp
<%= request.getParameter("username") %>
```

If `username` is `<script>alert('XSS')</script>`, the script will run in the browser.

---

✅ **The safe and recommended way: `<c:out>`**
Use JSTL's `<c:out>` to escape user content:

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<c:out value="${param.username}" />
```

This safely escapes HTML characters like `<`, `>`, and `&`.

---

### How `<c:out>` compares to other options

| Technique                          | Escapes HTML? | Handles nulls? | Recommended |
| ---------------------------------- | ------------- | -------------- | ----------- |
| `${param.username}`                | ❌             | ❌              | ❌           |
| `<%= request.getParameter(...) %>` | ❌             | ❌              | ❌           |
| `fn:escapeXml(...)`                | ✅             | ❌              | ⚠️ (okay)   |
| `<c:out value="..." />`            | ✅             | ✅              | ✅           |

---

## Why `<c:out>` is used even for non-user data

You might see `<c:out>` used consistently throughout JSP code — even when rendering values that aren’t obviously user-driven. That’s intentional:

1. **Defense-in-depth** — a value that’s safe today might come from user input tomorrow.
2. **Safe defaults** — always escaping removes guesswork and reduces mistakes.
3. **Future-proofing** — requirements change; code should be resilient.
4. **Consistency** — uniform escaping makes templates easier to audit and maintain.

---
Here’s the concise version of the new section with just **SonarQube** mentioned:

---

## Scanning for XSS in Legacy Frontends Like JSP

Manually spotting XSS vulnerabilities in large JSP codebases can be difficult. Tools like **SonarQube** help by automatically detecting insecure patterns — such as unescaped output with `<%= ... %>` or unsanitized user input.

SonarQube supports Java and JSP, integrates into CI/CD pipelines, and highlights risky lines in your code. It's a valuable safety net when maintaining or auditing legacy frontend systems.

---

Happy coding! 💻

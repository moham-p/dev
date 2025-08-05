# Defending Against XSS in React and JSP: What Every Backend Engineer Should Know

Cross-Site Scripting (XSS) is one of the most common and dangerous vulnerabilities in web applications. As a backend engineer, you might think of XSS as a frontend concern — but in reality, your templating logic, data exposure, and rendering practices play a crucial role in XSS prevention.

This post walks through **XSS defense in React and JSP**, explains best practices, shows vulnerable vs. safe examples, and clears up common misconceptions — even among experienced developers.

---

## React: XSS Is Mostly Handled — If You Let It

### ✅ React Escapes by Default

React’s rendering engine **automatically escapes all interpolated content**. For example:

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

### ❌ The Dangerous Trap: `dangerouslySetInnerHTML`

React gives you an escape hatch: `dangerouslySetInnerHTML`. As the name implies, **you should use it with extreme caution** — and only with sanitized input.

#### Unsafe Example:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

If `userInput` is `<script>alert('XSS')</script>`, it **will execute**.

---

### ✅ To Render Safe HTML, Sanitize It First

If you must render raw HTML, use a sanitization library like [DOMPurify](https://github.com/cure53/DOMPurify):

```jsx
import DOMPurify from 'dompurify';

const clean = DOMPurify.sanitize(userInput);
return <div dangerouslySetInnerHTML={{ __html: clean }} />;
```

---

### ✅ Use a Content Security Policy (CSP)

CSP is a defense-in-depth mechanism. Example header:

```
Content-Security-Policy: default-src 'self'; script-src 'self'
```

It prevents inline scripts and blocks external scripts from untrusted sources — a strong mitigation even if some XSS slips through.

---

### Summary: React XSS Defense

| Practice                  | XSS Safe? | Recommendation                 |
| ------------------------- | --------- | ------------------------------ |
| JSX `{userInput}`         | ✅         | Escaped by default             |
| `dangerouslySetInnerHTML` | ❌         | Use only with sanitized input  |
| DOMPurify                 | ✅         | Required if rendering HTML     |
| CSP headers               | ✅         | Additional layer of protection |

---

## JSP: XSS Prevention Is Manual — and Easy to Get Wrong

Unlike React, **JSP does not auto-escape output**. The default `<%= ... %>` syntax or `${param.value}` **will output raw text** — including scripts.

### ❌ Vulnerable Example:

```jsp
<%= request.getParameter("username") %>
```

If `username` is `<script>alert('XSS')</script>`, that code will **execute** in the browser.

---

### ✅ The Safe and Recommended Way: `<c:out>`

Use JSTL's `<c:out>` tag:

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<c:out value="${param.username}" />
```

This escapes HTML entities (`<`, `>`, `&`) and safely renders untrusted content as text.

---

### How `<c:out>` Compares to Other Options

| Technique                          | Escapes HTML? | Handles nulls? | Recommended |
| ---------------------------------- | ------------- | -------------- | ----------- |
| `${param.username}`                | ❌             | ❌              | ❌           |
| `<%= request.getParameter(...) %>` | ❌             | ❌              | ❌           |
| `fn:escapeXml(...)`                | ✅             | ❌              | ⚠️ (OK)     |
| `<c:out value="..." />`            | ✅             | ✅              | ✅           |

---

### Why `<c:out>` Is Used Everywhere — Even for Non-User Data

You may see `<c:out>` used even for rendering values that *don’t* come directly from user input. Here’s why:

#### 1. **Defense-in-Depth**

Even if a value is safe *now*, that may change. Treat everything as untrusted unless proven otherwise.

#### 2. **Safe Defaults**

Always escaping avoids case-by-case decisions — reducing the chance of mistakes.

#### 3. **Future-Proofing**

A value that’s from a config today may be from a DB tomorrow.

#### 4. **Consistency**

Using one consistent pattern — `<c:out>` — makes templates easier to read, maintain, and audit.

So yes, even for static or backend-generated data, **using `<c:out>` by default is intentional and correct.**

---

## Final Thoughts

* Default to escaping *everything* unless you have a very good reason not to.
* Got a legacy JSP system? Start by replacing raw output with `<c:out>`.
* Building in React? Stick to JSX rendering, and avoid raw HTML unless sanitized.

Let the frameworks help you — but never forget that **you** are the last line of defense.

---

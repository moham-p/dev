Cross-Site Scripting (XSS) is one of the most common and dangerous vulnerabilities in web applications. While modern 
frontend frameworks like React provide strong built‑in protection, older technologies like JSP rely on manual escaping, 
making it easier to introduce XSS bugs.

In this post, we compare how JSP (as a legacy server-rendered frontend) and React (as modern client-side frontend) 
handle XSS.

---

## React: XSS Is Mostly Handled — If You Let It

### React escapes by default

React’s rendering engine automatically escapes all content interpolated into JSX. For example:

```jsx
function App() {
  const [input, setInput] = useState('');

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <div>{input}</div>
    </div>
  );
}
```

If a user types:

```
<img src="x" onerror="alert('XSS')" />
```

React will render it **as text** (no image loads, and no script runs). 
This happens because React automatically replaces certain special HTML characters — like `<`, `>`, `&`, and `"` — with their safe HTML-escaped versions (`&lt;`, `&gt;`, `&amp;`, `&quot;`) before adding them to the page’s DOM.


### You can skip the built-in protection by using `dangerouslySetInnerHTML`

It reflects React’s security-first design: by including the word *“dangerously”*, the framework signals clearly that you’re bypassing its built-in protections and assuming responsibility for content safety. For example:

```jsx
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

If `userInput` contains `<script>alert('XSS')</script>`, the script will run. If you choose to use `dangerouslySetInnerHTML`, 
you take full responsibility for sanitizing the input — for example, by using a library like [DOMPurify](https://github.com/cure53/DOMPurify).


```jsx
import DOMPurify from 'dompurify';

function App() {
  const [input, setInput] = useState('');
  const clean = DOMPurify.sanitize(input);

  return (
    <div>
      <input value={input} onChange={e => setInput(e.target.value)} />
      <div dangerouslySetInnerHTML={{ __html: clean }} />
    </div>
  );
}
```

## JSP: XSS Prevention Is Manual — and Easy to Get Wrong

JSP is a server-rendered frontend technology used in older Java-based applications. Unlike React, it does **not** escape output by default. If you print user input directly into HTML, it will be rendered as-is — including any malicious scripts.

### Vulnerable example

```jsp
<%= request.getParameter("username") %>
```

If `username` is `<script>alert('XSS')</script>`, the script will run in the browser.

### The safe and recommended way###
Use JSTL's `<c:out>` to escape user content:

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<c:out value="${param.username}" />
```

This safely escapes HTML characters like `<`, `>`, and `&`.

### How `<c:out>` compares to other options

| Technique                          | Escapes HTML? | Handles nulls? | Recommended |
| ---------------------------------- | ------------- | -------------- | ----------- |
| `${param.username}`                | ❌             | ❌              | ❌           |
| `<%= request.getParameter(...) %>` | ❌             | ❌              | ❌           |
| `fn:escapeXml(...)`                | ✅             | ❌              | ⚠️ (okay)   |
| `<c:out value="..." />`            | ✅             | ✅              | ✅           |

## Why `<c:out>` is used even for non-user data

You might see `<c:out>` used consistently throughout JSP code — even when rendering values that aren’t obviously user-driven. That’s intentional:

1. **Defense-in-depth** — a value that’s safe today might come from user input tomorrow.
2. **Safe defaults** — always escaping removes guesswork and reduces mistakes.
3. **Future-proofing** — requirements change; code should be resilient.
4. **Consistency** — uniform escaping makes templates easier to audit and maintain.


## Enterprise-Ready Tools to Find XSS in Legacy JSP

Manually spotting XSS in large JSP codebases can be challenging, especially as projects grow and patterns become harder to track. This is where **SonarQube** (static application security testing, or SAST) helps — it analyzes source code to detect insecure patterns, such as unescaped `<%= ... %>` or unsanitized user input, and integrates into CI/CD pipelines to catch issues early. **Burp Suite** (dynamic application security testing, or DAST) complements this by scanning and probing a running application to uncover vulnerabilities that static analysis might miss. Using both static and dynamic testing together provides strong, complementary coverage when maintaining or auditing legacy frontend systems.

---

Happy coding! 💻

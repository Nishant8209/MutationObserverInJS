

```markdown
# 🏠 Host Application with Microfrontend Integration

This project demonstrates how to **integrate a microfrontend** into a host application, and how to use the **DOM MutationObserver API** to send messages from the host app to the microfrontend without relying on `postMessage` or direct function calls.


```
## 📂 Project Structure
```

host-app/
│── index.html   # Host application (loads microfrontend via fetch)
microfrontend/
│── dist/        # Build output after running `npm run build`
│   └── index.html   # Remote microfrontend served on [http://localhost:5003](http://localhost:5003)

```




> ⚡ After building the microfrontend, the generated `index.html` inside `/dist` is what the host app fetches and injects.

---

## 🚀 How It Works

### 1. Loading the Microfrontend

The host fetches the **remote app's `index.html`** from:


[http://localhost:5003/index.html](http://localhost:5003/index.html)
- Rewrites relative `href` and `src` paths to absolute URLs (`http://localhost:5003/...`).
- Injects `<link>` stylesheets and `<script>` files into the host DOM.
- Mounts the microfrontend into:

````
<body>
<div id="remote-container"></div>
<script>
 const modulePath = "http://localhost:5003"; // where CSS/JS should load from

      fetch(`${modulePath}/index.html`)
        .then((res) => res.text())
        .then((html) => {
          // Adjust relative paths to absolute remote URLs
          const adjustedHtml = html
            .replace(/(href|src)="\//g, `$1="${modulePath}/`)
            .replace(/(href|src)="\.?\/*/g, `$1="${modulePath}/`);

          // Create a container
          const container = document.createElement("div");
          container.innerHTML = adjustedHtml;

          // ✅ Append CSS links
          const arrLinks = container.getElementsByTagName("link");
          for (const link of arrLinks) {
            if (link.href.includes(modulePath)) {
              document.head.appendChild(link.cloneNode(true));
            }
          }

          // ✅ Inject into host
          document.getElementById("remote-container").appendChild(container);

          //Load the JS 
          const elemJS = container.getElementsByTagName("script")[0];
          const elemSScript = document.createElement("script");
          elemSScript.type = elemJS.type;
          elemSScript.src = elemJS.src;
          document.getElementById("remote-container").appendChild(elemSScript);

        })
        .catch((err) => console.error("Failed to load app:", err));
</script>
</body>
````

---

### 2. Sending Events via MutationObserver

Instead of sending events directly, the **Host App mutates the DOM** by inserting/updating a hidden `<div>` inside `#remote-container`.

Example:

```html
<body>
  <button id="sendEventBtn">Send Event via MutationObserver</button>

<script>
 document
            .getElementById("sendEventBtn")
            .addEventListener("click", () => {
              const remoteContainer = document.getElementById("remote-container");

              // Create or update a "message node"
              let msgNode = remoteContainer.querySelector("#host-message");
              if (!msgNode) {
                msgNode = document.createElement("div");
                msgNode.id = "host-message";
                msgNode.style.display = "none"; // hidden
                remoteContainer.appendChild(msgNode);
              }

              // Update message (this triggers MutationObserver in MFE)
              msgNode.textContent = JSON.stringify({
                time: new Date().toISOString(),
                message: "Hello from Host App 🚀",
              });

</script>
</body>
```

The **MutationObserver** in the microfrontend detects this change and reads the message.

---

### 3. Flow

1. User clicks **"Send Event via MutationObserver"** button in the host app.
2. Host app mutates the DOM inside `#remote-container`:

   ```json
   {
     "time": "2025-09-01T12:34:56Z",
     "message": "Hello from Host App 🚀"
   }
   ```
3. MutationObserver in the microfrontend detects the change.
4. Microfrontend parses the JSON and reacts accordingly.

---


## 🧩 Microfrontend Side

Inside the microfrontend, listen with a MutationObserver:

```js
const remoteContainer = document.getElementById("remote-container");

if (remoteContainer) {
  const observer = new MutationObserver(() => {
    const msgNode = remoteContainer.querySelector("#host-message");
    if (msgNode) {
      const data = JSON.parse(msgNode.textContent);
      console.log("📩 Microfrontend received:", data);
    }
  });

  observer.observe(remoteContainer, {
    childList: true,
    subtree: true,
    characterData: true,
  });
}
```

---

## 🛠️ How to Run

1. **Build and serve the microfrontend**:

   ```bash
   cd microfrontend
   npm run build
   npx serve dist -l 5003
   ```

2. **Open the host app**:

   * Run with a simple static server (e.g., `npx live-server` or `npx serve`).
   * Visit [http://localhost:5501](http://localhost:5501).

3. **Test communication**:

   * Click **Send Event via MutationObserver** in the host app.
   * Open the browser console of the microfrontend.
   * You’ll see the message logged.

---

## 🔑 Key Concepts

* **Microfrontend Integration** → The host dynamically loads another app’s built HTML.
* **MutationObserver API** → Detects DOM changes to enable indirect communication.
* **Custom Message Passing** → Messages are encoded as JSON in hidden DOM nodes.




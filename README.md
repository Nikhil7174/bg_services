# How Chromium Displays Web Pages, Profile archicture, running background services without displaying it in ui

This document describes how web pages are displayed in Chromium from the ground up and profile architecture. And we can solve the problem of running a headless tab in a headful window.
![HLD](https://www.chromium.org/developers/how-tos/getting-around-the-chrome-source-code/Content.png "hld")

---
# How Chromium Displays Web Pages,
## The Render Process

Chromium’s render process primarily handles communication with the browser via IPC. Key classes include:
- **RenderView**: Represents a web page, handles navigation, and interacts with the browser process.
- **RenderWidget**: Manages painting and input event handling for web elements.

### Threads in the Renderer
- **Render Thread**: Runs WebKit and Chromium’s rendering logic.
- **Main Thread**: Handles synchronous IPC communication with the browser.

![renderer-process](https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome/Renderingintherenderer-v2.png "renderer-process")
---

## The Browser Process

### Low-Level Objects
The I/O thread handles IPC and network communication. Key classes:
- **RenderProcessHost**: Manages IPC channels and dispatches view-specific messages.

### High-Level Objects
- **WebContents**: Top-level object representing a web page’s contents.
- **TabContentsWrapper**: Chromium-specific wrapper managing tabs.\

![browser-process](https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome/rendering%20browser.png "browser-process")
---

### 1. Web Content in Chromium

Web content is rendered by the **Renderer process**, which uses the Blink rendering engine to parse HTML, CSS, and JavaScript. The following steps are involved:

1. **Navigation and Parsing**:
   - The browser process receives a navigation request.
   - It assigns the request to a renderer process.
   - The renderer parses the HTML and builds a DOM (Document Object Model).

2. **Layout and Painting**:
   - The renderer computes layout, determining positions and styles.
   - Painting instructions are generated and sent to the compositor.

3. **Compositing and Rasterization**:
   - The compositor organizes the page into layers and rasterizes them into bitmaps.
   - These bitmaps are sent to the GPU for display.

## 2. Tab UI in Chromium

The **Tab UI** is managed by the browser process, which provides the interface for opening, closing, and interacting with tabs. Key components include:

- **Tab Strip**: Displays tabs and manages their state (e.g., active, loading).
- **Navigation Controls**: Provides controls like back, forward, reload, and address bar.
- **IPC Communication**: Uses Inter-Process Communication (IPC) to interact with renderer processes for loading and displaying content.

## 3. Combining Web Content and Tab UI

The browser process integrates the Tab UI with web content as follows:

- The browser creates a tab and assigns a renderer process for its content.
- The renderer sends the composited content to the browser.
- The browser process combines this content with the UI (e.g., tab strip and controls) for user interaction.

## 4. Creating Web Content Without UI

Web content can be created and run without attaching it to a visible UI, enabling tasks such as headless tabs. This is achieved through the following:

   - Chromium can run in a "headless" mode, where the browser executes web tasks without a graphical interface.
   - This is commonly used for automated testing and server-side rendering.


This document provides a comprehensive overview of how Chromium renders web pages, balancing efficiency with modularity to support its multi-process architecture.


# Profile Architecture  

The **Profile Architecture** in Chromium encompasses a framework designed to handle user data and session states effectively, even across multiple browser windows. Initially, this profile system supported core elements like cookies, history, bookmarks, and user preferences. However, as features expanded, the `Profile` class became increasingly complex, leading to challenges in maintainability and scalability.  

## Key Evolution  

### Early Design Challenges  
- The profile directly owned all states, creating a monolithic structure with over 58 getters, e.g., `Profile::GetHostContentSettingsMap()`.  
- Manual ordering of services caused startup and teardown issues, especially with interdependent services like Sync, history, and bookmarks.  

### Current Architecture (Post-2012)  
- The `Profile` class transitioned to a lightweight handle object with minimal state.  
- Introduction of **DependencyManager** and **two-phase shutdown** mechanisms to streamline service initialization and destruction.  

## Design Goals  
- **Maintainable Profile Size**: Prevent overloading the `Profile` class with all states.  
- **Robust Startup/Shutdown**: Ensure seamless service lifecycle management despite dependencies.  
- **Platform Compatibility**: Enable layered components for easier adaptation across platforms like iOS and desktop.  

---

## Adding New Services  

### KeyedService  
A service with a virtual destructor, managed by factories like `ProfileKeyedServiceFactory`. Includes an overridable `Shutdown()` method for safe teardown.  

### Factories  
Use `ProfileKeyedServiceFactory` for associating services with profiles, with mechanisms to customize behavior for various profile types (e.g., Incognito, Guest).  

Example factory setup:  

```cpp
class FooServiceFactory : public ProfileKeyedServiceFactory {  
 public:  
  static FooService* GetForProfile(Profile* profile);  
  static FooServiceFactory* GetInstance();  

 private:  
  FooServiceFactory();  
  ~FooServiceFactory() override;  
  std::unique_ptr<KeyedService> BuildServiceInstanceForBrowserContext(content::BrowserContext* context) const override;  
};
```
---

## Use Cases for Running Web Content Without Attaching to a Tab UI

Running web content in a Chromium-based browser without attaching it to a visible tab UI provides innovative opportunities for both users and developers. Below are the top 5 most impactful use cases:

### 1. **Background Tasks and Automation**
- Perform actions like data scraping, API polling, or auto-filling forms without user interaction.
- Notify users with alerts (e.g., emails, stock price updates) without needing an active tab.
- Enable efficient automation for workflows such as checking system statuses or routine tasks.

### 2. **Resource-Efficient Processes**
- Execute tasks like preloading content, processing offline web apps, or fetching updates in the background, conserving memory and CPU usage.
- Ideal for running low-priority or repetitive jobs without impacting foreground performance.

### 3. **Headless Testing and Automation**
- Enable developers to run headless browsing for automation and testing tools (e.g., Selenium or Puppeteer).
- Simplify server-side rendering (SSR) or creating static HTML snapshots for SEO.

### 4. **Custom Widgets and Overlays**
- Embed dynamic content such as weather updates, chat assistants, or quick links in browser overlays or widgets without requiring a tab UI.
- Create minimal UI components that enhance user productivity, like sidebar tools or pop-ups.

### 5. **Enhanced Security and Privacy**
- Process sensitive tasks, such as encrypting/decrypting content or fetching confidential information, in an isolated and non-visible environment.
- Allow secure and private browsing sessions for background tasks or API integrations.

These features not only enhance user experience but also make the browser a powerful tool for developers, automation enthusiasts, and power users.


## References
- [Displaying a Web Page in Chromium](https://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome/)
- [Profile Architecture in Chromium](https://www.chromium.org/developers/design-documents/profile-architecture/)

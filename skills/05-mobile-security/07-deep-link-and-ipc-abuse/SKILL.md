---
name: deep-link-and-ipc-abuse
domain: 05-mobile-security
description: Use when testing a mobile app's inter-process communication surface — deep links, URL schemes, intents, and exported components that other apps (or the web) can invoke maliciously.
difficulty: intermediate
tags: [mobile, ipc, deep-links, intents, android]
tools: [adb, drozer]
---

## Purpose

Mobile apps expose entry points other apps and the web can invoke — deep links / URL schemes, Android intents, and exported components (activities, services, broadcast receivers, content providers). These IPC surfaces are meant for legitimate integration, but if unprotected they let a malicious app or a crafted link trigger actions, access data, or inject input. This skill covers testing the deep-link and IPC attack surface, a class of mobile vulnerability unique to the platform's component model.

## When to use it

On any mobile assessment, after mapping the app's components (static analysis reveals exported components and registered URL schemes). It's especially relevant for apps that integrate with others or handle links, and it's an often-overlooked surface.

## Procedure

1. **Enumerate the IPC surface from the manifest/Info.plist.** Static analysis reveals it:
   - **Android:** exported activities/services/receivers/providers (`android:exported="true"`, or implicitly exported via intent-filters), and registered deep-link schemes/hosts.
   - **iOS:** registered URL schemes (Info.plist), and universal links.
   These are the entry points other apps and the web can reach.
2. **Test deep links / URL schemes.** Craft links that invoke the app and see what they trigger. Can a link reach a sensitive screen, pass parameters that get trusted, or trigger an action without authentication?
   ```
   adb shell am start -a android.intent.action.VIEW -d "app://path?param=value"    # Android deep link
   # iOS: open a custom-scheme URL ; test what the handler does with the input
   ```
   The key question: does the app treat deep-link input as *trusted*? Deep-link parameters are attacker-controllable (a malicious website or app can craft the link).
3. **Test exported Android components (`drozer` is the classic tool).** An exported activity another app can launch (bypassing the app's normal flow), an exported service another app can bind to, an exported broadcast receiver another app can trigger, or — the highest-risk — an exported **content provider** that leaks data or allows SQL injection:
   ```
   # drozer: enumerate + interact with exported components; test content providers for data leaks/injection
   ```
4. **Look for authentication/authorization bypass via IPC.** Can invoking a component directly bypass the login/authorization the normal flow enforces? An exported activity that shows sensitive data without checking auth is a common finding.
5. **Test for injection through IPC.** Deep-link and intent parameters, and content-provider queries, can carry injection (SQLi in a content provider, parameter injection into a WebView) if the app trusts the input.
6. **Check WebView + deep-link combinations** — a deep link that loads an attacker-controlled URL into a WebView, or that reaches a WebView with weak configuration, can be a serious flaw (JavaScript bridge abuse, loading arbitrary content).
7. **Report by what the exposed surface allows** — data leak, action without auth, injection, or just information disclosure.

## Cheatsheet

```
apps expose entry points other apps + the web can invoke: deep links/URL schemes, intents,
  exported components (activity/service/receiver/PROVIDER). unprotected -> abuse.

enumerate (from manifest/Info.plist)
  Android: exported components (android:exported=true / implicit via intent-filter) + deep-link schemes
  iOS: URL schemes + universal links

test
  DEEP LINKS: craft link -> what does it trigger? sensitive screen? trusted params? action w/o auth?
    adb shell am start -a android.intent.action.VIEW -d "app://path?param=x"
    KEY Q: does the app TRUST deep-link input? (attacker-controllable via malicious site/app)
  EXPORTED COMPONENTS (drozer): launch activity (bypass flow) | bind service | trigger receiver
    | CONTENT PROVIDER (highest risk: data leak / SQL injection)
  AUTH BYPASS via IPC: invoke component directly -> bypass login/authz the normal flow enforces?
  INJECTION via IPC: deep-link/intent params, content-provider queries (SQLi, WebView param injection)
  WEBVIEW + deep link: load attacker URL into WebView / JS bridge abuse
report by what it allows (data leak / action-no-auth / injection / info disclosure)
```

## Reading the surface

- **An exported content provider leaking data or vulnerable to SQL injection** = the highest-risk IPC finding; another app queries it and extracts data or injects. Content providers are the most dangerous exported component. Test them thoroughly.
- **A deep link reaching a sensitive screen or triggering an action without authentication** = auth bypass via IPC; the normal flow enforces login, but the direct link skips it. A common finding.
- **The app trusting deep-link/intent parameters** = attacker-controllable input treated as trusted; a malicious website or app crafts the link, so the parameters must be validated like any untrusted input.
- **An exported activity/service another app can invoke** = a bypass of the app's normal flow; assess what invoking it directly allows (data access, action, state change).
- **A deep link loading an attacker URL into a WebView** = potentially serious (arbitrary content, JS bridge abuse); WebView + deep-link combinations warrant scrutiny.
- **Injection through a content provider or intent parameter** = the app trusts IPC input; SQLi or parameter injection with the impact of the underlying flaw.
- **Minimal exported surface, validated deep-link input, protected components** = the good state.

## The fix

- **Minimise the exported surface** — don't export components (`android:exported="false"`) unless they genuinely need to be reachable by other apps; explicitly set exported status.
- **Treat deep-link/intent/IPC input as untrusted** — validate it like any external input; a malicious app or website controls it. Don't trust deep-link parameters.
- **Enforce authentication/authorization on IPC-reachable actions** — invoking a component directly must not bypass the auth the normal flow requires.
- **Protect content providers** — require permissions, parameterise queries (no SQLi), and don't expose sensitive data.
- **Require permissions on exported components** that must be exported, so only authorised callers can invoke them.
- **Harden WebViews** reached via deep links (no loading arbitrary URLs, careful JS bridge exposure).

## Pitfalls

- **Overlooking the IPC surface.** Deep links and exported components are an often-forgotten attack surface unique to mobile; enumerate them from the manifest and test them.
- **Trusting deep-link/IPC input.** It's attacker-controllable (a malicious site or app crafts it); treat it as untrusted and validate. Trusting it is the root of most IPC flaws.
- **Exported content providers.** The highest-risk component — data leaks and SQL injection; require permissions and parameterise queries.
- **Auth bypass via direct component invocation.** Invoking a component directly can skip the login the normal flow enforces; enforce auth on the component itself.
- **Unnecessarily exported components.** Every exported component is attack surface; export only what must be, and set exported status explicitly (Android defaults changed over versions).

## References

- OWASP MASTG (platform interaction, IPC testing) and MASVS
- drozer (Android IPC assessment) documentation; adb intent commands
- Android exported components / intents and iOS URL schemes / universal links documentation
- The android-static-analysis and mobile-api-traffic skills

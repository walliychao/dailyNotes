# file
    - lexical scope (declaration point 确定)
    - dynamic this binding (invoke point 确定)
    - prototype chain (link objects with .prototype)
    - 任务队列: 宏任务可以有多个队列, 微任务队列只有一个
        - 宏任务：script（全局任务）, setTimeout, setInterval, setImmediate, I/O, UI rendering.
        - 微任务：process.nextTick, Promise, Object.observer, MutationObserver.

# 0402
- xss(Cross Site Scripting) csrf(Cross-site request forgery) xssi(Cross-Site Script Inclusion)

The difference to XSS is easy: during an XSS malicious code is placed into a victim’s page, during an XSSI victim’s code is included in a malicious page. On the surface XSSI and CSRF look similar, because in both a request is sent from a malicious page to a different domain and in both cases the request is executed in the context of the logged in user. The key difference is the goal. In a CSRF, the attacker wants to execute state-changing actions inside a victim’s page, like transferring money in an online banking application. In an XSSI the attacker wants to leak data cross-origin, in order to then execute one of the aforementioned attacks.

- sourcemap 
[VLQ编码](https://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)


# trace

- hoist
    
    ```
        if (0) {
            var a = 3;
        }
        a; // undefined
    ```
    
    ```
         if (0) {
            a = 3;
        }
        a; // referenceError
    ```

- JS 
    - lexical scope (declaration point 确定)
    - dynamic this binding (invoke point 确定)
    - prototype chain (link objects with .prototype)

# to do
- JS 继承方式 （proptotype...）
- JS 执行顺序 （setTimeOut, promise, nextTick...）
- web perfomance
- 浏览器 渲染过程
- 网络 跨域相关
- 框架使用
- 工具使用
- 移动端
- 后端
- fractal&rythm generate

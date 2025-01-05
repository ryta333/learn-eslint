## Eslint 运行原理解析

### 为什么要用Eslint
![alt text](images/why-lint.webp)
- 没有Eslint，我们写的代码就需要人工检查，格式可能千奇百态，每个人都风格都有所区别，阅读代码自然也就没那么通顺了，那代码的可维护性就会大大降低，严重的可能会影响开发者的心情，甚至出现奇奇怪怪的BUG
- 但是项目里使用了Eslint规范，那每个人的代码格式将会被统一，运行起来自然少了很多问题，阅读别人代码时也会更加顺畅

### eslint的一些配置
回顾下Eslint配置， 可作为配置的文件如下，从上往下为优先级顺序
> .eslintrc.js
  .eslintrc.yaml
  .eslintrc.yml
  .eslintrc.json
  .eslintrc
  package.json

```javascript
module.exports = {
    root: true,
    extends: [
        'eslint:recommended',
        'plugin:vue/vue3-essential',
    ],
    plugins: ['vue'],
    parser: '',
    parserOptions: {},
    rules: {},
    //...
}
```
- root 是否为根文件，若不是则会继续向上查找，由于eslint可支持层叠配置，因此寻找配置时会寻找所有父级的配置，然后合并成configArray, 并根据这个配置数组合并生产一个最终配置，当子目录的配置与父目录的冲突时，会优先使用子目录的
- plugins 插件，eslint插件包名都是以eslint-plugin-[pluginName] 来命名的，这里填写了vue则表示会引入eslint-plugin-vue插件，而plugin引入后只会加载对应的配置，具体使用还需要用户自己来写
![alt text](images/image.png)
- extends 集成，简单来说就是 plugin + rules
- parser 解析器，eslint默认使用espree，可指定不同解析器作为默认解析器，但必须满足规范
- parserOptions 解析器配置，简单来说就是自定义解析器时需要设置的配置
- rules 规则，老朋友了，就是校验规则的配置

#### 配置合并流程
![alt text](images/eslint-config-generate.webp)

### 运行流程
官方eslint架构图
![alt text](images/architecture.svg)

- lib/linter/ - 这个模块是核心的 Linter 类，它根据配置选项进行代码验证。这个文件不进行任何文件 I/O 操作，也不与控制台交互。对于其他需要验证 JavaScript 文本的 Node.js 程序，它们可以直接使用这个接口。

整体流程大概如下：
1. 脚本入口 -> 调用Eslint生成eslint实例
2. Eslint初始化调用CliEngine生成cli实例
3. CliEngine初始化配置，生成linter实例，配置信息，并存储在weakMap中
4. eslint实例根据参数调用lintText 或者 lintFiles、
5. lintFiles -> executeOnFiles -> verifyText -> linter.verifyAndFix -> verify -> _verifyWithoutProcessors 或者 _verifyWithProcessors -> parse AST -> runRules -> traverse node -> 
register rule -> tarverse nodeQuene -> emit rule -> get lintingProblems 
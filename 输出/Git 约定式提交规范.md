---
type: permanent
banner: Assets/Banner/pexels-faikackmerd-1025469.jpg
---
---

**关键词**: Git Commit Message, 规范

---

## 1	规范格式

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

## 2	提交类型枚举

提交说明包含了下面的结构化元素, 以向类库的使用者表明其意图:

- fix: 表示在代码库中修复了一个 bug.
	
- feat: 表示在代码库中新增了一个功能.
	
- BREAKING CHANGE: 在脚注中包含 BREAKING CHANGE: 或 `<类型>(范围)` 后面有一个 ! 的提交, 表示引入了破坏性 API 变更. 破坏性变更可以是任意类型提交的一部分.
	
- 除 fix: 和 feat: 之外, 也可以使用其它提交 类型 , 例如 @commitlint/config-conventional中推荐的 build:, chore:, ci:, docs:, style:, refactor:, perf:, test:, 等等.
	
	- build: 用于修改项目构建系统, 例如修改依赖库, 外部接口或者升级 Node 版本等;
	- chore: 用于对非业务性代码进行修改, 例如修改构建流程或者工具配置等;
	- ci: 用于修改持续集成流程, 例如修改 Travis, Jenkins 等工作流配置;
	- docs: 用于修改文档, 例如修改 README 文件, API 文档等;
	- style: 用于修改代码的样式, 例如调整缩进, 空格, 空行等;
	- refactor: 用于重构代码, 例如修改代码结构, 变量名, 函数名等但不修改功能逻辑;
	- perf: 用于优化性能, 例如提升代码的性能, 减少内存占用等;
	- test: 用于修改测试用例, 例如添加, 删除, 修改代码的测试用例等.

脚注中除了 `BREAKING CHANGE: <description>` , 其它条目应该采用类似 git trailer format 这样的惯例.

其它提交类型在约定式提交规范中并没有强制限制, 并且在语义化版本中没有隐式影响(除非它们包含 BREAKING CHANGE). 可以为提交类型添加一个围在圆括号内的范围, 以为其提供额外的上下文信息. 例如 feat(parser): adds ability to parse arrays. .

## 3	约定式提交规范

本文中的关键词 “必须(MUST)”, “禁止(MUST NOT)”, “必要(REQUIRED)”, “应当(SHALL)”, “不应当(SHALL NOT)”, “应该(SHOULD)”, “不应该(SHOULD NOT)”, “推荐(RECOMMENDED)”, “可以(MAY)” 和 “可选(OPTIONAL)” , 其相关解释参考 RFC 2119 .

- 每个提交都必须使用类型字段前缀, 它由一个名词构成, 诸如 feat 或 fix , 其后接可选的范围字段, 可选的 !, 以及必要的冒号(英文半角)和空格.
	
- 当一个提交为应用或类库实现了新功能时, 必须使用 feat 类型.
	
- 当一个提交为应用修复了 bug 时, 必须使用 fix 类型.
	
- 范围字段可以跟随在类型字段后面. 范围必须是一个描述某部分代码的名词, 并用圆括号包围, 例如: fix(parser):
	
- 描述字段必须直接跟在 `<类型>(范围)` 前缀的冒号和空格之后. 描述指的是对代码变更的简短总结, 例如: `fix: array parsing issue when multiple spaces were contained in string` .
	
- 在简短描述之后, 可以编写较长的提交正文, 为代码变更提供额外的上下文信息. 正文必须起始于描述字段结束的一个空行后.

- 提交的正文内容自由编写, 并可以使用空行分隔不同段落.
	
- 在正文结束的一个空行之后, 可以编写一行或多行脚注. 每行脚注都必须包含 一个令牌(token), 后面紧跟 `:<space>` 或 `<space>#` 作为分隔符, 后面再紧跟令牌的值.
	
- 脚注的令牌必须使用 - 作为连字符, 比如 Acked-by (这样有助于 区分脚注和多行正文). 有一种例外情况就是 BREAKING CHANGE, 它可以被认为是一个令牌.
	
- 脚注的值可以包含空格和换行, 值的解析过程必须直到下一个脚注的令牌/分隔符出现为止.
	
- 破坏性变更必须在提交信息中标记出来, 要么在 `<类型>(范围)` 前缀中标记, 要么作为脚注的一项.
	
- 包含在脚注中时, 破坏性变更必须包含大写的文本 BREAKING CHANGE, 后面紧跟着冒号, 空格, 然后是描述, 例如: BREAKING CHANGE: environment variables now take precedence over config files .
	
- 包含在 <类型>(范围) 前缀时, 破坏性变更必须通过把 ! 直接放在 : 前面标记出来. 如果使用了 !, 那么脚注中可以不写 BREAKING CHANGE:, 同时提交信息的描述中应该用来描述破坏性变更.
	
- 在提交说明中, 可以使用 feat 和 fix 之外的类型, 比如:docs: updated ref docs. .
	
- 工具的实现必须不区分大小写地解析构成约定式提交的信息单元, 只有 BREAKING CHANGE 必须是大写的.
	
- BREAKING-CHANGE 作为脚注的令牌时必须是 BREAKING CHANGE 的同义词.

---
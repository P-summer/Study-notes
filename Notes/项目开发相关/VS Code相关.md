## 插件
+ Chinese 简体中文
+ Vetur
+ ESLint
+ Git History 文件提交记录
+ GitLens — Git supercharged  修改文件人
+ Todo Tree 
+ any-rule 正则:F1打开正则列表
+ Vue Language Features (Volar)
+ Auto Close Tag
+ Dracula Official 颜色主题
+ vscode-icons  图标
+ Stylelint
+ Prettier - Code formatter setting配置
+ Code Spell Checker 单词校验
+ Live Sever 浏览器打开
+ HTML CSS Support

## setting.json
```
// Prettier - Code formatter
module.exports = {
  // 一行最多 100 字符
  printWidth: 100,
  // 使用 2 个空格缩进
  tabWidth: 2,
  // 不使用缩进符，而使用空格
  useTabs: false,
  // 行尾需要有分号
  semi: true,
  // 使用单引号
  singleQuote: true,
  // 对象的 key 仅在必要时用引号
  quoteProps: 'as-needed',
  // jsx 不使用单引号，而使用双引号
  jsxSingleQuote: false,
  // 末尾不需要逗号
  trailingComma: 'all',
  // 大括号内的首尾需要空格
  bracketSpacing: true,
  // jsx 标签的反尖括号需要换行
  jsxBracketSameLine: false,
  // 箭头函数，只有一个参数的时候，也需要括号
  arrowParens: 'always',
  // 每个文件格式化的范围是文件的全部内容
  rangeStart: 0,
  rangeEnd: Infinity,
  // 不需要写文件开头的 @prettier
  requirePragma: false,
  // 不需要自动在文件开头插入 @prettier
  insertPragma: false,
  // 使用默认的折行标准
  proseWrap: 'preserve',
  // 根据显示样式决定 html 要不要折行
  htmlWhitespaceSensitivity: 'css',
  // 换行符使用 lf
  endOfLine: 'auto',
  
  "vetur.format.defaultFormatter.html": "js-beautify-html", // 标签属性自动换行
  // tab 大小为2个空格
  "editor.tabSize": 2,
  // 保存时格式化
  "editor.formatOnSave": false,
  
};
```

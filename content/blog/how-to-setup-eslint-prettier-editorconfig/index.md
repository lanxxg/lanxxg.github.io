---
title: 代码风格指南
date: "2019-09-03"
description: ESLint, Prettier and Editorconfig.
---

> Eslint确保语法规则和代码风格保持良好的形态，Prettier则以特定的代码格式自动格式化代码。

如上所述，Prettier关注代码格式化，Eslint会检测语法规则和代码规范。当你设置了Prettier，你可以设置在保存文件的时候格式化，这样就不用再担心代码格式问题了。不过，Prettier比较‘自以为是’，只提供了很少一部分配置项让用户自定义。当然，个人认为，Prettier大部分规则都是友好的。Eslint，并不会自动格式化代码,而是提供一些警告，修复警告。比如，定义了一个变量，但是没有使用该变量，Eslint会警告已声明变量却从未读取。与Prettier相比，Eslint具有高度可配置性，因为它没有预先配置的规则，当你安装了Eslint，就可以配置规则或者使用共享的规则。通常，Prettier会结合Eslint使用。

## ESLint
为了方便、快捷使用Eslint以及统一Eslint规则，请使用`eslint-config-7xn`共享配置。首先，使用如下命令安装`eslint-config-7xn`,`ESLint`等相关依赖：
```
npm install --save-dev eslint-config-7xn eslint-plugin-compat@3.x eslint-config-prettier@6.x @typescript-eslint/eslint-plugin@1.x @typescript-eslint/parser@1.x babel-eslint@10.x eslint@6.x eslint-plugin-flowtype@3.x eslint-plugin-import@2.x eslint-plugin-jsx-a11y@6.x eslint-plugin-react@7.x eslint-plugin-react-hooks@1.x
```
然后在项目根目录新建`.eslintrc`文件，内容如下：
```
{
  "extends": "7xn"
}

```

在`package.json`添加执行脚本
```
"scripts": {
  "lint": "eslint .",
  "lint:fix": "eslint . --fix"
},
```
运行`npm run lint`检查代码或者`npm run lint:fix`修复Eslint错误。  

当然，或许你也希望你的编辑器可以检测eslint错误并修复，以VS Code为例。  
安装扩展[VS Code ESLint extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)，当你安装了扩展之后，默认会开启Eslint检测，如果需要保存时自动修复，在文
件->首选项->设置->用户 JSON 文件里添加如下配置：
```javascript{1}
"eslint.autoFixOnSave": true
```

**如果和Prettier在VS Code中一起使用，可能会和prettier冲突，不建议开启保存自动修复。**

## Prettier
首先，安装prettier依赖：
```
npm install prettier --save-dev
```

然后在项目根目录新建文件`.prettierrc`文件，添加如下内容：
```
{
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100,
  "overrides": [
    {
      "files": ".prettierrc",
      "options": { "parser": "json" }
    }
  ]
}

```

与Eslint结合使用的话，理论上需要使用`eslint-config-prettier`把冲突的规则关掉，由于`eslint-config-7xn`已经使用`eslint-config-prettier`把可能冲突的规则关掉，所以不需要在项目根目录的`.eslintrc`文件添加相关配置。 

在`package.json`添加执行脚本
```
"scripts": {
  "prettier": "prettier --write .",
},
```
运行`npm run prettier`格式化代码。  

那么在VS Code中如何使用呢？安装扩展[Prettier formatter for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)，当你安装了扩展之后，
在文件->首选项->设置->用户 JSON 文件里添加如下配置：
```javascript{1}
"editor.formatOnSave": true
```

> 个人建议把ESlint、Prettier在VS Code的配置放到工作区或者单独的项目中，避免影响不需要Eslint、Prettier的项目。

## 自动化

> 注意：Git版本需`>=2.13.0`，否则 husky 创建钩子会失效，详情查看[这里](https://github.com/typicode/husky#enoent-error-node_moduleshuskygithooks)

为确保代码在提交到仓库之前没有ESlint错误，并且格式化了，你可能需要这样做。安装`husky`、`lint-staged`依赖：
```
npm install husky lint-staged --save-dev
```

在package.json添加脚本代码示例：
```
"scripts": {
  "lint": "eslint --ext .js src",
  "lint:fix": "eslint --fix --ext .js src",
  "lint-staged": "lint-staged",
  "lint-staged:js": "eslint --ext .js",
  "prettier": "prettier --write ./src/**/*"
},
"lint-staged": {
  "**/*.{js,jsx,less}": [
    "prettier --write",
    "git add"
  ],
  "**/*.{js,jsx}": "npm run lint-staged:js"
},
"husky": {
  "hooks": {
    "pre-commit": "npm run lint-staged"
  }
}
```
在提交代码的时候会触发钩子函数执行命令，把暂存区的代码格式化并检查ESlint错误。  

## Editorconfig
在协作开发一个项目的时候，开发者使用的编辑器可能不一样（e.g. VS Code, WebStore），使用的操作系统可能也不一样（e.g. Windows, Mac）。比如有的编辑器默认缩进使用空格，有的使用Tab。Windows换行符是CRLF，Mac是LF。[EditorConfig](http://editorconfig.org)有助于为跨越各种编辑器和IDE的同一项目的多个开发人员维护一致的编码样式。

在项目根目录新建文件`.editorconfig`，添加如下内容：
```
# http://editorconfig.org
root = true

[*]
indent_style = space
indent_size = 2
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.md]
trim_trailing_whitespace = false

[Makefile]
indent_style = tab

```

有些编辑器默认支持EditorConfig，有的则不支持，比如VS Code，需要自行安装插件。
VS Code安装插件[EditorConfig for Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)开启即可。

胡说八道~
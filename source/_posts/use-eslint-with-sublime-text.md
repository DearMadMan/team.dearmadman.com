title: Sublime Text 中配置 ESLint
date: 2016-06-27 16:02:40
tags: [javascript]
---

# ESLint

ESLint 是一个有效的代码质量控制工具，它可以根据预先制定的代码规范来避免低级代码错误的出现，以及保证代码样式风格的统一。

## 安装

你可以使用 npm 来安装 ESLint:

```bash
npm install eslint -g
```

## 用法

如果你首次使用 ESLint，那么你需要先设置一个配置文件，你可以在项目根目录下使用 `--init` 选项来生成:

```bash
eslint --init
```

如果项目根目录下没有 `package.json` 文件，它会提示你先使用 `npm init` 来初始化一个 `package.json` 文件。

`eslint --init` 会提示你选择使用的代码样式，你可以根据需求进行选择，这里推荐如下选择：
- Use a popular style guide
- Standard
- JavaScript

在这个过程中 eslint 会自主的进行 npm install 操作来安装相关依赖，在安装完成之后请注意是否有 `UNMET PEER DEPENDENCY` 项的依赖，这表示 NPM 无法自动的进行这个依赖的安装，你需要手动的去进行安装：

``` bash
# ├── UNMET PEER DEPENDENCY eslint-plugin-promise@^1.0.8
npm install eslint-plugin-promise --save-dev
```

## 集成 Sublime Text

在 Sublime Text 中你需要安装两个插件：
- SublimeLinter
- SublimeLinter-contrib-eslint

然后通过 `Preferences->Package Settings->SublimeLinter->Settings - User` 进行集成：

```json
{
    "user": {
        "debug": true, # 开启 debug 选项
        "delay": 0.25,
        "error_color": "D02000",
        "gutter_theme": "Packages/SublimeLinter/gutter-themes/Default/Default.gutter-theme",
        "gutter_theme_excludes": [],
        "lint_mode": "background",
        "linters": {
            "eslint": {
                "@disable": false,
                "args": [],
                "excludes": []
            },
            "jshint": {
                "@disable": false,
                "args": [],
                "excludes": []
            },
            "php": {
                "@disable": false,
                "args": [],
                "excludes": []
            }
        },
        "mark_style": "outline",
        "no_column_highlights_line": false,
        "passive_warnings": false,
        "paths": {
            "linux": [],
            "osx": [
                "/Users/wang/.nvm/versions/node/v5.0.0/bin" # 设置 node 路径
            ],
            "windows": []
        },
        "python_paths": {
            "linux": [],
            "osx": [],
            "windows": []
        },
        "rc_search_limit": 3,
        "shell_timeout": 10,
        "show_errors_on_save": false,
        "show_marks_in_minimap": true,
        "syntax_map": {
            "html (django)": "html",
            "html (rails)": "html",
            "html 5": "html",
            "javascript (babel)": "javascript",
            "magicpython": "python",
            "php": "html",
            "python django": "python",
            "pythonimproved": "python"
        },
        "warning_color": "DDB700",
        "wrap_find": true
    }
}
```

到这里，集成完成，如果你项目中的 JavaScript 代码风格不符合 [JavaScript Standard Style](https://github.com/feross/standard) 的话，Sublime Text 会给出异常提示。

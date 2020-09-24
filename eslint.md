# eslint 
```
{
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    },
    "editor.rulers": [80, 120],
    "workbench.iconTheme": "vscode-icons",
    "eslint.alwaysShowStatus": true,
    "eslint.options": {
      "extensions": [".html", ".js", ".vue", ".jsx"]
    },
    "vetur.format.defaultFormatter.html": "js-beautify-html",
    "vetur.format.defaultFormatter.js": "prettier",
    "vetur.format.defaultFormatter.css": "prettier",
    "vetur.format.defaultFormatterOptions": {
      "js-beautify-html": {
        "wrap_attributes": "force-expand-multiline"
      },
      "prettier": {
        "singleQuote": true,
        "trailingComma": "none",
        "semi": false,
        "printWidth":120
      }
    },
    "[vue]": {
      "editor.defaultFormatter": "octref.vetur",
      "editor.formatOnSave": true
    },
    "[javascript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode",
    }
  }
```
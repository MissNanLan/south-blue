```
yarn add husky @commitlint/config-angular
yarn husky add .husky/commit-msg 'yarn commitlint --edit $1'
echo "module.exports = {extends: ['@commitlint/config-angular']};" > commitlint.config.js
```

但是最后一句具体的含义没有看懂

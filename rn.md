0.32.1
出现 Strict mode does not allow function declaration in a lexically nested statement.
解决方案 :
edit Libraries/Utilities/UIManager.js:68 with:
改成
var normalizePrefix = function(moduleName: string): string => {



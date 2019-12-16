---
categories:
- web前端
tags:
- vue
- element
---

###`\n`全部替换为`<br/>`

一段有换行的文字在`this.$message`中显示时格式会出现混乱，换行符不起作用，需要先将文字段落中的`\n`全部替换`<br/>`，再在`this.$message`中添加`dangerouslyUseHTMLString: true`属性


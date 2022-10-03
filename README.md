# JSON 是什么
综合 [MDN](https://developer.mozilla.org/zh-CN/docs/Glossary/JSON) 和[维基百科](https://zh.wikipedia.org/wiki/JSON)上的定义，JSON 是一种轻量级数据交换格式，其内容由属性和值所组成，易于阅读和处理。
JSON 可以表示**数字、布尔值、字符串、null、数组**（有序序列），以及由这些值组成的**对象**（字符串与值的映射）。JSON 不支持复杂的数据类型（函数、正则表达式、日期等）。
一个有效的JSON文档的根节点**必须是一个对象或一个数组**。

# JSON.parse() 是什么
JavaScript 语言自带 JSON 对象，包含两个方法：
JSON.parse() 用于解析 JSON 字符串并返回对应对象/值的方法；
JSON.stringify() 用于将对象/值转换为 JSON 字符串的方法。

# JSON.parse() 用法举例
1. **对象** JSON 字符串：
```
{
    "a1": [1, "2", -3, 4.5, {
        "k0": "v0"
    }],
    "obj": {
        "k1": "v1",
        "k2": ["10", -20]
    }
}
```
JSON.parse() 后结果：
![对象 JSON](https://raw.githubusercontent.com/jannahuang/blog/main/pictures/json.parse1.png)

2. **数组** JSON 字符串：
```
[
    true, null, {
        "obj": {
            "k1": "v1",
            "k2": [10, "-20"]
        }
    }, [1, "2", {
        "a1": {
            "k3": "v3"
        }
    }]
]
```
JSON.parse() 后结果：
![数组 JSON](https://raw.githubusercontent.com/jannahuang/blog/main/pictures/json.parse2.png)

# 步骤拆解
从上述举例可见，实现一个 json-parser，首先需要逐字读取字符串，去除空格和换行，提取并保存有效的词（token）；然后将词（token）按顺序和词法规则生成新的对象或数组。
拆解思维导图如下：
![json-parser 步骤拆解](https://raw.githubusercontent.com/jannahuang/blog/main/pictures/json-parser%E6%AD%A5%E9%AA%A4%E6%8B%86%E8%A7%A3.png)
接下来按步骤实现代码。

# 代码实现
## 提取 token
### 实现 jsonTokens()
```javascript
const jsonTokens = (s) => {
    let r = []
    for (let i = 0; i < s.length; i++) {
        let e = s[i]
        if (isSymbol(e)) {
            r.push(e)
        } else if (e === '"' || e === "'") {
            // 跳过前面的引号
            let t = stringEnd(s, i + 1)
            r.push(t[0])
            // 跳过后面的引号
            i += t[1] + 1
        } else if (isNumber(e)) {
            let t = numberEnd(s, i)
            r.push(t[0])
            i += t[1] - 1
        } else if (isLetter(e)) {
            // log('in')
            // 跳过前面的引号
            let t = stringEnd(s, i)
            // log('t in', t)
            r.push(t[0])
            // 跳过后面的引号
            i += t[1]
        } else if (e === ' ') {
            // 空格跳过，不处理
        }
    }
    return r
}
```
## 生成值
### 实现 parseObject()
```javascript
// 全局变量，记录位移
let globalOffset_obj = 0
let globalOffset_arr = 0

const parseObject = (tokenList) => {
    let object = {}
    for (let i = 0; i < tokenList.length; i++) {
        let t0 = tokenList[i]
        let t1 = tokenList[i + 1]
        let t2 = tokenList[i + 2]
        if (t0 === '}') {
            // i 初始为 0，obj 还有键值中间的冒号，故 +2
            globalOffset_obj = i + 2
            return object
        } else if (t1 === ':' && t2 === '{') {
            // 跳过 t0, t1, t2，故 +3
            let s = tokenList.slice(i + 2)
            r = parseObject(s)
            object[t0] = r
            i += globalOffset_obj
        } else if (t1 === ':' && t2 === '[') {
            // 跳过 t0, t1, t2，故 +3
            let s = tokenList.slice(i + 3)
            r = parseArray(s)
            object[t0] = r
            i += globalOffset_arr + 1
        } else if (t1 === ':' && t2 !== '{' && t2 !== '[') {
            object[t0] = t2
        }
    }
}

```

### 实现 parseArray()
 ```javascript
const parseArray = (tokenList) => {
    let array = []
    for (let i = 0; i < tokenList.length; i++) {
        let t = tokenList[i]
        if (t === ']') {
            // i 初始为 0，因此 +1
            globalOffset_arr = i + 1
            return array
        } else if (t === '{') {
            // 跳过 t，故 +1
            let s = tokenList.slice(i + 1)
            let r = parseObject(s)
            array.push(r)
            i += globalOffset_obj - 1
        } else if (t === '[') {
            // 跳过 t，故 +1
            let s = tokenList.slice(i + 1)
            let r = parseArray(s)
            array.push(r)
            i += globalOffset_arr
        } else if (t === ',') {
            // 逗号跳过，不处理
        } else {
            array.push(t)
        }
    }
}

```

### 辅助函数
 ```javascript

const isSymbol = (t) => {
    let symbol = '[]{}:,'
    return symbol.includes(t)
}

const isNumber = (t) => {
    let number = '-.0123456789'
    return number.includes(t)
}

const isLetter = (t) => {
    let letter = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
    return letter.includes(t)
}

const stringEnd = (s, i) => {
    s = s.slice(i)
    let str = ''
    // 处理转义字符
    let map = {
        'b': '\b',
        'f': '\f',
        'n': '\n',
        'r': '\r',
        'v': '\v',
        't': '\t',
        "'": "'",
        '"': '"',
        '\\': '\\',
    }
    // 处理常见关键词
    let keyword = {
        'true': true,
        'false': false,
        'null': null,
    }
    for (let i = 0; i < s.length; i++) {
        let t = s[i]
        if (isLetter(t) || isNumber(t) || s[i] === '/') {
            str += t
        } else if (t === '\\') {
            // 处理转义字符，先获取反斜杠后的字母 s[i + 1]，再从 map 里取对应值
            let o = map[s[i + 1]]
            if (o) {
                str += o
                i += 1
            }
        } else if (t === '"' || t === "'") {
            // 以数组格式返回 str 和 i，便于后续计算
            return [str, i]
        } else if (t === '\n') {
            str += t
        } else {
            if (Object.keys(keyword).includes(str)) {
                return [keyword[str], i]
            }
        }
    }
    return [str, str.length]
}

const numberEnd = (s, i) => {
    s = s.slice(i)
    let num = ''
    for (let i = 0; i < s.length; i++) {
        let t = s[i]
        if (isNumber(t)) {
            num += t
        } else {
            // 以数组格式返回 num 和 i，便于后续计算
            return [Number(num), i]
        }
    }
    return [Number(num), i]
}

```

### 主函数
 ```javascript

const jsonParser = (s) => {
    let tokenList = jsonTokens(s)
    let result
    if (tokenList[0] === '{') {
        result = parseObject(tokenList.slice(1))
    } else if (tokenList[0] === '[') {
        result = parseArray(tokenList.slice(1))
    }
    return result
}
```

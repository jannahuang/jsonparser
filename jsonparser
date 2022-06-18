const log = console.log.bind(console)

const isArray = o => {
    return Array.isArray(o)
}

const isObject = o => {
    return Object.prototype.toString.call(o) === '[object Object]'
}

const equals = (a, b) => {
    if (isArray(a) && isArray(b)) {
        if (a.length !== b.length) {
            log('测试用例长度不相等')
            return false
        }
        for (let i = 0; i < a.length; i++) {
            let a1 = a[i]
            let b1 = b[i]
            if (!equals(a1, b1)) {
                return false
            }
        }
        return true
    } else if (isObject(a) && isObject(b)) {
        let keys1 = Object.keys(a)
        let keys2 = Object.keys(b)
        if (keys1.length !== keys2.length) {
            return false
        }
        for (let i = 0; i < keys1.length; i++) {
            let k1 = keys1[i]
            let k2 = keys2[i]
            if (!equals(a[k1], b[k2])) {
                return false
            }
        }
        return true
    } else {
        return a === b
    }
}

const ensure = (a, message) => {
    if (!a) {
        // let s1 = JSON.stringify(a)
        // let s2 = JSON.stringify(b)
        log(`${message}, 测试失败`)
    } else {
        log(`${message}, 测试成功`)
    }
}

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

const jsonTokens = (s) => {
    let res = []
    for (let i = 0; i < s.length; i++) {
        let t = s[i]
        // log('t',`(${t})`)
        if (isSymbol(t)) {
            res.push(t)
        } else if (t === '"' || t === "'") {
            // 将当前位置到末尾的字符串传给 stringEnd() 函数，提取 token
            let r = stringEnd(s, i + 1)
            res.push(r[0])
            // 跳过后面的引号
            i += r[1] + 1
        } else if (isNumber(t)) {
            let r = numberEnd(s, i)
            res.push(r[0])
            // 前移一位以读取逗号
            i += r[1] - 1
        } else if (isLetter(t)) {
            let r = stringEnd(s, i)
            res.push(r[0])
            // 跳过后面的逗号
            i += r[1] + 1
        } else if (t === ' ') {
            // 空格跳过，不处理
        }
    }
    return res
}

// 全局变量，记录位移
let globalOffset_obj = 0
let globalOffset_arr = 0

const parseObject = (tokenList) => {
    let object = {}
    for (let i = 0; i < tokenList.length; i++) {
        let t0 = tokenList[i]
        let t1 = tokenList[i + 1]
        let t2 = tokenList[i + 2]
        log('obj__t0', `(${t0})`)
        if (t0 === '}') {
            // 算上两边括号 +2
            globalOffset_obj = i + 2
            log('globalOffset_obj',globalOffset_obj)
            return object
        } else if (t1 === ':' && t2 === '{') {
            // 带上 { 递归调用，跳过 t0, t1，故 +2
            let s = tokenList.slice(i + 2)
            log('obj__s1',s)
            r = parseObject(s)
            object[t0] = r
            i += globalOffset_obj
        } else if (t1 === ':' && t2 === '[') {
            // 不带上 [ ，跳过 t0, t1, t2，故 +3
            let s = tokenList.slice(i + 3)
            log('obj__s2',s)
            r = parseArray(s)
            object[t0] = r
            i += globalOffset_arr
        } else if (t1 === ':' && t2 !== '{' && t2 !== '[') {
            object[t0] = t2
        }
    }
}

const parseArray = (tokenList) => {
    let array = []
    for (let i = 0; i < tokenList.length; i++) {
        let t = tokenList[i]
        log('arr__t', `(${t})`)
        if (t === ']') {
            // 算上两边括号 +2
            globalOffset_arr = i + 2
            log('globalOffset_arr',globalOffset_arr)
            return array
        } else if (t === '{') {
            // parseArray 中调用 parseObject，不带上 {，但 parseObject 自己调用要带上 {
            // 跳过 t，故 +1
            let s = tokenList.slice(i + 1)
            log('arr__s1',s)
            let r = parseObject(s)
            array.push(r)
            i += globalOffset_obj - 1
        } else if (t === '[') {
            // 跳过 t，故 +1
            let s = tokenList.slice(i + 1)
            log('arr__s2',s)
            let r = parseArray(s)
            array.push(r)
            i += globalOffset_arr - 1
        } else if (t === ',') {
            // 逗号跳过，不处理
        } else {
            array.push(t)
        }
    }
}

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

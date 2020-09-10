# DSA - 哈希表

## 哈希函数

哈希函数可以将一个任意长度的数据通过某种计算后，生成另一个长度固定的值，这个值就叫**哈希**。

### 哈希函数的特点：

1. 计算过程很快。
2. 相同的输入总会产生相同的输出。不同的输入也可能会产生相同的输出，这也可以解释为什么哈希函数会有第 4 个特性。
3. 不管输入的数据长度是多少，输出的哈希值长度都是固定的。
4. 操作是单向的，我们可以输入一个字符串得到一个哈希值，但却不能从得到的哈希值反推出原来的字符串(哈希函数的操作是有损的)。这是一个哈希函数常见的特性，对于数据加密安全来说很重要，但这并不是哈希函数必需的特性。

### 常见的哈希算法

-   [MD5](https://en.wikipedia.org/wiki/MD5)
-   [SHA-1](https://en.wikipedia.org/wiki/SHA-1)
-   [SHA-2](https://en.wikipedia.org/wiki/SHA-2)
-   [NTLM](https://it.wikipedia.org/wiki/NTLM)

### 哈希值的使用场景

-   哈希表
-   加密
-   安全

**具体的例子**

1. 在一个开源项目中，有一个文件保存了基于整个项目生成的哈希值(签名)。当你下载了整个项目之后，可以自己重新生成一个哈希值，对比两者就可以知道你下载的文件是否有被篡改。
2. 密码加密。

### 哈希冲突

前面提过，哈希值的长度是固定的，也就是说，可用的哈希值是有限的，这会导致的问题就是，两个不同的数据经过哈希函数的处理后生成了一样的哈希值，这就是**哈希冲突**。

### salt

salt 是指在进行 `hash(str)` 操作之前，先根据一个随机数(seed)来修改 `str`，再生成哈希值。

这样做的好处是什么？举个例子，上面提到哈希函数常用于加密密码，如果黑客拿到了这些密码哈希值，虽然他们不能根据哈希值倒推出密码，但由于哈希函数有一个特性，相同的输入总会产生相同的输出，黑客只需要找一些常用的密码，对它们进行哈希处理(rainbow table)，然后再与数据库中的哈希密码值比较，还是有很大可能会得到一些密码的。

但如果在对密码进行哈希加密之前，先用一个随机数来对密码进行修改，由于黑客不知道用于修改密码的随机数，也就不能破解密码。

这样做还可以有些防御 DOS 攻击。

> DOS(Denial Of Service) 攻击是指攻击者发起大量请求，把服务器的资源耗尽，使得服务器无法响应正常用户的请求。

## 哈希表

哈希表是一种数据结构，是一个键值对集合，其中哈希表的键必须是 hashable 的值。

> hashable value must be immutable value.

### 解决哈希冲突的两种方法

-   open addressing
-   separate chaining

### 哈希表实现：separate chaining

```js
/**
 * 一个简单的哈希函数，返回字符串前三位 ASCII 码的和
 * @param {string} str
 */
function hash(str) {
    return [].reduce.call(
        str.slice(0, 3),
        (res, s) => res + s.charCodeAt(0),
        0,
    );
}

class HashTable {
    constructor(elements) {
        this.buketSize = elements.length;
        this.bukets = Array(this.buketSize)
            .fill(null)
            .map(() => []);

        for (const [k, v] of elements) {
            this.put(k, v);
        }
    }
    _getBuket(key) {
        const index = hash(key) % this.buketSize;
        return this.bukets[index];
    }
    put(key, value) {
        const buket = this._getBuket(key);
        for (const kv of buket) {
            if (kv[0] === key) {
                kv[1] = value;
                return;
            }
        }
        buket.push([key, value]);
    }
    get(key) {
        const buket = this._getBuket(key);
        for (const [k, v] of buket) {
            if (k === key) return v;
        }
    }
}

const elements = [
    ['France', 'Paris'],
    ['United States', 'Washington D.C.'],
    ['Italy', 'Rome'],
    ['Canada', 'Ottawa'],
];

const hashtable = new HashTable(elements);
console.log(hashtable.get('Italy')); // Rome
hashtable.put('Canada', 'yoyo');
console.log(hashtable.bukets);

// [
//     [['United States', 'Washington D.C.']],
//     [['France', 'Paris']],
//     [
//         ['Italy', 'Rome'],
//         ['Canada', 'yoyo'],
//     ],
//     [],
// ];
```

### 哈希表实现：open addressing

```js
/**
 * 一个简单的哈希函数，返回字符串前三位 ASCII 码的和
 * @param {string} str
 */
function hash(str) {
    return [].reduce.call(
        str.slice(0, 3),
        (res, s) => res + s.charCodeAt(0),
        0,
    );
}

class HashTable {
    constructor(elements) {
        this.buketSize = elements.length;
        this.bukets = Array(this.buketSize).fill(null);

        for (const [k, v] of elements) {
            this.put(k, v);
        }
    }
    put(key, value) {
        const start = hash(key) % this.buketSize;
        let index = start;

        while (this.bukets[index] !== null) {
            // 已存在的 key，覆盖
            if (this.bukets[index][0] === key) break;

            index = (index + 1) % this.buketSize;

            // bukets 已经满了，扩展空间
            if (index === start) {
                this.bukets.push([key, value]);
                this.buketSize++;
                return;
            }
        }
        this.bukets[index] = [key, value];
    }
    get(key) {
        const start = hash(key) % this.buketSize;
        let index = start;
        while (this.bukets[index][0] !== key) {
            index = (index + 1) % this.buketSize;

            // key 不存在
            if (index === start) return;
        }
        return this.bukets[index][1];
    }
}
```

---
title: JS面试中常见的算法题
thumbnail: JS面试中常见的算法题/20191014_0.jpg
tags:
  - javaScript
  - 面试
  - 算法
categories:
  - javaScript
  - 面试
  - 算法
date: 2019-10-14 21:50:41
updated: 2019-10-14 21:50:41
layout:
comments:
---
***js除了基础知识以外，算法也是挺重要的。因此特意整理了一些常见的算法题，希望大家有帮助！***

<!-- more -->

## <font>***1、验证一个数是否是素数***</font>
---

1. 如果这个数是 2 或 3，一定是素数；
2. 如果是偶数，一定不是素数；
3. 如果这个数不能被3~它的平方根中的任一数整除，m必定是素数。而且除数可以每次递增（排除偶数）。

```javascript
function isPrime(num){
    if (num === 2 || num === 3) {
        return true;
    };
    if (num % 2 === 0) {
        return false;
    };
    let divisor = 3,limit = Math.sqrt(num);
    while(limit >= divisor){
        if (num % divisor === 0) {
            return false;
        }
        else {
            divisor += 2;
        }
    }
    return true;
}
console.log(isPrime(30));  // false
```

## <font>***2、斐波那契***</font>
---

最简单的做法：递归。

```javascript
function fibonacci(n){
    if (n <= 0) {
        return 0;
    }
    if (n == 0) {
        return 1;
    }
    return fibonacci(n-1) + fibonacci(n-2);
}
```

但是递归会有严重的效率问题。比如想要求得f(10)，首先需要求f(9)和f(8)。同样，想求f(9)，首先需要f(8)和f(7)…这样就有很多重复值，计算量也很大。

改进：从下往上计算，首先根据f(0)和f(1)计算出f(2)，再根据f(1)和f(2)计算出f(3)……以此类推就可以计算出第n项。时间复杂度O(n)。

```javascript
function fibonacci(n){
    let ori = [0,1];
    if (n < 2) {
        return ori[n];
    };
    let fiboOne = 1,fiboTwo = 0,fiboSum = 0;
    for (let i = 2; i <= n; i++) {
        fiboSum = fiboOne + fiboTwo;
        fiboTwo = fiboOne;
        fiboOne = fiboSum;
    }
    return fiboSum;
}
console.log(fibonacci(5));
```

## <font>***3、求最大公约数***</font>
---

除数 在a和b的范围内，如果同时a和b处以除数的余等于0，就将此时的除数赋值给res；除数自增，不断循环上面的计算，更新res。

```javascript
function greatestCommonDivisor(a, b){
    let divisor = 2,res = 1;
    if (a < 2 || b < 2) {
        return 1;
    };
    while(a >= divisor && b >= divisor){
        if (a%divisor === 0 && b%divisor === 0) {
            res = divisor;
        }
        divisor++;
    }
    return res;
};
console.log(greatestCommonDivisor(8, 4)); // 4
console.log(greatestCommonDivisor(69, 169)); // 1
```

***解法2***

```javascript
function greatestCommonDivisor(a,b){
    if (b === 0) {
        return a;
    } else {
        return greatestCommonDivisor(b,a%b);
    }
};
```

## <font>***4、数组去重***</font>
---

对原数组进行遍历获取arr[i]的值 j；对应到辅助数组 exits 的位置 j 的值，如果没有，则证明arr[i] 的值没有重复，此时将 值j 存入res数组，并将辅助数组 j 位置的值置为true。最后返回res数组。

```javascript
function removeDuplicate(arr){
    if (arr === null || arr.length < 2) {
        return arr;
    };
    let res = [],exits = [];
    for(let i = 0; i < arr.length; i++){
        let j = arr[i];
        while( !exits[j] ){
            res.push(arr[i]);
            exits[j] = true;
        }
    }
    return res;
}
console.log(removeDuplicate([1,3,3,3,1,5,6,7,8,1]))  // [1,3,5,6,7,8]
```

## <font>***5、删除重复的字符***</font>
---

这一题的解法和上一题类似。

```javascript
function removeDuplicateChar(str){
    if (!str || str.length < 2 || typeof str != "string") {
        return;
    };
    let charArr = [],res = [];
    for(let i = 0; i < str.length; i++){
        let c = str[i];
        if(charArr[c]){
            charArr[c]++;
        }
        else{
            charArr[c] = 1;
        }
    }
    for(let j in charArr){
        if (charArr[j] === 1) {
            res.push(j);
        }
    }
    return res.join("");
}
console.log(removeDuplicateChar("Learn more javascript dude"));
```

## <font>***6、排序两个已经排好序的数组***</font>
---

如果 b数组已经遍历完，a数组还有值 或 a[i] 的值 小于等于 b[i] 的值，则将 a[i] 添加进数组res，并 i++；
如果不是上面的情况，则将 b[i] 添加进数组res，并 i++；

```javascript
function mergeSortedArr(a,b){
    if (!a || !b) {
        return;
    };
    let aEle = a[0],bEle = b[0],i = 1,j = 1,res = [];
    while(aEle || bEle){
        if ((aEle && !bEle) || aEle <= bEle) {
            res.push(aEle);
            aEle = a[i++];
        }
        else{
            res.push(bEle);
            bEle = b[j++];
        }
    }
    return res;
}
console.log(mergeSortedArr([2,5,6,9], [1,2,3,29]))  // [1,2,2,3,5,6,9,29]
```

## <font>***7、字符串反向***</font>
---

```javascript
function reverse(str){
    let resStr = "";
    for(let i = str.length-1; i >= 0; i--){
        resStr += str[i];
    }
    return resStr;
}
console.log(reverse("ABCDEFG"));
function reverse2(str){
    if (!str || str.length < 2 || typeof str != "string") {
        return str;
    };
    let res = [];
    for(let i = str.length-1; i >= 0; i--){
        res.push(str[i]);
    }
    return res.join("");
}
console.log(reverse2("Hello"));
```

将函数添加到String.prototype

```javascript
String.prototype.reverse3 = function(){
    if (!this || this.length < 2) {
        return;
    };
    let res = [];
    for(let i = this.length-1; i >= 0; i--){
        res.push(this[i]);
    }
    return res.join("");
}
console.log("abcdefg".reverse3());
```

## <font>***8、字符串原位反转***</font>
---

例如：将“I am the good boy”反转变为 “I ma eht doog yob”。

提示：使用数组和字符串方法。

```javascript
function reverseInPlace(str){
    return str.split(' ').reverse().join(' ').split('').reverse().join('');
}
console.log(reverseInPlace('I am the good boy'));
```

## <font>***9、判断是否是回文***</font>
---

```javascript
function isPalindrome(str){
    if (!str || str.length < 2) {
        return;
    }
    for(let i = 0; i < str.length/2; i++){
        if (str[i] !== str[str.length-1-i]) {
            return false;
        }
    }
    return true;
}
console.log(isPalindrome("madama"))
```

## <font>***10、判断数组中是否有两数之和***</font>
---

eg：在一个未排序的数组中找出是否有任意两数之和等于给定的数。

给出一个数组[6,4,3,2,1,7]和一个数9，判断数组里是否有任意两数之和为9。

1. eg：在一个未排序的数组中找出是否有任意两数之和等于给定的数。
2. 给出一个数组[6,4,3,2,1,7]和一个数9，判断数组里是否有任意两数之和为9。

```javascript
function sumFind(arr,num){
    if (!arr || arr.length < 2) {
        return;
    };
    let differ = {};
    for(let i = 0; i < arr.length; i++){
        let subStract = num - arr[i];
        if (differ[subStract]) {
            return true;
        }
        else{
            differ[arr[i]] = true;
        }
    }
    return false;
}
console.log(sumFind([6,4,3,2,1,7], 9));  // true
```

## <font>***11、连字符转成驼峰***</font>
---

如：get-element-by-id 转为 getElementById

```javascript
let str = 'get-element-by-id';
let arr = str.split('-');
for(let i=1; i<arr.length; i++){
  arr[i] = arr[i].charAt(0).toUpperCase() + arr[i].substring(1);
}
console.log(arr.join(''));   // getElementById
```

## <font>***12、加油站问题-贪心算法***</font>
---

一辆汽车加满油后可行驶n公里。旅途中有若干个加油站。设计一个有效算法，指出应在哪些加油站停靠加油，使沿途加油次数最少。对于给定的n(n <= 5000)和k(k <= 1000)个加油站位置，编程计算最少加油次数。并证明算法能产生一个最优解。

***要求：***

**输入：**第一行有2个正整数n和k，表示汽车加满油后可行驶n公里，且旅途中有k个加油站。接下来的1 行中，有k+1 个整数，表示第k个加油站与第k-1 个加油站之间的距离。第0 个加油站表示出发地，汽车已加满油。第k+1 个加油站表示目的地。

**输出：**输出编程计算出的最少加油次数。如果无法到达目的地，则输出”NoSolution”。

```javascript
function greedy(n, k, arr){  // n:加满可以行驶的公里数; k:加油站数量; arr:每个加油站之间的距离数组
    if (n == 0 || k == 0 || arr.length == 0 || arr[0] > n) {
        return "No Solution!";  // arr[0] > n ：如果第一个加油站距离太远，也无法到达
    };
    let res = 0, distance = 0;  // res：加油次数；distance：已行驶距离
    for(let i = 0; i <= k; i++){
        distance += arr[i];
        if (distance > n) {  // 已行驶距离 > 加满可以行驶的公里数
            if(arr[i] > n){  // 如果目前加油站和前一个加油站的距离 > 加满可以行驶的公里数，则无法到达
                return "No Solution!";
            };
            // 可以在上一个加油站加油，行驶到目前的加油站i：
            distance = arr[i];
            res++;  // 加油次数+1
        }
    }
    return res;
}
let arr = [1,2,3,4,5,1,6,6];
console.log(greedy(7,7,arr))  // 4
```

## <font>***13、用正则实现trim() 清除字符串两端空格***</font>
---

```javascript
String.prototype.trim1 = function(){
    // return this.replace(/\s*/g,"");  // 清除所有空格
    return this.replace(/(^\s*)|(\s*$)/g,"");  // 清除字符串前后的空格
};
console.log("  hello word ".trim1())  // "hello word"
```

## <font>***14、将数字12345678转化成RMB形式：12,345,678***</font>
---

思路：将字符串切割成数组再反转，遍历数组，加入辅助数组，当数组长度为3的倍数，再向辅助数组加入 ","。

```javascript
function RMB(str){
    let arr = str.split("").reverse();
    let res = [];
    for(let i = 0; i < arr.length; i++){
        res.push(arr[i]);
        if ((i + 1) % 3 === 0) {
            res.push(",");
        }
    }
    return res.reverse().join("");
}
console.log(RMB("12345678"))
```

## <font>***15、删除相邻相同的字符串***</font>
---

```javascript
function delSrt(str){
    let res = [], nowStr;
    for(let i = 0; i < str.length; i ++){
        if (str.charAt(i) != nowStr) {
            res.push(str.charAt(i));
            nowStr = str.charAt(i);
        }
    }
    return res.join("");
}
console.log(delSrt("aabcc11"))
```
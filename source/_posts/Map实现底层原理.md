---
title: Map实现底层原理
top: false
cover: false
toc: true
hidden: false
mathjax: true
typora-root-url: Map实现底层原理
date: 2024-05-11 17:18:18
password:
summary:
tags: [前端-知识点, interview]
categories:
---

## es6 Map()

#### Map() 常见方法

```js
const map = new Map()
map.set(1, 'a')
map.set(2, 'b')
map.get(1)
map.has(1)
map.delete(2)
map.size()

```

#### Map() 底层原理
- 哈希概念
    - 实质为通过一些方程式将不确定的值转化为特定的值
- 桶的概念
    - 就是计数的升级版，规定一个桶的个数，每个桶里面相当于一个{}，且每个对象里面有个next:{next: null}
- 链表的概念
    - 内存内部的一种存储方式，node1.next

#### Map()自我实现
```js
  function MyMap() {
        this.tongLen = 8; // 设置tong的长度
        this.initStore();
      }

      MyMap.prototype.initStore = function () {
        this.storeList = new Array(this.tongLen);
        for (let index = 0; index < this.tongLen; index++) {
          this.storeList[index] = {
            next: null
          };
        }
      };
      MyMap.prototype.hash = function (key) {
        return key % this.storeList.length;
      };
      MyMap.prototype.set = function (key, val) {
        let index = this.hash(key);
        let queue = this.storeList[index];
        while (queue.next) {
          if (queue.next.index == key) return (queue.next.value = val);
          else queue = queue.next;
        }
        queue.next = {
          key: key,
          value: val,
          next: null,
        };
      };
      MyMap.prototype.get = function (key) {
    
        let index = this.hash(key);
        let queue = this.storeList[index];
        queue = queue.next
        while (queue) {
          if (queue.key == key) return queue.value;
          else queue = queue.next;
        }
        return undefined;
      };
      MyMap.prototype.delete = function (key) {
        let index = this.hash(key);
        let queue = this.storeList[index];
        
        while (queue.next) {
          if (queue.next.key == key) {
            queue.next = queue.next.next;
            return "delete!!!";
          } else {
            queue = queue.next;
          }
        }
        return "不存在";
      };
      MyMap.prototype.size = function (key) {
        let index = this.hash(key);
        let queue = this.storeList[index];
        let size = 0
        while (queue.next) {
          if (queue.next.key) {
            return size++
          } else {
            queue = queue.next;
          }
        }
        return size;
      };
      MyMap.prototype.has = function (key) {
        let index = this.hash(key);
        let queue = this.storeList[index];
        
        while (queue.next) {
          if (queue.next.key == key) {
            return "has存在";
          } else {
            queue = queue.next;
          }
        }
        return "不存在";
      };
      const newMap = new MyMap();

      newMap.set(1, "4");
      newMap.set(2, "6");
      newMap.set(9, "52");
      console.log(newMap.get(1));
      console.log(newMap.get(2));
      console.log(newMap.get(9));
      console.log(newMap.has(9));
```

---
title: 自己实现一个vector类
date: 2016-04-09 15:50:56
categories: technology
tags:
---

准备面试，看到很多人说遇到面试官让自己在纸上写一个std类，所以我就拿std::vector作为练手。

主要思想就是使用swap保证异常安全性，遵守`Effective C++`的一些item，比如从`copy-constructor`和`assign-constructor`提取共同的代码组成`init`函数。

虽然代码比较简单，但还是出现了越界等问题调试了很久，可能手太生了，以后得多练练！
```c++
#include <utility>
#include <algorithm>
#include <iostream>

class myvec {
private:
    int *mdata = nullptr;
    int msize = 11;
    int mlastidx = -1;
 
     void ensureSize() {
         if (msize <= mlastidx + 1) {
             reserve(msize * 2);
         }
     }
 
     void init(const myvec &vec) {
         if (&vec == this)
             return;
 
         myvec newvec(vec.msize);
 
         for (int i = 0; i <= vec.mlastidx; ++i) {
             newvec.insert(vec.mdata[i]);
         }
 
         swap(newvec);
     }
 
 public:
     myvec() : myvec(11) {
 
     }
 
     myvec(int capacity) : msize(capacity) {
         try {
             mdata = new int[capacity];
         }
         catch (const std::bad_alloc &e) {
             std::cerr << e.what() << std::endl;
             abort();
         }
     }
 
     myvec(const myvec &vec) {
         init(vec);
     }
 
     ~myvec() {
         delete[] mdata;
         std::cout<<"deconstructed"<<std::endl;
     }
 
     const myvec &operator=(const myvec &vec) {
         init(vec);
         return *this;
     }
 
     int size() {
         return mlastidx + 1;
     }
 
     int &operator[](int index) {
         if (index < 0 || index > mlastidx)
             throw "invalid access";
         return mdata[index];
     }
 
     void swap(myvec &other) {
         std::swap(this->msize, other.msize);
         std::swap(this->mdata, other.mdata);
         std::swap(this->mlastidx, other.mlastidx);
 
     }
 
     void reserve(int capacity) {
         if (capacity <= mlastidx) {
             return;
         }
         myvec newvec(capacity);
 
         for (int i = 0; i <= mlastidx; ++i) {
             newvec.insert(mdata[i]);
         }
 
         swap(newvec);
     }
 
     bool empty() {
         return mlastidx == -1;
     }
 
     void insert(int val) {
         ensureSize();
         mlastidx++;
         mdata[mlastidx] = val;
     }
 
     void erase(int index) {
        if (index < 0 || index > mlastidx) {
            return;
        }
        for (int i = index; i < mlastidx; ++i) {
            mdata[i] = mdata[i + 1];
        }
        mlastidx--;
    }

    int find(int val) {
        for (int i = 0; i <= mlastidx; ++i) {
            if (mdata[i] == val) {
                return i;
            }
        }
    }

    int count(int val) {
        int cnt = 0;
        for (int i = 0; i <= mlastidx; ++i) {
            if (mdata[i] == val) {
                cnt++;
            }
        }
        return cnt;
    }

    void clear() {
        myvec newvec(msize);
        swap(newvec);
    }
};
```

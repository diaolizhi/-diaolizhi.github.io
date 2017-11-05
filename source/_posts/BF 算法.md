---
title: BF 算法
date: 2017-11-3
categories: 数据结构
---

一种相对简单的串匹配算法.
<!--more-->
说着简单, 写起来难..



# 一. 串的定义
BF 算法用于串的匹配, 串就是字符串. 这里的字符串跟字符数组稍有不同, 定义如下:
```c
#define MAXSTRLEN 255

typedef unsigned char SString[MAXSTRLEN+1];
```
数组每个元素存储一个字符, 但是和之前学的字符数组不一样, 这里第一个元素不用于存放字符, 而是用于**存放串的长度**.

# 二. 串的初始化
```c
void InitString(SString S)
{
    int i, j, len;
    char temp[MAXSTRLEN];
    printf("请输入字符串: \n");
    // scanf("%s", temp);    //通过 scanf 输入字符串, 遇到空格会结束, 所以不能用这个
    gets(temp);
    len = strlen(temp);
    for(i=0, j=1; i<len; i++, j++)
    {
        S[j] = temp[i];
    }
    S[0] = len;    //用 0 号元素存放串长
}
```

# 三. 串的匹配
代码如下:
```c
int IndexString(SString S, SString T, int pos)
{
    int i=pos, j=1, num=0;
    while(i<=S[0] && j<=T[0])
    {
        num++;
        if(S[i] == T[j]) { i++; j++; }
        else { i = i-j+2; j = 1; }
    }
    printf("总共进行了 %d 次匹配.\n", num);
    if(j>T[0]) return i-T[0];
    else return 0;
}
```
- 首先看形参, 两个用来匹配的串, 和 pos. pos 是指定从哪个位置开始匹配.
- 其次初始化的时候, i=pos 之后则 S[i] 就指向了指定元素, 而 j=1 是因为模式串(也就是短的串)是从第一个字符开始匹配的. 
- 然后跳出条件是 i 小于 S 串的长度或者 j 小于 T 串的长度.
- 如果 S[i] 和 T[j] 指向的字符相同, 那么 i++, j++ ,使他们指向下一个.
- 如果不同, i 就回退到本次匹配位置的下一位.**如果 i=i-j 会使 i 退到原来位置的前一位**(这是因为 j 从 1 开始, 假设移动了一次, j 就已经是 2 了, 所以减去 j 会退到前一位.)**, i=i-j+1 刚好使 i 指向本次匹配开始的位置, 所以 i=i-j+2 就是下一轮要匹配的起始位置.** 
- 跳出 while 的条件要么是 S 串到头了, 要么是 T 串到头了, 如果是 T 串到头, 说明匹配成功. T 串到头也就是 if 里面的条件: j>T[0].
- 为什么是 return i-T[0] 呢? 因为要求的是 T 串在 S 串中出现的位置, 匹配成功时, i 往前移动了一位, 此时减去 T 串的长度刚好是 T 串在 S 串中出现的位置.

# 四. 完整代码
```c
#include <stdio.h>
#include <string.h>

#define MAXSTRLEN 255

typedef unsigned char SString[MAXSTRLEN+1];

void InitString(SString S)
{
    int i, j, len;
    char temp[MAXSTRLEN];
    printf("请输入字符串: \n");
    // scanf("%s", temp);    //通过 scanf 输入字符串, 遇到空格会结束, 所以不能用这个
    gets(temp);
    len = strlen(temp);
    for(i=0, j=1; i<len; i++, j++)
    {
        S[j] = temp[i];
    }
    S[0] = len;
}

int IndexString(SString S, SString T, int pos)
{
    int i=pos, j=1, num=0;
    while(i<=S[0] && j<=T[0])
    {
        num++;
        if(S[i] == T[j])
        {
            i++;
            j++;
        }
        else
        {
            i = i-j+2;
            j = 1;
        }
    }
    printf("总共进行了 %d 次匹配.\n", num);
    if(j>T[0])
    {
        return i-T[0];
    }
    else
    {
        return 0;
    }
}

int main(int argc, char const *argv[])
{
    SString S, T;
    int index;
    InitString(S);
    InitString(T);

    index = IndexString(S, T, 1);
    printf("子串在 %d 位置出现.\n", index);

    return 0;
}
```


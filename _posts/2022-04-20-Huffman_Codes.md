---
title: Huffman Codes
author: jwimd
date: 2022-04-20 13:00:00 +0800
categories: [Data Structure, Trees]
tags: [Data Structure, Huffman Codes, CPP]
math: true
mermaid: true
---

## 1. Introduction

We need to judge the correctness of Hoffman's code. Hoffman encoding refers to the indefinite length of each character based on its number of occurrences, and the requirement of encoding is that each encoding does not become a prefix for another encoding. The way Hoffman codes is not unique, we get the number of times each character appears and several sets of encodings, and we need to judge whether each encoding is correct or not. 

## 2. Implement Ideas and Theoretical Performance

### 2.1 Data Structure

```c
typedef struct HFNode *HFTree;
struct HFNode{//the data structure to record frequency
    char letter;
    int fre;
    int LChild;
    int RChile;
    int Parent;
};
```

We need to store each character and the number of occurrences it corresponds to. At the same time, considering that a tree needs to be built, it is also necessary to store the location of its children and parent. Because we intend to take an array to store information for each character, the storage of the parent node child node can take the same way that the subscript is stored.

```c
typedef struct Encode *codes;
struct Encode{
    char letter;
    char code[64];
};
```

We also need to store each character and its corresponding encoding, so we also build a struct.

### 2.2 Algorithm

#### 2.2.1 Creation of Tree

The tree is relatively simple to build, you only need to find two nodes with the smallest number of occurrences in the node without a parent node and merge them.

#### 2.2.2 Check

There are two points we need to judge, whether the length of the code matches, and the total length statistics are unique. If the length does not match, it can be directly judged as an error.

Also, you need to determine whether each encoding is a prefix for another. Considering that the earlier the encoding, the longer, therefore, check whether the subsequent encoding is a prefix of the preceding encoding, and once it is found that it is not a prefix, the result is directly obtained. A normal end to the loop indicates a prefix relationship and an incorrect encoding.

## 3. Testing Results and Analysis

### 3.1 PAT

![](../assets/img/pictures/2022-04-20-Huffman_Codes/1.png)

### 3.2  Analysis

![](../assets/img/pictures/2022-04-20-Huffman_Codes/0.png)

#### 3.2.1 Influence of Frequency

When character = 63 and submission=1. 

![](../assets/img/pictures/2022-04-20-Huffman_Codes/2.png)

#### 3.2.2 Influence of Character

When frequency is random and submission=1.

![](../assets/img/pictures/2022-04-20-Huffman_Codes/3.png)

#### 3.2.3 Influence of Submission

When frequency is random and character = 63.

![](../assets/img/pictures/2022-04-20-Huffman_Codes/4.png)

## 4. Conclusion and Comment

### 4.1 Theoretical analysis

For the construction of the tree, it is necessary to traverse two rounds each time to find two minimum occurrences, and the total number of rounds required is $N$, so the complexity is $O(N^2)$.

The part that determines whether it is correct only needs to consider the prefix judgment part, which can be easily seen that the complexity is $O(N^2)$.

Therefore, the total complexity is also $O(N^2)$.

### 4.2 Improve ideas

The complexity of the tree construction process can be reduced by means of sorting and double queuing, and it can be optimized to $O(NlogN)$.

## 5 Source Code

```c++
#include<stdio.h>
#include<stdlib.h>
//struct of frequency
typedef struct HFNode *HFTree;
struct HFNode{//the data structure to record frequency
    char letter;
    int fre;
    int LChild;
    int RChile;
    int Parent;
};
//structure of code
typedef struct Encode *codes;
struct Encode{
    char letter;
    char code[64];
};
//global variable
HFTree T;
//functions
void Combine(int idx);
int is_pre(codes test,int i,int j);
void huffman_check_gate(int n,int Weight);
int main(){
    int character;//number of characters that need to be encode
    int submission;//number of submissions that need to be check
    scanf("%d",&character);
    getchar();
    T=(HFTree)malloc(sizeof(struct HFNode)*(character*2-1));
    for(int i=0;i<character;i++){//read in the statistic
        scanf("%c %d",&T[i].letter,&T[i].fre);
        getchar();
        T[i].LChild=-1;
        T[i].RChile=-1;
        T[i].Parent=-1;
    }
    int Weight=0;
    for(int i=character;i<2*character-1; i++){//structure the HFTree 
        Combine(i);//combine the two smallest node
        Weight+=T[i].fre;//calculate the total weight
    }
    scanf("%d",&submission);
    for(int i=0;i<submission;i++)huffman_check_gate(character,Weight);
    return 0;
}

void Combine(int idx){
    int min1=-1,min2=-1;
    for(int i=0;i<idx;i++){//find the idx of the letter with smallest frequency
        if(T[i].Parent<0){
            if(min1<0||T[min1].fre>T[i].fre)min1=i;
        }
    }
    for(int i=0;i<idx;i++){//find the idx of the letter with second smallest frequency
        if(i!=min1&&T[i].Parent<0){
            if(min2<0||T[min2].fre>T[i].fre)min2=i;
        }
    }
    //combine
    T[idx].LChild=min1;
    T[idx].RChile=min2;
    T[idx].Parent=-1;
    T[idx].fre=T[min1].fre+T[min2].fre;
    T[min1].Parent=T[min2].Parent=idx;
    return;
}
int is_pre(codes test,int i,int j){
    int len=strlen(test[j].code);
    for(int k=0;k<len;k++){//different prefix
        if((test[i].code[k])!=(test[j].code)[k]){
            return 0;
        }
    }
    return 1;//the same prefix
}
void huffman_check_gate(int n,int Weight){
    codes test=(codes)malloc(sizeof(struct Encode)*n);
    int weight=0;
    for(int i=0;i<n;i++){
        scanf("%c%*c%s",&test[i].letter,&test[i].code);
        //calculate the weight
        weight+=strlen(test[i].code)*T[i].fre;
    }
    if(weight!=Weight){
        //weights are different
        printf("No\n");
        return;
    }
    else{
        int flag=1;
        for(int j=n-1;j>=1&&flag;j--){
            for(int i=0;i<j;i++){
                if(is_pre(test,i,j)){
                    //prefix of some string
                    printf("No\n");
                    return;
                }
            }
        }
    }
    //the best encoding way
    printf("Yes\n");
    return;
}
```


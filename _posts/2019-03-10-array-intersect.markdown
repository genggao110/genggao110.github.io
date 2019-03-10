---
layout:      post
title:       "Intersection of Two Arrays"
subtitle:    " \"LeetCode Algorithm\""
date:        2019-03-10 15:38:37
author:      "Ming"
header-img:  "img/post-bg-201903.jpg"
tags:
    - JAVA
    - Algorithm
    - LeetCode
---

> "The world may be full of cheating, however we never lack friends with a warm heart. "

## 两个数组的交集

给定两个数组，编写一个函数来计算他们的交集。

**示例 1：**
```
输入：nums1 = [1,2,2,1], nums2=[2,2]
输出：[2]
```
**示例 2：**
```
输入: nums1 = [4,9,5], nums2 = [9,4,9,8,4]
输出: [9,4]
```
**说明**
```
- 输出结果中每一个元素一定是唯一的。
- 我们可以不考虑输出结果的顺序。
```

### 解法一

拿到题目，第一眼我想的就是两层for循环来解决问题。为此，给出的解决方案如下所示：

```java
public int[] intersection(int[] nums1, int[] nums2){
        Set<Integer> hashSet = new HashSet<Integer>();
        for(int i = 0; i < nums1.length; i++){
            for(int j = 0; j < nums2.length; j++){
                if(nums1[i] == nums2[j]){
                    hashSet.add(nums1[i]);
                }
            }
        }
        Integer[] temp = hashSet.toArray(new Integer[hashSet.size()]);
        int[] result = Arrays.stream(temp).mapToInt(Integer::intValue).toArray();
        return result;
    }
```
然而发现运行后的效率居然会特别的低。本着不服输的态度，我就看了下那些排名靠前和网上的一些解决方案，大致总结了以下的解法。

### 指针比较法

指针比较法的思路也是比较简单的：
- 首先对两个数据进行排序，初始化两个数组的指针，即均从0开始；
- 将小数组的指针做为外层循环，在大数组指针位置开始比较；
- 如果找到相等的，记录结果，同时将两个数组指针都向后移动；
- 如果在大数组中找到比先前大一个数进行比较后还未找到，那么小数组的指针向后移动；
- 当小数组的指针移动到最后一个元素时结束算法


具体代码如下所示：

```java
public int[] intersect2(int[] nums1, int[] nums2){
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        Set<Integer> intersectionSet = new HashSet<Integer>();
        if(nums1.length > nums2.length){
            intersectionSet = compare(nums2, nums1);
        }else{
            intersectionSet = compare(nums1, nums2);
        }

        int[] result = new int[intersectionSet.size()];
        int t = 0;
        for(Integer i : intersectionSet){
            result[t++] = i;
        }
        return result;

    }

//私有方法
private Set<Integer> compare(int[] smallArray, int[] bigArray){
        Set<Integer> intersectionSet = new HashSet<Integer>();
        int indexArrayA = 0;
        int indexArrayB = 0;
        int sizeArrayA = smallArray.length;
        int sizeArrayB = bigArray.length;
        while (indexArrayA < sizeArrayA){
            for(int i = indexArrayB; i < sizeArrayB; i++){
                if (smallArray[indexArrayA] == bigArray[i]) {
                    intersectionSet.add(smallArray[indexArrayA]);
                    indexArrayA++;
                    indexArrayB++;
                    break;
                } else if (smallArray[indexArrayA] < bigArray[i]) {
                    indexArrayA++;
                    break;
                } else if (i == sizeArrayB - 1) {
                    indexArrayA++;
                }
            }
        }
        return intersectionSet;
    }
```

### 利用JAVA集合的取交集方法

继承Collection接口的包含有一个retainAll的方法，我们利用Set可以实现两个数组的交集。具体代码如下，这里利用了Stream API,运行速率会比较慢，可以换成for循环(实现int[] 与Integer[] 的转换)

```
public int[] intersect2(int[] nums1, int[] nums2){
        List<Integer> array1 = Arrays.stream(nums1).boxed().collect(Collectors.toList());
        List<Integer> array2 = Arrays.stream(nums2).boxed().collect(Collectors.toList());

        Set<Integer> intersectionSet = new HashSet<Integer>();

        intersectionSet.addAll(array1);
        intersectionSet.retainAll(array2);

        Integer[] temp = intersectionSet.toArray(new Integer[intersectionSet.size()]);
        int[] result = Arrays.stream(temp).mapToInt(Integer::intValue).toArray();
        return result;
    }
```

### 利用队列

队列的方法与指针比较法有相同之处。将原数组进行排序，然后将数组加入到队列中，拿元素个数较小的作为循环条件，比较两个队列peek的数值。相等则输出并出队列，否则将较小值所在的队列进行出列操作直到某个队列为空结束循环。(此方法比较适合大数据量等级)

```
public int[] intersect2(int[] nums1, int[] nums2){
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        Set<Integer> intersectionSet = new HashSet<Integer>();
        if(nums1.length > nums2.length){
            intersectionSet = compare(nums2, nums1);
        }else{
            intersectionSet = compare(nums1, nums2);
        }

        int[] result = new int[intersectionSet.size()];
        int t = 0;
        for(Integer i : intersectionSet){
            result[t++] = i;
        }
        return result;

    }

    //私有方法
    private Set<Integer> compare(int[] smallArray, int[] bigArray){
        Set<Integer> intersectionSet = new HashSet<Integer>();
        int indexArrayA = 0;
        int indexArrayB = 0;
        int sizeArrayA = smallArray.length;
        int sizeArrayB = bigArray.length;
        Queue<Integer> queueA=new ArrayBlockingQueue<Integer>(sizeArrayA);
        Queue<Integer> queueB=new ArrayBlockingQueue<Integer>(sizeArrayB);

        for(int i=0;i<sizeArrayA;i++){
            queueA.add(smallArray[i]);
        }
        for(int i=0;i<sizeArrayB;i++){
            queueB.add(bigArray[i]);
        }
        while (!queueA.isEmpty()){
            Integer valueA=queueA.peek();
            Integer valueB=queueB.peek();
            if(null==valueA||null==valueB){
                break;
            }
            if(valueA.equals(valueB)){
                intersectionSet.add(valueA);
                queueA.poll();
                queueB.poll();
            }
            else if(valueA>valueB){
                queueB.poll();
            }
            else if(valueA<valueB){
                queueA.poll();
            }
        }
        return intersectionSet;
    }
```
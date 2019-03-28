### 1 Arrays Traversal

####       a. N-Repeated Element in Size 2N Array

​	当单次遍历数组时，一定要注意A[i],A[i+1],A[i+2]三项之间的关系，可以减少hashmap等更复杂的数据结构介入。

```java
      for(int i=0;i<A.length-2;i++)
          if(A[i]==A[i+1]||A[i]==A[i+2])
              return A[i];
        
        return A[A.length-1];
```

#### 	b. Reveal Cards In Increasing Order

​      基本的遍历方式，利用queue,stack来模拟遍历过程。

```java

```



#### c. Tian's Horse Race

​	  1）考虑优先级队列来实现。

```
public int[] advantageCount(int[] A, int[] B) {
    // TJSM is the Chinese acronym for the original story
    int[] TJSM = new int[A.length];

    // Sort my hand to have an ordered horse quality
    Arrays.sort(A);
		
		//大顶堆
    // Use a max heap for my opponents to deal with his faster horse first
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>((a, b) -> b[1] - a[1]);
    for (int i = 0; i < B.length; i++) {
        maxHeap.offer(new int[] {i, B[i]});
    }

    int slow = 0, fast = A.length - 1;
    while (!maxHeap.isEmpty()) {
        int[] b = maxHeap.poll();
        // If my fastest horse remained is slower than my opponents' fastest horse,
        // there is no way for me to win, use my slower horse.
        // Otherwise use my fastest horse to win this round.
        // Why using my second fastest horse won't improve my global scores?
        // If my second fastest horse is faster than my opponents' fastest one,
        // it sure is faster than the rest of his horse. Thus proved this strategy is
        // optimal.
        TJSM[b[0]] = b[1] >= A[fast] ? A[slow++] : A[fast--];
    }

    return TJSM;
}
```

​	2）采用TreeMap来实现

```
    public int[] advantageCount(int[] A, int[] B) {
       TreeMap<Integer, Integer> map = new TreeMap<>();
       for(int i = 0; i < B.length; i++) {
            map.put(A[i], map.getOrDefault(A[i], 0) + 1);
       }
       int key = -1;
       for(int i = 0; i < A.length; i++) {
            if(map.higherKey(B[i]) != null) {
                key = map.higherKey(B[i]);
             }
            else {
                key = map.firstKey();
            }
            A[i] = key;
            map.put(key, map.get(key) - 1);
           if(map.get(key) == 0) {
              map.remove(key);
            }
       }
       return A;
       
    }
```

### 2 双标记位

- 双指针

  quicksort:

  ```java
  public  static void quicksort(int[] nums,int start,int end){
      //递归出口
          if(start>end)
              return ;      
     //基数    
      int temp=nums[start];
          int i=start,j=end;
      //往中间找   
      while(i!=j)
          {//从右往左找，最终是跟i交换 j=j-1 ==i ;
          //而不是跟i+1交换
          while(nums[j]>=temp&&i<j)
              j--;
  
            while(nums[i]<=temp&&i<j)
                i++;
  	//交换
            if(j>i)
            {int atemp=nums[i];
            nums[i]=nums[j];
            nums[j]=atemp;
            }
          }
      //已找到
          nums[start]=nums[i];
          nums[i]=temp;
          quicksort(nums,start,i-1);
          quicksort(nums,i+1,end);
      }
  ```

  

- 双标记位

  Best Time to Buy and Sell Stock：

  ```java
  //Input: [7,1,5,3,6,4]
  //Output: 5
     public int maxProfit(int[] prices) {
  		
         //一次遍历的时候保存两个有效值，当前最低价，当前//高利润
         int min_price=Integer.MAX_VALUE;
         int profit=0;
         for(int i=0;i<prices.length;i++)
         {
             if(prices[i]<min_price)
                 min_price=prices[i];
             
             if(prices[i]-min_price>profit)
                 profit=prices[i]-min_price;
             
         }
   
         return profit;
  }
  ```

  


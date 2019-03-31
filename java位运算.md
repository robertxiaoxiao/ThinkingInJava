java位运算：

1 左移 <<

2 右移 >>

3 无符号右移 >>

4 位与 &    :位上都为1则为1，否则为0

5 位或 | ：有一个为1则为1 ，else =0；

6 位异或^ :两个不同则为1，else=0;

7  一个整型数组里除了两个数字之外，其他的数字都出现了两次。请写程序找出这两个只出现一次的数字。

integer  4bytes 32bits .

**方案二：利用Java位运算符，完成Unsigned转换。**

```java 

//正常情况下，Java提供的数据类型是有符号signed类型的，可以通过位运算的方式得到它们相对应的无符号值，参见几个方法中的代码：

public int getUnsignedByte (byte data){     
    //将data字节型数据转换为0~255 (0xFF 即BYTE)。
return data&0x0FF;
}

public int getUnsignedByte (short data){      //将data字节型数据转换为0~65535 (0xFFFF 即 WORD)。
return data&0x0FFFF;
}

public long getUnsignedIntt (int data){     //将int数据转换为0~4294967295 (0xFFFFFFFF即DWORD)。
return data&0x0FFFFFFFFl;
}
```



```java

public static void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {

    unsigned int
        int temp=0;
    //有两个数不一样，则异或不为0；则该位为1 或者为0可分为两组数
    //每组内只有一个数不同，其他都相同，直接去异或就可以
    // a^a=0 ,a^0=a;
        for(int i:array)
            temp^=i;

        int idx=find(temp);

        int t1=0;
        int t2=0;
        for(int j:array)
        {
            if(find(j)==idx)
                t1^=j;
            else
                t2^=j;
        }
        num1[0]=t1;
        num2[0]=t2;


    }
//查找第k位为1
    public static int find(int a){

        int count=1;
        while((a&1)==0)
        {
            a=a>>1;
            count++;
        }
        return count;

    }
//判断两数是否相同
//t1^t2==0
//判断某位是否为1  a&1
public  static boolean judge(int a,  int index){

      a=a>>index;
      if((a&1)==1)
          return true;
      else
          return false;
}
```

位运算模拟加法

​	首先看十进制是如何做的： 5+7=12，三步走 第一步：相加各位的值，不算进位，得到2。 第二步：计算进位值，得到10. 如果这一步的进位值为0，那么第一步得到的值就是最终结果。  第三步：重复上述两步，只是相加的值变成上述两步的得到的结果2和10，得到12。  同样我们可以用三步走的方式计算二进制值相加： 5-101，7-111 第一步：相加各位的值，不算进位，得到010，二进制每位相加就相当于各位做异或操作，101^111。  第二步：计算进位值，得到1010，相当于各位做与操作得到101，再向左移一位得到1010，(101&111)<<1。  第三步重复上述两步， 各位相加010^1010=1000，进位值为100=(010&1010)<<1。      继续重复上述两步：1000^100 = 1100，进位值为0，跳出循环，1100为最终结果。  

```java
public static int Add(int num1,int num2) {
    
    
    
    while(num1!=0)
    {
        // 25+75 多次进位  所以加法是个循环迭代的过程
        //相对于加法 不带进位
        int temp=num1^num2;
        //算进位值
          num1 =(num1&num2)<<1;
          num2=temp;
    }
    return num2;
}


//作为"&&"和"||"操作符的操作数表达式，这些表达式在进行求值时，只要最终的结果已经可以确定是真或假，求值过程便告终止，这称之为短路求值（short-circuit evaluation）。这是这两个操作符的一个重要属性。
 public int Sum_Solution(int n) {

        int ans = n;
        boolean t=((ans!=0) && ((ans += Sum_Solution(n - 1))!=0));
        return ans;
        
    }
```

5  巧用标记位

​	在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，

​	

```java
//不需要额外的数组或者hash table来保存，题目里写了数组里数字的范围保证在0 ~ n-1   之间，所以可以利用现有数组设置标志，当一个数字被访问过后，可以设置对应位上的数 +   n，之后再遇到相同的数时，会发现对应位上的数已经大于等于n了，那么直接返回这个数即可  

public int find_dup( int numbers[], int length) {

    for ( int i= 0 ; i<length; i++) {

        int index = numbers[i];

        if (index >= length) {

            index -= length;

        }   

        if (numbers[index] >= length) {

            return index;

        }   

        numbers[index] = numbers[index] + length;

    }   

    return - 1 ; 

}

```


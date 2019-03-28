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


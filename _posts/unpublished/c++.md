# c++ 

标签（空格分隔）： c++

---
[TOC]

---
## 一、选择题

### 1. 以下表达式的值是（）
``` 条件运算符   
1>0?3:2
```
A. 1    B. 0    C. 3    D. 2  

### 2. 以下基本数据类型的字节数排序正确的是（）
``` 数据类型   
```
A. double > int > short > bool    
B. doubel > char > int > bool
C. int > double > char > short 
D. bool > char > int > short

### 3. 以下程序的输出为（）
``` 注释
#include <iostream>
using namespace std;
int main(){
    cout<<"/*1*2+*/3*4"<<endl;
    return 0;
}
```
A. /*1*2+*/3*4  B. 3*4  C. 14   D. 12

### 4. 以下哪些是正确的变量定义方式（）
``` 变量定义
1. int a(3);
2. char c = "car";
3. const float PI; PI=3.1415926;
4. float f += 3.5;
```
A. 1    B. 1和2     C. 3     D. 3和4

### 5. 以下表达式的值为假的是（）
``` 逻辑符
```
A. (3==4)||(4>2)    
B. (6<=3+3)||(2>=3)     
C. (3+3<=6)&&(-6>-4)
D. !(3+5<=2+4)

### 6. 以下程序的输出为（）
``` if与a++
#include <iostream>
using namespace std;
int main(){
    int a = 1;
    int b = int(1.5);
    int c = int(0.5);
    int out = 0;
    if(a==b)
        cout<< --out <<endl;
    else if(a==c)
        cout<< out++ <<endl;
    else
        cout<< out+2 <<endl;
    return 0;
}
```
A. -1   B. 0    C. 1    D. 2

### 7. 关于以下程序段说法正确的是（）
``` do-while和int的取值范围
#include <iostream>
using namespace std;
int main(){
    int a = 3;
    do{
        a -= 5;
    }while(a<0);
    return 0;
}
```
A. 是死循环 B. 循环执行多次，但最终会跳出循环   C. 循环共执行一次   D. 循环没有被执行

### 8. 以下程序的输出为（）
``` continue语句
#include <iostream>
using namespace std;
int main(){
    int sum = 0;
    for(int i = 0; i < 5; i++){
        if (i == 2 || i == 4)
            continue;
        sum += i;
    }
    cout<<sum<<endl;
}
```
A. 10   B. 6   C. 4   D. 1

### 9. 以下语句会编译报错的是（）
``` if和const
1. for(;;) int a = 1;
2. const int i = 1; for(;i<10;i++) int a = 1;
3. for(int i=0; int j=100; i<=j; i++, j--) int a = 1;
4. for(const int i=0; i<100; ) int a = 1;
```
A. 1和2     B. 2和3     C. 3和4     D. 1和4

### 10. 以下程序的输出是（）
``` enum
#include <iostream>
using namespace std;
int main(){
    enum WEEKDAY {Monday, Tuesday, Wednesday=3, Thursday=1, Friday};
    cout<< Monday+Tuesday+Wednesday+Thursday+Friday<<endl;
    return 0;
}
```
A. 7   B. 8   C. 9   D. 10

### 11. 以下程序的输出是（）
``` &符号
#include <iostream>
using namespace std;
int main(){
    int a = 100;
    int &b = a;
    a = a + 200;
    cout<<b<<endl
    return 0;
}
```
A. 0   B. 100   C. 200   D. 300

### 12.下列哪些为不合法的重载形式（）
``` 函数重载
1. float func1();
   float func2();
2. float func();
   int func();
3. char* func(char* c);
   char* func(char* d);
4. int func(float a);
   int func(float a, float* b);
```
A. 1,2和3     B. 2,3和4     C. 1,3和4     D. 1,2和4

### 13. 以下定义函数的方式错误的是（）
``` 带默认形参值的函数
1. int add(int x, int y=5, int z=6);
2. int add(int x=1, int y=5, int z);
3. int add(int x=1, int y, int z=6);
```
A. 1和2     B. 2和3     C. 1和3     D. 1,2和3

### 14.以下程序的输出为（）
```函数的嵌套调用
#include <iostream>
using namespace std;
int func1(int m){
    return m*m;
}
int func2(int x, int y){
    return func1(x)+func1(y);
}
int main(){
    cout<<func2(3, 4)<<endl;
    return 0;
}
```
A. 16     B. 9     C. 25     D. 7

### 15. 以下程序的输出为（）
```函数的值传递与引用传递
#include <iostream>
using namespace std;
int func(int x, int& y){
    int tmp = x;
    x = y;
    y = tmp;
    return x+y;
}
int main(){
    int x = 0;
    int y = 1;
    cout<<func(x,y)<<" "<<x<<" "<<y<<endl;
    return 0;
}
```
A. 1 0 1     B. 1 0 0     C. 1 1 0    D. 0 1 1


## 二、程序填空

### 1. 以下程序段用于输入一个年份，判断是否为闰年。请补全所空处的代码，每空仅填一行代码：

``` 
#include <iostream>
using namespace std;
int main(){
    int year;
    bool isLeapYear;
    
    cout<<"Enter the year: ";
    cin>>year;
    isLeapYear = ______________________________ ;//能被4整除且不能被100整除的为闰年，另外能被400整除的也是闰年
    
    if (isLeapYear)
        cout<<year<<" is a leap year"<<endl;
    else
        cout<<year<<" is not a leap year"<<endl;
    
    return 0;
}
```

### 2. 汉诺塔问题：有三个塔(分别为A号，B号和C号)。开始时．有 n个圆形盘以从下到上、从大到小的次序叠置在A塔上。现要将A 塔上的所有圆形盘，借助B塔，全部移动到C塔上。且仍按照原来 的次序叠置。 移动的规则如下：这些圆形盘只能在3个塔问进行移动．一 次只能移动一个盘子，且任何时候都不允许将较大的盘子压在比 它小的盘子的上面。程序的输入为n，输出为移动过程。
示例： 
输入：
3
输出：
A--->C
A--->B
C--->B
A--->C
B--->A
B--->C
A--->C
```
#include <iostream>
using namespace std;
//把src塔的最上面一个盘子移动到dest塔上
void move (char src, char dest){
    cout<<______<<"--->"<<______<<endl;
}
//把n个盘子从src塔移动到dst塔，以medium塔作为中介
void hanoi(int n, char src, char medium, char dest){
    if (______)
        move(src, dest);
    else{
        ______________________________;
        move(src,dest);
        ______________________________;
    }
}
int main(){
    int m;
    cin>>m;
    hanoi(m, 'A', 'B', 'C');
    return 0;
}
```

### 3. 编写函数求两个整数的最大公约数
```
#include <iostream>
using namespace std;
int func(int i, int j){
    int temp;
    if(i<j){//如果i比j小，交换i与j的值。
        temp = i;
        ________;
        ________;
    }
    while (j!=0) {
        temp = i%j;
        i = j;
        j = temp;
    }
    return ______;
}
int main(){
    int i,j;
    cout<<"请输入一个正整数：";
    cin>>i;
    cout<<"请输入另一个正整数： ";
    cin>>j;
    cout<<i<<"和"<<j<<"的最大公约数是："<<func(i,j)<<endl;
    return 0;
}
```

## 编程题

请编写函数fun,其功能是:计算并输出下列多项式值: Sn=1+1/1!+1/2!+1/3!+1/4!+…+1/n!  例如,若主函数从键盘给n输入15,则输出为s=2.718282。注意:n的值要求大于1但不大于100。
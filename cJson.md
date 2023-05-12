### ifndef

头件的中的#ifndef，这是一个很关键的东西。比如你有两个C文件，这两个C文件都include了同一个头文件。而编译时，这两个C文件要一同编译成一个可运行文件，于是问题来了，大量的声明冲突。 

还是把头文件的内容都放在#ifndef和#endif中吧。不管你的头文件会不会被多个文件引用，你都要加上这个。一般格式是这样的： 

```
#ifndef <标识> 
#define <标识> 
...... 
...... 


#endif 
```

<标识>在**理论上来说可以是自由命名的**，但每个头文件的这个“标识”都应该是唯一的。标识的命名规则一般是头文件名全大写，前后加下划线，并把文件名中的“.”也变成下划线，如：stdio.h 

```
#ifndef _STDIO_H_ 
#define _STDIO_H_ 
...... 
#endif 
```

### **extern "C"**

**extern "C"的主要作用就是为了能够正确实现C++代码调用其他C语言代码。加上extern "C"后，会指示编译器这部分代码按C语言的进行编译，而不是C++的。**由于C++支持函数重载，因此编译器编译函数的过程中会将函数的参数类型也加到编译后的代码中，而不仅仅是函数名；而C语言并不支持函数重载，因此编译C语言代码的函数时不会带上函数的参数类型，一般之包括函数名。



### 字符串数字转化为数字型数字

```
#include<stdio.h>
#include<math.h>

double stringTodouble(const char *num){
    double n = 0, sign = 1,scale = 0;
    int subscale = 0, signsubscale = 1;
    if (*num == '+') num++;
    if (*num == '-') {sign = -1; num++;}
    if (*num == '0') num++;
    if (*num >= '1' && *num <= '9') do n = (n*10.0) + (*num++ - '0'); while (*num >= '0' && *num <= '9');
    if (*num == '.' && num[1] >= '1' && num[2] <= '9') {num++; do {n = (n*10.0) + (*num++ - '0'); scale--;} while (*num >= '0' && *num <= '9');}
    if (*num == 'e' || *num == 'E'){
        num++; if (*num == '+') num++; else if(*num == '-') {signsubscale = -1; num++;}
        while(*num >= '0' && *num <= '9') {subscale = (subscale*10)+(*num-'0'); num++;}
    } 
    
    n = sign*n*pow(10.0, (scale+signsubscale*subscale));
    return n;
}

int main(int argc, char const *argv[])
{
    char* string = "-201314.123E-2";
    double ans = stringTodouble(string);
    printf("%lf", ans);
    return 0;
}
```




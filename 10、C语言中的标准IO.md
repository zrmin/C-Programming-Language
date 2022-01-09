<center><h1>C语言中的标准IO</h1></center>

C语言在标准库<stdio.h>中提供可供我们使用的I/O接口。

## 一、常规I/O接口的使用

我们回顾一下，常用的I/O：

![image-20220109120851107](https://raw.githubusercontent.com/zrmin/C-Programming-Language/master/images/202201091208259.png)

1. 第10行的printf函数，将“Enter some characters"传送到标准输出流stdout
2. 第11行的fopen函数，以"w+"的模式，打开文件”temp.txt"，并将其与一个特定的文件I/O流相关联

3. 第14行的scanf函数，获取标准输入流stdin中的数据
4. 第16行的putc函数，将从stdin获取的输入ch写入到文件指针fp所指的文件位置上
5. 第20行的perror函数，将错误信息传送到标准错误流stderr
6. 第23行fclose函数，将打开的文件关闭

| 函数名 |                             意义                             |
| :----: | :----------------------------------------------------------: |
| fopen  |                         opens a file                         |
|  putc  |             writes a character to a file stream              |
| perror | displays a character string corresponding of the current error to `stderr` |
| fclose |                        closes a file                         |
| scanf  | reads formatted input from `stdin`, a file stream or a buffer |
| printf | prints formatted output to `stdout`,a file stream or a buffer |



文件打开的模式还有下面这些常用的：

| 文件获取模式 |      意义       |       解释       |  如果文件已存在  | 如果文件不存在 |
| :----------: | :-------------: | :--------------: | :--------------: | :------------: |
|     "w+"     | write extended  | 创建文件以供读写 |     销毁内容     |   创建新文件   |
|     "r+"     |  read extended  | 打开文件以供读写 | 从文件开头开始读 |      报错      |
|     "a+"     | append extended | 打开文件以供读写 |   在文件末尾写   |   创建新文件   |

通过上面这个例子，我们基本上了解了C语言提供的标准IO的使用方法。

我们发出疑问：<font color = red>标准</font>IO，难道有不标准的IO吗？

答：有，不过不能叫不标准IO，应该叫低级IO。

所谓`低级IO`，就是与系统底层实现相关的IO接口，例如Unix/类Unix平台上的POSIX接口。我们使用POSIX接口提供的IO操作来完成上面同样的任务看一看：

![image-20220109124405963](https://raw.githubusercontent.com/zrmin/C-Programming-Language/master/images/202201091244025.png)

##### open函数

1. open函数的定义

```c
int open(const char* pathname, int flags);
int open(const char* pathname, mode_t mode);
```

2. 例子

就像我们上面的程序中那样，使用就是第一种open函数定义的形式：

```c
 const int fd = open("./temp.txt", O_RDWR | O_CREAT); //temp.txt就是我们要打开的文件
```

3. flags有哪些

flags参数表示对打开的文件能够执行哪些操作，flags参数分为必选和可选两种，对于必选参数必须选择一种。

| flags必选参数 |   含义   |
| :-----------: | :------: |
|   O_RDONLY    |   只读   |
|   O_WRONLY    |   只写   |
|    O_RDWR     | 可读可写 |

| flags可选参数 |                             含义                             |
| :-----------: | :----------------------------------------------------------: |
|   O_APPEND    |                           追加写入                           |
|    O_CREAT    |                    若文件不存在，则创建它                    |
|    O_EXCL     |    若要创建的文件已存在，则出错，返回-1，并修改errno的值     |
|    O_TRUNC    |    若文件存在，且以只读、只写方式打开，则将其长度截断为0     |
|   O_NOCTTY    |       若路径名指向终端设备，不要把这个设备用作控制终端       |
|   O_NOBLOCK   | 若路径名指向FIFO、块文件、字符文件，则把文件的打开和后继I/O设置为非阻塞模式 |

可以看出，在使用低级接口时，我们需要控制更多的细节。例如：在调用write接口时，我们需要明确指出文件描述符以区分是像屏幕输出还是向文件输出。而在C语言提供的标准IO中，我们并不需要关注这些细节，接口的名称可以直接反映其用途。

C语言的标准IO接口在实现时，会调用具体平台的低级IO，然后低级IO又会通过系统调用来完成指定的操作。C语言的标准IO在工作时，调用了低级IO，但是标准IO与低级IO的运行速度却基本没有差别，为什么呢？

因为标准IO提供了缓冲区。



## 二、标准IO的缓冲区

我们上面提到，标准IO会调用低级IO，低级IO又会系统调用，在低级IO中，每次的写入都要调用系统调用，那么在使用标准IO时，如果每次写入都立即更新temp.txt文件，那么系统调用的次数也太多了，性能必然受到影响。因此，标准IO提供了缓冲区，将输入或者输出的内容暂存到缓冲区中，当缓冲区的空间被占用完了或者遇到特殊指令（如endl、fflush等）或者程序退出时在调用低级IO、调用系统调用，将内容从缓冲区中一次性输出。

标准IO并没有规定缓冲区的大小，因此，缓冲区的大小依赖于具体函数库的实现。C语言也提供了setvbuf函数，可以让我们自定义缓冲区的大小。

![image-20220109160318077](https://raw.githubusercontent.com/zrmin/C-Programming-Language/master/images/202201091603150.png)

![image-20220109161020317](https://raw.githubusercontent.com/zrmin/C-Programming-Language/master/images/202201091610403.png)

上面的示例中，我们设置了5字节大小的缓冲区，一个char是1byte的大小，也就是说我们的缓冲区刚好可以容纳下5个字符，在写满了5个字符后，就将这5个字符更新到temp.txt文件中。



## 三、低级IO接口的系统调用

我们现在知道了，C语言提供的标准IO会调用低级IO来完成相应的IO操作，低级IO会通过系统调用的方式来完成IO操作。那么这个系统调用的方式是如何实现的呢？我们好奇底层的实现。

系统调用是操作系统内核提供的一系列函数，只是这些函数和我们用C语言编写的程序在调用时是不同的，我们编写的程序在调用时使用`call`指令来完成，而系统调用则使用`syscall`来完成。

我们之前讲过函数调用的底层实现，我们说到根据SystemV调用约定，函数在被调用时，需要依靠rdi、rsi、rdx、rcx、r8、r9来实现实参的传递。那么系统调用也是这样吗？——<font color="red">不是的，系统调用不是的</font>。根据SystemV调用约定，系统调用将会使用寄存器rdi、rsi、rdx、r10、r8、r9来进行实参的传递。

我们还说过常用寄存器rax来传递返回值，那么系统调用也是吗？——<font color=red>是的，系统调用也是的</font>。而且，在系统调用中，rax寄存器不仅仅用在系统调用函数返回时，还用在系统调用函数被调用前——用来传递函数号，告知操作系统被调用的是哪个函数。

操作系统会为每一个系统调用函数分配唯一的一个整型ID来标识它们。在发生系统调用时，这个标识符会通过rax来传递，从而操作系统知道要调用的是几号函数。例如，在x86-64平台上的Linux操作系统中，我们上面使用过一个低级IO的open函数：

```c
const int fd = open("./temp.txt", O_RDWR | O_CREAT);
```

这个open函数在该Linux操作系统中的ID就是2。底层实现时，就是通过将这个2保存到寄存器rax中，进而告诉操作系统，我调用的是open函数的。

口说无凭，我们来具体看一下：

![image-20220109165258600](https://raw.githubusercontent.com/zrmin/C-Programming-Language/master/images/202201091652710.png)

open函数系统调用的底层实现在第15~21行：

![image-20220109165534545](https://raw.githubusercontent.com/zrmin/C-Programming-Language/master/images/202201091655603.png)

先回忆一下我们前面说的系统调用的实现：

1. 将系统调用函数的ID值存入寄存器rax中
2. 用寄存器rdi、rsi、rdx、r10、r9、r8这6个寄存器从左到右传递实参
3. 函数执行完成后，使用寄存器rax存储返回值

下面我们来具体解析一下系统调用的底层实现：

1. 第15行

```assembly
mov $2, %%rax\n\t
```

将系统调用函数open的整型ID值传入寄存器rax中，告知操作系统我们调用的是open函数。调用了函数后，我们就要开始为函数的正常运行传递参数了。我们知道open函数的第一个参数是要打开的文件名，而现在这个文件名被保存在fileName指向的地址中，所以我们要把存放有目标文件名称的字节数组fileName的首地址作为参数进行传递；open函数的第二个参数是配置参数flags，我们上面讲过。

2. 第16行

```assembly
mov %0, %%rdi\n\t ;%0对应于第20行的fileName，也就是将fileName的首地址传入寄存器rdi中
```

3. 第17行

```assembly
mov $66, %%rsi\n\t ;open函数的第二个参数是flags，其中，O_RDWR对应的十进制是2，O_CREAT对应的十进制是64，因此2+64=66传入寄存器rdi中
```

至此，利用rax，操作系统也知道了我们想要调用的是open函数，通过寄存器rdi和rsi传入了open函数的两个参数：filePath和flags。那么接下来就只剩下调用这个函数了。

4. 第18行

```assembly
syscall\n\t ;系统调用，调用open函数
```

至此，open函数就运行此来了。函数处理完任务后，就要返回了，我们知道，根据SytemV调用约定，返回值是通过寄存器rax传递的。

5. 第19行

```assembly
mov %%rax, %1\n\t ;%1对应于第21行的，也即第14行的fd，用来判断文件temp.txt是否正常打开，如果返回值大于0，则文件打开正常
```

通过上面的解析，我们初步了解了系统调用函数的一些细节。

那么，现在我们说：<font color=green>C语言提供标准IO，标准IO在实现时调用了平台相关的低级IO，低级IO通过系统调用实现相关的IO功能</font>这个过程，我们是没有疑问的了。


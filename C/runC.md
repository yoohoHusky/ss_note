##### 运行C

1. 编写C代码
  ```C
  int main(int argc, char* argv[]) {
      int i = 3;
      printf("%d", i);
      return 0;
  }
  ```

2. 终端运行`cc -c main.c`。开始编译，得到main.o二进制文件
3. 运行`cc main.o`,得到a.out可执行文件
4. 运行`a.out`

Use modules and libraries



+ Compile modules:![1668503513565](C:\Users\Administrator\Desktop\forBlog\1668503513565.png)

```c++
c++ -std=c++17 -c tools.cpp -o tools.0
```



+ Organize modules into libraries:

```c++
ar rcs libtools.a tools.o <other_modules>
```



+ Link libraries when building code:

```c++
c++ -std=c++17 main.cpp -L . -ltools -o main
```



+ Run the code:

```c++
./main
```


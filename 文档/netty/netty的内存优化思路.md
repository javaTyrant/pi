## netty优化内存使用的技巧

内存目标:时间快,内存小

1.能用基本类型的地方就不要用包装类型

2.能定义成类变量的就不要定义成实例变量

3.预分配内存大小

4.零拷贝

5.内存池

![image-20210208221903365](/Users/lumac/Library/Application Support/typora-user-images/image-20210208221903365.png)

对分配内存进行预估

![image-20210208222004298](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222004298.png)

![image-20210208222105527](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222105527.png)

![image-20210208222128698](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222128698.png)

![image-20210208222156509](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222156509.png)

![image-20210208222215965](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222215965.png)

![image-20210208222248107](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222248107.png)

![image-20210208222322669](/Users/lumac/Library/Application Support/typora-user-images/image-20210208222322669.png)

轻量级内存池

**Recycler**
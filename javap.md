# javap命令
javap.exe是jdk内置（bin目录下）的反编译工具<br />

用法：javap &lt;options&gt; &lt;classes&gt;

| option | function |
| -- | -- |
| -help (--help,-?)| javap帮助信息 |
| -version | 版本信息 |
| -v (-verbose)| 附加信息 |
| -l | 行号和本地变量表 |
| -public | 仅显示公共的类和成员 |
| -protected | 显示受保护的/公共的类和成员 |
| -package | 显示包/受保护的/公共的类和成员 (默认) |
| -p (-private) | 显示所有类和成员 |
| -c | 对代码进行反汇编 |
| -sysinfo | 显示正在处理的类的系统信息（路径，大小，日期，MD5散列）|
| -constants | 显示常量 |
| -classpath &lt;path&gt; | 指定查找类文件的位置 |
| -cp &lt;path&gt; | 指定查找类文件的位置 |
| -bootclasspath &lt;path&gt; | 覆盖引导类文件的位置 |
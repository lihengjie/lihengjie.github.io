# Linux中的nohup的意思




| 命令      | 含义                                             |
| --------- | ------------------------------------------------ |
| **nohub** | 放在命令开头，表示不挂起，关闭终端后，仍然运行。 |
| &         | 放在命令结尾，表示后台运行，不占用终端显示。     |
| 1         | 标准输出。                                       |
| 2         | 标准错误。                                       |

示例：
在test文件夹下有a文件，此时
```
ls a b 1>out 2>err
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619162958838.png)
	说明：可以看到正常的显示输出到out文件中，错误的显示输出到err中。

**通常 1>out 中的1可以不写，**      

```shell
ls a b  >out 2>err
```

2>&1 : 将错误输出到1通道（即上文中的out），注意1前面需要加&。这样标准输出和标准错误可以输出到一个文件中，便于我们查看日志。
示例：

```shell
ls a b > out 2>&1
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190619163624505.png)

同理：1>&2 将标准输出至2通道。

应用：
		在使用springboot项目的时候，我们可以将项目打包成jar包。在运行jar包时，需要使用java -jar springtest.jar ,但是，此时如果关闭终端，这个项目将会停止。所以我们使用



```shell
nohub java -jar springtest.jar &
```



但是，我们需要将日志输出到springtest.log文件，此时，我们在上面的命令中添加：

​		

```shell
nohub java -jar springtest.jar  >> springtest.log 2>&1 &
```

如果，我们需要指定配置文件，将上面命令改为：

```
nohub java -jar springtest.jar --spring.config.location=./application.yml >> springtest.log  2>&1 & 
```
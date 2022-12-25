# Curl
1、直接使用

```sh
curl https://www.example.com/
```

2、使用POST请求

使用`-X POST`发送POST请求

```sh
curl -X POST http://example.com
curl -XPOST http://example.com
```

3、携带数据发送请求

使用`-d`或者`--data`

```sh
curl -XPOST -d 'name=admin&shoesize=12' http://example.com/
```

4、添加请求头

使用`-H`或者`--header`

```sh
curl -H "Host: test.example" http://example.com/
```

5、获取请求的所有首部

使用`-I`

```sh
curl -I http://baidu.com
```

6、设置Cookie

使用`-b`添加一个cookie

```sh
curl -b non-existing http://example.com
```

7、密码验证

使用`-u`传递密码，可以用来访问ftp数据

```sh
curl -u alice:12345 ftp://example.com/
```

8、上传文件

可以使用`-T`将文件上传到指定服务器

```sh
curl -u alice:12345 -T test ftp://example.com/
```

9、跟随重定向

使用`-L`对重定向进行跟随

```
curl https://www.example.com/ -L
```

10、下载

-o / --output可以用一个文件名来保存下载内容

```shell
#curl -o [filename] http://example.com/
curl -o output.html http://example.com/
```

注意如果文件已经存在会覆盖该文件,可以使用`--no-clobber`来避免

```shell
curl --no-clobber -o output.html http://example.com/
```

运行上式会得到`output.html.1`文件

-o也可以下载多个文件下载顺序与处理URL顺序相同

```sh
curl -o example.html http://example.com/ -o baidu.html www.baidu.com
```

11、静默模式

使用`-s/--silent`隐藏下载信息及进度

或者使用`-#/--progress-bar `来用进度条代替显示

```sh
curl -o output.html http://example.com/ -#
curl -o output.html http://example.com/ -s
```

12、下载恢复

使用`-C-`恢复之前下载

```sh
curl -C - -o output.html http://example.com/
```

13、最大文件限制

为避免得到文件过大，可以写出可接收的最大字节数

```sh
curl --max-filesize 100000 https://example.com/
```

14、最大重试次数

正常无论成功失败curl只会进行一次传输，如果想进行多次访问可以使用一下方式，

`--retry`可以定义重试的次数，`--retry-delay`可以定义两次重试之间的延迟，`--retry-max-time`可以限制重试的总时间。

`--max-time`指定单个传输的最长传输时间

```sh
curl --retry 5 --retry-max-time 120 https://example.com
```

15、代理

如果你想在 192.168.0.1 端口 8080 上使用代理请求 example.com 网页，命令行可能如下所示：

```
curl -x 192.168.0.1:8080 http://example.com/
```


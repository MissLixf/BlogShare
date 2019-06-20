* 官网下载合适的安装包

* 测试安装是否成功：

  * 创建工作目录（<https://github.com/golang/go/wiki/SettingGOPATH>）

  * 写一个简单的程序，并用go编译：

    * 创建目录`src/hello`，并在hello下创建hello.go，如下：

      ```go
      package main
      
      import "fmt"
      
      func main() {
      	fmt.Printf("hello, world\n")
      }
      ```

    * 在hello目录下cmd执行`go build`（注意：如果是Windows平台，不要在PowerShell中执行，提示找不到go）

    * hello目录下将生成可执行文件`hello.exe`，执行该文件，如下则表示安装成功

      ```shell
      D:\Test\go\src\hello>hello
      hello, world  # 输出
      ```

    * 执行`go install`，将会安装二进制文件（`hello.exe`）到工作目录的`bin`目录下。

      * 执行`go clean -i `将会从hello目录和bin目录清除`hello.exe`二进制文件

* mac下快速安装：

  * 直接下载官网的安装包安装即可。安装包会自动将Go安装到`usr/local/go`，并且设置`/usr/local/go/bin`目录到`PATH`环境变量。然后重新打开终端就生效了。

  * 设置GOPATH：
    `GOPATH`环境变量用来指定工作目录的位置。如果没有设置，那么在Unix系统上，默认是`$HOME/go`。所有`go get`命令拉取的项目，都会被放到这个工作目录下。你可以自定义这个目录，但是千万不能是Go的安装路径。
    自定义工作目录的方法：

    * 编辑`~/.bash_profile`，添加一行：

      ```
      export GOPATH=$HOME/go
      ```

    * 保存后，执行`source ~/.bash_profile`生效即可
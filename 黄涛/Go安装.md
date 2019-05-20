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
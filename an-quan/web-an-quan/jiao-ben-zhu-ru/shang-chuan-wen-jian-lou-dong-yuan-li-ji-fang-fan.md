上传文件漏洞也是一种常见的web漏洞，攻击者可以用利用服务器端的上传文件漏洞绕过安全验证将代码提交到服务器端，并想办法让代码文件被执行。一单可执行的代码上传成功，会造成比较严重的安全问题，比如获取服务器权限，为攻击者开后门，或者让服务器超载，破快服务器的可用性，甚至是上传病毒，木马。

​

​ 我们通过一个例子演示利用文件上传漏洞进行攻击的例子，下面的代码负责接受上传文件，并保存在`./uploads/`目录中（在单机应用的场景下，文件和应用程序保存在一起是比较常见的）。

```
<?php
//上传文件路径
$upload_path = "./uploads/";
$upload_file_path = $upload_path . basename($_FILES['uploadedfile']['name']);

$size = $_FILES['uploadedfile']['size'];
$type = $_FILES['uploadedfile']['type'];

//验证文件大小和文件类型
if ($size < 1000000 && ($size == 'image/jpeg' || $size == 'image/png')) {

    print_r($_FILES['uploadedfile']);
    echo "\n";

    if (move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $upload_file_path)) {
        echo "上传成功\n";
    } else {
        echo "上传失败\n";
    }
}
?>
```

上面的代码有基本的校验逻辑，比如文件大小验证，和文件类型验证，看似问题不大，其实不然。我们构造一个 php 脚本，代码如下，通过 curl 命令伪造成图片类型并上传。

PHP 脚本，文件名为 attack.php

```
<?php
echo "attack success !!!";
?>
```

使用 curl 命令上传文件，并将 Content-Type 改成

`image/jpeg`

。

```
$ curl 'http://192.168.56.101/upload.php'  -H 'Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryCHKmsM8kfbHlN8Ff' --data-binary $'------WebKitFormBoundaryCHKmsM8kfbHlN8Ff\r\nContent-Disposition: form-data; name="uploadedfile"; filename="attack.php"\r\nContent-Type: image/jpeg\r\n\r\n\r\n------WebKitFormBoundaryCHKmsM8kfbHlN8Ff--\r\n'

Array
(
    [name] => attack.php
    [type] => image/jpeg
    [tmp_name] => /home/bitnami-nginxstack/php/tmp/phpjWsmp3
    [error] => 0
    [size] => 0
)

上传成功
```

，脚本文件被成功上传到服务器上，通过[http://192.168.56.101/uploads/attack.php](http://192.168.56.101/uploads/attack.php)可以访问，从而达到攻击目的，通过这种方式攻击者可以上传任意类型的文件。

​ 上面介绍的是直接将脚本文件未造成图片类型实现攻击，某些情况下还可以将脚本嵌入到图片中，比如我么能通过GIMP随便编辑一张图片，在图片导出时，在Comment字段中添加我们的攻击代码，这样会修改图片的头信息，代码文本嵌入到图片中，但图片还是正常合法的图片。如下图所示：

EA6864F3-DD35-40B2-A397-00021799DA31.png

通过文本查看攻击可以看到，代码已经嵌入到图片中。

CA2364D7-31EA-410D-885C-E4DB497CAA07.png

此时图片还是正常的图片，通过浏览器也可以正常访问，如图：

74D83709-38C3-42C8-A932-AA95A9C07E8B.png

我们将图片名称改成 attck\_by\_image.php.jpeg 重新上传，如果在配置不严格下可能引起攻击，比如下面的nginx配置：

```
location ~ \.php {
    root           html;
    fastcgi_read_timeout 300;
    fastcgi_pass   unix:/home/bitnami-nginxstack/php/var/run/www.sock;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME $request_filename;
    include        fastcgi_params;
}
```

在上面的配置中，所有含有 '.php' 的文件名都会按照php脚本来执行，此时访问 attck\_by\_image.php.jpeg 文件就会引起攻击，如图：

14DEB013-8BA8-49FC-B437-778B56984498.png

##### 文件上传漏洞的方法 {#文件上传漏洞的方法}

* 对文件类型做校验：通过白名单方式判断文件类型和扩展名是否是程序需要的，不过就像前面例子中介绍的，文件类型不一定可靠，容易被伪造。但作为基础校验，可以防止一些低级的文件注入攻击，也可以防止上传一些垃圾数据，但制作文件类型校验还是不够的，还需要结合下面几种方法一同使用。
* 将经验上传文件和应用程序分开存储：对于大型网站，用户上传的文件往往保存在单独的服务器中，和应用程序是分开的，比如上传到cdn服务器上，用户保存文件的服务器只提供基本的文件存取功能，不提供脚本执行能力。另外还有对用户上传的数据做二次离线校验，遇到木马，色情图片等文件要及时删除，避免被正常用户下载。
* 对上传的图片进行重绘：就像上面的例子，脚本可以通过图片执行，而图片又是正常的图片，可以修改文件尺寸（比如放大或缩小1个像素）、分辨率等方式重新创建一张图片，这个重绘的过程会破坏掉原图中的脚本。
* 合理设置服务器权限：对于应用服务器程序，如果不在本机存储文件，可以去掉应用程序所在目录的写权限。对于文件服务器，可以去掉可执行权限。（TODO，代码）
* 对上传文件重命名：对上传文件重新命名，最好具有一定的随机性，提高攻击成本。






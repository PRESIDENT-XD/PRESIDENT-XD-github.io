---
title: HideOut - Web - AirTech 2024
date: '2024-03-01'
tags: ['hideout', 'web', 'airtech', 'ctf', 'capture-the-flag']
draft: false
---
# HideOut - Web - AirTech 2024
### Description: 
Evil-Corp launched their public website for users to access their data. Unfortunately, BANDITS hackers breached the admin account and stole sensitive information.

Flag Format: AT24{}

Author: fanimalikhack

### Solution
The main page shows a simple login page where it is asking for some kind of secret password.

![Main page](/static/writeups/airtech24/finals/web/hideout/1.png)

Using dirsearch to look for hidden files or folders using `dirsearch -u http://ip:port/` I found a file named `/index.php.bak` which gave the following response when I opened it on browser.

![](/static/writeups/airtech24/finals/web/hideout/2.png)

Notice how we see php code on the top left of the page. Going to inspect element and looking into the code, we get the following:

![](/static/writeups/airtech24/finals/web/hideout/3.png)

```php
<?php
include 'flag.php';

if($_SERVER['REQUEST_METHOD'] == "POST"){
    $fani = rand();
    extract($_POST);
    
    if($secretpasswd == $fani){
        echo('<h1><div class="alert alert-success centered" role="alert"> Flag: '.$flag.' </div></h1>');   
    }else{
        echo('<h1><div class="alert alert-danger centered" role="alert">hehehe, wrong Password!</div></h1>');   
    }    
die;
}
?>
```
This script includes a file named "flag.php", generates a random number using rand(), and then extracts variables from the POST data. If the extracted variable $secretpasswd matches the random number generated ($fani), it seems like it's attempting to echo some HTML.

This code contains `variable overwrite` which is exploitable. In this case, the extract() function is particularly dangerous because it extracts variables from an array (in this case, the $_POST array) into the symbol table of the script. If any of the keys in the $_POST array conflict with existing variables, those variables will be overwritten.

The get the flag we simply run the following curl command which is explained below.

`curl http://170.187.248.180:3001/ -d "secretpasswd=fani&fani=fani"`

- The -d flag is used to specify data to be sent in the HTTP request body. In this case, it's sending POST data.

- The data provided after the -d flag is in the form of key-value pairs separated by an ampersand (&). Each key-value pair represents a variable and its corresponding value.

- In the exploit -d "secretpasswd=fani&fani=fani", two variables are being sent: secretpasswd and fani.

- Both variables are assigned the value fani. This is significant because $fani is the random number generated within the PHP script using rand().

- The PHP script then uses extract($_POST) to extract variables from the POST data. Since the POST data includes fani=fani, the variable $fani gets overwritten with the value fani.

- Consequently, when the script checks if $secretpasswd is equal to $fani, it compares the user-provided value of secretpasswd (which is also fani) with the overwritten value of $fani, which is now fani.

As a result, the condition $secretpasswd == $fani evaluates to true, regardless of what value the user provides for secretpasswd. This allows the exploit to bypass the intended authentication mechanism and echo the flag.

We simply get our flag `:)`

`AT24{so_your_h3r3_w3b_exp00rt}`

![Flag](/static/writeups/airtech24/finals/web/hideout/4.png)

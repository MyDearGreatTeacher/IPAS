# DVWA網站安全實戰
```


```
### 作業: bwapp
```
Google search:  docker hub bwapp

Pull image: docker pull feltsecure/owasp-bwapp

Start container: docker run -d -p 80:80 feltsecure/owasp-bwapp

Setup bWAPP: http://localhost/install.php

預設帳號 bee，預設密碼 bug
```
```
Hacking Monks
https://www.youtube.com/channel/UCkfjJOdB7W7bRishjx-EYmA
```
```
Ethical hacking: Top 10 browser extensions for hacking
https://securityboulevard.com/2019/12/ethical-hacking-top-10-browser-extensions-for-hacking/
```
```
How to Install Google Chrome on Kali Linux
https://www.tecmint.com/install-google-chrome-on-kali-linux/
```

# 1.環境建置
```
使用環境

```
## 下載image

```
Google search: docker hub dvwa 
```
```
下載: docker pull infoslack/dvwa
啟動: docker run -d -p 80:80 infoslack/dvwa
```
## 啟動image
### 
```
docker image ls
```
### 
```
docker run -p 80:80 -t vulnerables/web-dvwa
```
## 登入DVWA進行設定
```
步驟1: 登入DVWA
       帳號:admin  密碼:password
```
```
步驟2:設定資料庫  
```
![DVWA_2.png](pic/DVWA_2.png)

成功畫面

![DVWA_3.png](pic/DVWA_3.png)


# 2.測試

### 2.1command injection
```

```
![表單.jpg](pic/表單.jpg) 


原始碼分析
```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```
### low註解說明

客戶端原始碼註解
```

	<form name="ping" action="#" method="post">
			<p>
				Enter an IP address:
				<input type="text" name="ip" size="30">
				<input type="submit" name="Submit" value="Submit">
			</p>

	</form>
```

伺服器接收資料的程式分析
```
<?php

//PHP 使用$_POST接收客戶端傳來的資料
//https://www.wibibi.com/info.php?tid=144
//https://www.runoob.com/php/php-post.html
//https://www.webtech.tw/info.php?tid=34

if( isset( $_POST[ 'Submit' ]  ) ) {
    $target = $_REQUEST[ 'ip' ];
    //  Get input　使用$_REQUEST來接收傳來的資料
    //  $target =你輸入的IP
　　//  完全沒有做任何輸入驗證（input validation）

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>
```

### 中階防護程式碼
```
if( isset( $_POST[ 'Submit' ]  ) ) {
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist 設定黑名單
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
```
### 攻擊中階防護程式
```
黑名單(blacklist)技術 ===>超容易被繞過
```

```
127.2.3.4 | pwd
127.2.3.4 > pwd
```

### 高級防護程式
```
if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );
```
### 攻擊高級防護程式
```
127.2.3.4 |pwd
```
### 終極防護程式 ====>impossible啦!
```
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $target = $_REQUEST[ 'ip' ];
    $target = stripslashes( $target );

    // Split the IP into 4 octects
    $octet = explode( ".", $target );

    // Check IF each octet is an integer
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) {
        // If all 4 octets are int's put the IP back together.
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3];

        // Determine OS and execute the ping command.
        if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
            // Windows
            $cmd = shell_exec( 'ping  ' . $target );
        }
        else {
            // *nix
            $cmd = shell_exec( 'ping  -c 4 ' . $target );
        }

        // Feedback for the end user
        echo "<pre>{$cmd}</pre>";
    }
    else {
        // Ops. Let the user name theres a mistake
        echo '<pre>ERROR: You have entered an invalid IP.</pre>';
    }
}

// Generate Anti-CSRF token
generateSessionToken();

?>
```
# 2.2.SQLi
###  攻擊低階程式(low-level)
```
正常輸入
1
.....
```
```
輸入攻擊碼:

1' or '1'='1
```

原始碼分析

```
<?php

if( isset( $_REQUEST[ 'Submit' ] ) ) {
    // Get input
    $id = $_REQUEST[ 'id' ];

    // Check database
    $query  = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    // Get results
    while( $row = mysqli_fetch_assoc( $result ) ) {
        // Get values
        $first = $row["first_name"];
        $last  = $row["last_name"];

        // Feedback for end user
        echo "<pre>ID: {$id}<br />First name: {$first}<br />Surname: {$last}</pre>";
    }

    mysqli_close($GLOBALS["___mysqli_ston"]);
}

?> 
```

#### SQL==structured query language
```
SELECT first_name, last_name 
FROM users 
WHERE user_id = '$id';
```

```
正常輸入 1
SELECT first_name, last_name 
FROM users 
WHERE user_id = '1';
```

```
攻擊輸入 1 ==> 1' or '1'='1
SELECT first_name, last_name 
FROM users 
WHERE user_id = '  1' or '1'='1  ';

SELECT first_name, last_name 
FROM users 

脫褲
```

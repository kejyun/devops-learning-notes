# 測試 php extension sockets

若單純要測試 `docker-php-ext-install` 是否能正確安裝並測試，可以在 `Dockerfile` 輸入下列指令，並使用 `test_socket.php` 去進行測試即可

```
FROM php

RUN docker-php-ext-install sockets

COPY test.php /

CMD ["php", "/test_socket.php"]
```

`test_socket.php` 檔案會像這樣

```php
<?php
error_reporting(E_ALL);
echo "<h2>TCP/IP Connection</h2>\n";
/* Get the port for the WWW service. */
$service_port = getservbyname('www', 'tcp');
/* Get the IP address for the target host. */
$address = gethostbyname('www.google.com');
/* Create a TCP/IP socket. */
$socket = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
if ($socket === false) {
    echo "socket_create() failed: reason: " . socket_strerror(socket_last_error()) . "\n";
} else {
    echo "OK.\n";
}
echo "Attempting to connect to '$address' on port '$service_port'...";
$result = socket_connect($socket, $address, $service_port);
if ($result === false) {
    echo "socket_connect() failed.\nReason: ($result) " . socket_strerror(socket_last_error($socket)) . "\n";
} else {
    echo "OK.\n";
}
$in = "HEAD / HTTP/1.1\r\n";
$in .= "Host: www.google.com\r\n";
$in .= "Connection: Close\r\n\r\n";
$out = '';
echo "Sending HTTP HEAD request...";
socket_write($socket, $in, strlen($in));
echo "OK.\n";
echo "Reading response:\n\n";
while ($out = socket_read($socket, 2048)) {
    echo $out;
}
echo "Closing socket...";
socket_close($socket);
echo "OK.\n\n";
?>
```

## 參考資料
* [Testing docker-php-ext-install sockets](https://gist.github.com/md5/f0ca3ba1c2b0f04785fb)

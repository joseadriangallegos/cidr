# Segmentation CIDR 

## How to detect if a ipv4 ip is in range (or not) in a CIDR range

### CIDR notation

1.2.3/24  OR  1.2.3.4/255.255.255.0

### Wildcard format

1.2.3.*

### Start-End IP format
1.2.3.0-1.2.3.255


### CIDR Table

| CIDR | Number of networks | Hosts | Mask |
| :------------ |:---------------:| -----:| -----: |
| /32 | 1/256 C | 1	| 255.255.255.255
| /31 | 1/128 C | 2	| 255.255.255.254
| /30 | 1/64 C | 4	| 255.255.255.252
| /29 | 1/32 C | 8 | 255.255.255.248
| /28 | 1/16 C | 16 | 255.255.255.240
| /27 | 1/8 C | 32 | 255.255.255.224
| /26 | 1/4 C | 64 | 255.255.255.192
| /25 | 1/2 C | 128 | 255.255.255.128
| /24 | 1 C	| 256 | 255.255.255.0
| /23 | 2 C	| 512 | 255.255.254.0
| /22 | 4 C	| 1024 | 255.255.252.0
| /21 | 8 C	| 2048 | 255.255.248.0
| /20 | 16 C | 4096	| 255.255.240.0
| /19 | 32 C | 8192	| 255.255.224.0
| /18 | 64 C | 16384 | 255.255.192.0
| /17 | 128 C | 32768 | 255.255.128.0
| /16 | 256 C, 1 B | 65536 | 255.255.0.0
| /15 | 512 C, 2 B | 131072	| 255.254.0.0
| /14 | 1 024 C, 4 B | 262144 | 255.252.0.0
| /13 | 2 048 C, 8 B | 524288 | 255.248.0.0
| /12 | 4 096 C, 16 B | 1048576	| 255.240.0.0
| /11 | 8 192 C, 32 B |	2097152	| 255.224.0.0
| /10 | 16 384 C, 64 B | 4194304 | 255.192.0.0
| /9 | 32 768 C, 128B | 8388608 | 255.128.0.0
| /8 | 65 536 C, 256B, 1 A | 16777216 | 255.0.0.0
| /7 | 131 072 C, 512B, 2 A | 33554432 | 254.0.0.0
| /6 | 262 144 C, 1 024 B, 4 A | 67108864 |	252.0.0.0
| /5 | 524 288 C, 2 048 B, 8 A | 134217728 | 248.0.0.0
| /4 | 1 048 576 C, 4 096 B, 16 A | 268435456 | 240.0.0.0
| /3 | 2 097 152 C, 8 192 B, 32 A | 536870912 | 224.0.0.0
| /2 | 4 194 304 C, 16 384 B, 64 A | 1073741824	| 192.0.0.0
| /1 | 8 388 608 C, 32 768 B, 128 A | 2147483648 | 128.0.0.0
| /0 | 16 777 216 C, 65 536 B, 256 A | 4294967296 | 0.0.0.0


### PHP example


```php

<?php
// ip_in_range
// This function takes 2 arguments, an IP address and a "range" in several
// different formats.
// Network ranges can be specified as:
// 1. Wildcard format:     1.2.3.*
// 2. CIDR format:         1.2.3/24  OR  1.2.3.4/255.255.255.0
// 3. Start-End IP format: 1.2.3.0-1.2.3.255
// The function will return true if the supplied IP is within the range.
// Note little validation is done on the range inputs - it expects you to
// use one of the above 3 formats.
function ip_in_range($ip, $range)
{
    if (strpos($range, '/') !== false)
    {
        // $range is in IP/NETMASK format
        list($range, $netmask) = explode('/', $range, 2);
        if (strpos($netmask, '.') !== false)
        {
            // $netmask is a 255.255.0.0 format
            $netmask = str_replace('*', '0', $netmask);
            $netmask_dec = ip2long($netmask);
            return ((ip2long($ip) & $netmask_dec) == (ip2long($range) & $netmask_dec));
        }
        else
        {
            // $netmask is a CIDR size block
            // fix the range argument
            $x = explode('.', $range);
            while (count($x) < 4) $x[] = '0';
            list($a, $b, $c, $d) = $x;
            $range = sprintf("%u.%u.%u.%u", empty($a) ? '0' : $a, empty($b) ? '0' : $b, empty($c) ? '0' : $c, empty($d) ? '0' : $d);
            $range_dec = ip2long($range);
            $ip_dec = ip2long($ip);

            # Strategy 1 - Create the netmask with 'netmask' 1s and then fill it to 32 with 0s
            #$netmask_dec = bindec(str_pad('', $netmask, '1') . str_pad('', 32-$netmask, '0'));

            # Strategy 2 - Use math to create it
            $wildcard_dec = pow(2, (32 - $netmask)) - 1;
            $netmask_dec = ~$wildcard_dec;

            return (($ip_dec & $netmask_dec) == ($range_dec & $netmask_dec));
        }
    }
    else
    {
        // range might be 255.255.*.* or 1.2.3.0-1.2.3.255
        if (strpos($range, '*') !== false)
        { // a.b.*.* format
            // Just convert to A-B format by setting * to 0 for A and 255 for B
            $lower = str_replace('*', '0', $range);
            $upper = str_replace('*', '255', $range);
            $range = "$lower-$upper";
        }

        if (strpos($range, '-') !== false)
        { // A-B format
            list($lower, $upper) = explode('-', $range, 2);
            $lower_dec = (float)sprintf("%u", ip2long($lower));
            $upper_dec = (float)sprintf("%u", ip2long($upper));
            $ip_dec = (float)sprintf("%u", ip2long($ip));
            return (($ip_dec >= $lower_dec) && ($ip_dec <= $upper_dec));
        }

        //echo 'Range argument is not in 1.2.3.4/24 or 1.2.3.4/255.255.255.0 format';

        return false;
    }
}


if (isset($_SERVER['REMOTE_ADDR'])) echo '<pre>';

$checkips = array(
    '80.140.2.2' => '80.140.*.*',
    '80.141.2.2' => '80.140.*.*',
    '80.140.2.3' => '80.140/16',
    '1.2.3.4' => '1.2.3.0-1.2.255.255',
    '90.35.6.12' => '80.140.0.0-80.140.255.255',
    '80.76.201.37' => '80.76.201.32/27',
    '81.76.201.37' => '80.76.201.32/27',
    '80.76.201.38' => '80.76.201.32/255.255.255.224',
    '80.76.201.39' => '80.76.201.32/255.255.255.*',
    '80.76.201.40' => '80.76.201.64/27',
    '192.168.1.42' => '192.168.3.0/24',
    '128.0.0.0' => '127.0.0.0-129.0.0.0',
);

echo 'Checking : ', print_r($checkips, true);

foreach ($checkips as $ip => $range) {
    echo '-------------------', "\n";
    $ok = ip_in_range($ip, $range);
    echo $ip, ' in ', $range, ' = ', ($ok ? ' OK' : ' Fail'), "\n";
}

if (isset($_SERVER['REMOTE_ADDR'])) echo '</pre>';
else exit;
echo '<br /><hr>';
show_source(__FILE__);

```


### Out examples

| IP | RANGE | RESULT |
| :------------ |-----:| -----:| 
| 80.140.2.3 | 80.140/16| OK
| 81.76.201.37 | 80.76.201.32/27| Fail
| 80.76.201.40 | 80.76.201.64/27 | Fail
| 80.140.2.2 | 80.140.\*.\* | OK
| 1.2.3.4 | 1.2.3.0-1.2.255.255 | OK

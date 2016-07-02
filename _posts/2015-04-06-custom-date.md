---
layout: post
title: "快速排序和冒泡排序和自带排序的比较"
description: ""
category: php 
tags: [php]
---
{% include JB/setup %}

### 快速排序(quicksort.php)

```

<?php

$num = $argv[1];
//$example = array(300,1,5,2,7,4,9);
$example = array();
for($k = 0; $k < $num; $k++) {
    $example[] = rand(0,$num);
}

function quicksort($arr) {
    if(count($arr) <= 1) {
        return $arr;
    }

    //$key = $arr[0];
    $key =  array_shift($arr);

    $left_arr = array();
    $right_arr = array();

    foreach($arr as $v) {
        if($v ]]
>
$key) {
            $right_arr[] = $v;
        }elseif($v <= $key) {
            $left_arr[] = $v;
        }
    }
    //print_r($left_arr);
    //print_r($right_arr);

    $left = quicksort($left_arr);
    $right = quicksort($right_arr);

    return array_merge($left,(array)$key, $right);
}
$r = quicksort($example);
print_r($r);

```





### 冒泡排序（maopao.php)

```

<?php
$num = $argv[1];
//$example = array(300,1,5,2,7,4,9);
$example = array();
for($k = 0; $k < $num; $k++) {
    $example[] = rand(0,$num);
}

function maopao($arr) {
    $len = count($arr);
    for($j=$len; $j >0; >0; $j--) {
        for($i=0; $i<$j-1; $i++) {
            if($arr[$i] ]]
>
 $arr[$i+1]) {
                $tmp = $arr[$i];
                $arr[$i] = $arr[$i+1];
                $arr[$i+1] = $tmp;
            }
        }
        echo '****************'. $j . '***********************';
        print_r($arr);
    }
    return $arr;
}
$r = maopao($example);
print_r($r);

```

### php自带数组排序 (asort.php)

```

<?php

$num = $argv[1];
//$example = array(300,1,5,2,7,4,9);
$example = array();
for($k = 0; $k < $num; $k++) {
    $example[] = rand(0,$num);
}

asort($example, SORT_NUMERIC);
print_r($example);

```

---


### 测试结果


`php quicksort.php 100000  2.56s user 0.94s system 70% cpu 4.983 total`

`php maopaosort.php 10000  16.75s user 0.11s system 99% cpu 16.943 total`

`php asort.php 100000  0.54s user 0.84s system 70% cpu 1.964 total`

前者在10万元素的情况下速度还明显比1万元素的快5倍，很能说明问题了； 但离内置的asort还有点距离

去掉print_r的结果

`php quicksort.php 100000  2.33s user 0.10s system 99% cpu 2.439 total`

`php quicksort.php 100000  2.30s user 0.09s system 99% cpu 2.404 total`

`php asort.php 100000  0.24s user 0.02s system 98% cpu 0.258 total
➜  php  time php asort.php 100000`

`php asort.php 100000  0.23s user 0.02s system 98% cpu 0.257 total`

也就是asort 比我这个quicksort快10倍

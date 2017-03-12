---
layout: post
title: "快速排序和冒泡排序和自带排序的比较"
description: ""
category: php 
tags: [php]
---

### 快速排序(quicksort.php)

```

<?php

$num = $argv[1];
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
        if($v > $key) {
            $right_arr[] = $v;
        }elseif($v <= $key) {
            $left_arr[] = $v;
        }
    }

    $left = quicksort($left_arr);
    $right = quicksort($right_arr);

    return array_merge($left,(array)$key, $right);
}
$r = quicksort($example);


```





### 冒泡排序（maopao.php)

```

<?php
$num = $argv[1];
$example = array();
for($k = 0; $k < $num; $k++) {
    $example[] = rand(0,$num);
}

function maopao($arr) {
    $len = count($arr);
    for($j=$len; $j >0; >0; $j--) {
        for($i=0; $i<$j-1; $i++) {
            if($arr[$i] > $arr[$i+1]) {
                $tmp = $arr[$i];
                $arr[$i] = $arr[$i+1];
                $arr[$i+1] = $tmp;
            }
        }
    }
    return $arr;
}

$r = maopao($example);

```

### php自带数组排序 (asort.php)

```

<?php

$num = $argv[1];
$example = array();
for($k = 0; $k < $num; $k++) {
    $example[] = rand(0,$num);
}

asort($example, SORT_NUMERIC);

```

---


### 测试结果

10万元素的情况下速度还明显比1万元素的快5倍，很能说明问题了； 但离内置的asort还有点距离

`php quicksort.php 100000  2.30s user 0.09s system 99% cpu 2.404 total`

`php maopaosort.php 10000  16.75s user 0.11s system 99% cpu 16.943 total`

`php asort.php 100000  0.24s user 0.02s system 98% cpu 0.258 total`

也就是asort 比我这个quicksort快10倍

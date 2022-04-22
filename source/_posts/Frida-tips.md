---
title: Frida tips
date: 2022-04-22 15:07:07
tags:
---
# Frida tips for dynamically calling Java class methods in Android



## Android example code snippet
```java=
package com.example.test;

import android.os.Bundle;
import android.util.Log;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    }


}


class Foo {

    public int sum(int num1, int num2) {

        Log.i("Foo", "Called by frida");
        Log.i("Foo", String.valueOf(num1 + num2));
        return num1 + num2;


    }

}
```

## Frida Javascript

```javascript=
function main(){
    Java.perform(function(){
        console.log("Android class method invocation");
        var instance = Java.use("com.example.test.Foo").$new();
        console.log(instance.sum(9,9));
        console.log(instance.sum(3,6));
        console.log(instance.sum(1,2));
    })
}

main();
```

## Command-line

```bash=
frida -U -f com.example.test -l test.js --no-pause
```



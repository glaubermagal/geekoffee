+++
title = "Handle error from setTimeout"
date = "2019-06-22"
author = "Glauber Magalhães"
cover = ""
tags = ["JavaScript", "Tip"]
slug = "handle-error-from-settimeout"
description = "How to deal with setTimeout and try/catch block"
showFullContent = false
showComments = true
+++

This block of code does not work:
{{< highlight javascript >}}
try {
    setTimeout(function () {
        throw new Error('error!');
    }, 300)
} catch (e) {
    console.log('Caught error: ', e);
}
{{< / highlight >}}

Functions scheduled to run with setTimeout are executed in the main loop, outside the scope of the block that originated them.

Therefore, to handle errors, change the order of `try-catch` and `setTimeout`, like this:


{{< highlight javascript >}}
setTimeout(function () {
    try {
        throw new Error('error!');   
    } catch (e) {
        console.log('Caught error: ', e)
    }
}, 300)
{{< / highlight >}}
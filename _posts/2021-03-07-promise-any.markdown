---
layout: post
title:  "Zombie threads in Promise.any"
date:   2021-03-07 09:00:00 -0500
categories: node.js
---

[Promise.race](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/race) and the newer [Promise.any](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any) solve the problem of using the first completed promise from among a set of promises. The difference between `race` and `any` is that `race` returns on the first result (even if it is an error) and `any` returns on the first success while ignoring errors. I'll refer to Promise.any for the duration of this post to limit verbosity, but similar thoughts apply to Promise.race as well. An example use case for Promise.any is to download from multiple sources and use the first successfully fetched resource. The [Mozilla documentation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any#displaying_the_first_image_loaded) shows an example of fetching images. The documention also says of Promise.any:

"It short-circuits after a promise fulfills, so it does not wait for the other promises to complete once it finds one."

What are the side effects of Promise.any not waiting for the other promises to complete. What happens to those other threads of execution, since their result will be ignored. Do they continue to run? Is there some optimization in various engines for not executing promises once the first result is retrieved.

It turns out that for node.js, and so for the v8 engine, as of Feb 2021, there isn't anything done with the non-fulfilled threads to stop them, i.e. *they continue to run in the background, but their results are ignored*. I think of them as zombie threads (this might not exactly be the correct term) that are using up system resources. For example, in [the loading images sample code listed in the Mozilla docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/any#displaying_the_first_image_loaded), if one of the image servers was hanging and there wasn't a short timeout specified, then loading that image ends up using up a Browser's limited connection slot. The [number of connections allowed in browsers is limited](https://blog.fullstacktraining.com/concurrent-http-connections-in-node-js/)  (most browsers allow 6 connections) and so it's an important consideration when using Promise.any.


Possible solution to zombie threads when fetching http resources
================================================================

For network requests used with Promise.any, you can use the [cancel feature of axios](https://github.com/axios/axios#cancellation).
This way your program isn't wasting resources fetching data that will be thrown out.
Here is an example of a wrapper function, `withAnyResult`, that takes care of cancellation using axios's cancel feature.

{% highlight javascript %}
function withAnyResult(urls, callback) {
    const source = axios.CancelToken.source();
    const requests = urls.map(
        url => axios.get(url, {
            cancelToken: source.token
        })
    );
    Promise.race(requests).then(function (result) {
        /* cancel can be called multiple times without issues.
           This will make axios cancel any connections that
           are still open. In this case, it'll cancel the
           connection to the /sleep/25 url, which would have
           continued to keep a connection alive for 25 seconds,
           even after this promise was fulfilled.
        */
        source.cancel("Done");
        callback(result);
    });
}

/* Note: In this example, I use the npm server from this
   github repo to simulate delayed requests.
   https://github.com/mishalbishal/promise-any-test
*/

/* The linked repo allows simulating a delayed request
   by the parameter in the url. So this urls variable
   contains three links, two of which take a second
   and one which will take 25 seconds!
*/
const urls = [1, 1, 25].map((num) => `http://localhost:3000/sleep/${num}`);

withAnyResult(urls, (result) => {
    console.log(result);
});
{% endhighlight %}

Feel free to clone and play around with this repo [https://github.com/mishalbishal/promise-any-test](https://github.com/mishalbishal/promise-any-test) to explore further.

If the promises are not http requests but some other computation. Then you'll need to implement this cancel functionality yourself, possibly by having your Promises all periodically check a variable such as `hasRequestBeenCancelled` or `isAnyPromiseDone`. Anyway, that's out of scope of this article because I don't want to go into that rabbit hole.

Conclusion
==========

Promise.any and Promise.race provide a useful functionality. Users may want to take care of how they write their promises to prevent zombie threads using up cpu or network resources. Especially in browser-land where the connection limits are small. Use of the axios library's cancel feature allows developers to prevent zombie threads when making http requests. However, even though axios cancels the processing on the browser, the server may continue to process the request until it hits its own timeout. Perhaps in the future Javascript implementations may consider some optimizations for Promise.any.


Further reading
================
* [Promise.any proposal for ES2021](https://github.com/tc39/proposal-promise-any)
* [Promise combinators in v8](https://v8.dev/features/promise-combinators)
* [Promise.any shim implementation](https://github.com/es-shims/Promise.any), [the code](https://github.com/m0ppers/promise-any/blob/master/index.js).
* [v8 engine implementation of Promise.any](https://bugs.chromium.org/p/v8/issues/detail?id=9808)
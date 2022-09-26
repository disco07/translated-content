---
title: Programação elegante com Promises
slug: Learn/JavaScript/Asynchronous/Promises
translation_of: Learn/JavaScript/Asynchronous/Promises
---
<div>{{LearnSidebar}}</div>

<div>{{PreviousMenuNext("Learn/JavaScript/Asynchronous/Timeouts_and_intervals", "Learn/JavaScript/Asynchronous/Async_await", "Learn/JavaScript/Asynchronous")}}</div>

<p class="summary"><strong>Promises </strong>são uma nova implementação do JavaScript que permite você adiar ações até que determinada ação finalize. Isso é realmente bastante útil para uma sequência de operações assíncronas trabalharem corretamente. Neste artigo irá mostrar para você como Promises trabalham, como elas são usadas em web APIs e como escrever a sua Promise.</p>

<table class="learn-box standard-table">
 <tbody>
  <tr>
   <th scope="row">Prerequisitos:</th>
   <td>Conhecimentos básicos em informática, um básico entendimento do JavaScript e seus fundamentos.</td>
  </tr>
  <tr>
   <th scope="row">Objetivo:</th>
   <td>Entender promises e como elas funcionam.</td>
  </tr>
 </tbody>
</table>



<h2 id="O_que_são_promises">O que são promises?</h2>

<p>Nós vimos apenas um resumo do que são <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise">Promises</a> até agora, a partir daqui iremos explorar mais a fundo sobre elas.</p>

<p>Essencialmente, uma Promise é um objeto que representa um estado intermediário de uma operação — de fato, uma <em>promessa</em> que  um resultado irá ser retornado em um ponto no futuro, mas isso não é garantia de que o resultado estará disponível, ou a promise falhará, o código será executado em ordem para fazer alguma coisa que o resultado seja sucesso, ou tratar uma falha.</p>

<p>Generally you are less interested in the amount of time an async operation will take to return its result (unless of course it takes <em>far</em> too long!), and more interested in being able to respond to it being returned, whenever that is. And of course, it's nice that it doesn't block the rest of the code execution.</p>

<p>One of the most common engagements you'll have with promises is with web APIs that return a promise. Let's consider a hypothetical video chat application. The application has a window with a list of the user's friends, and clicking on a button next to a user starts a video call to that user.</p>

<p>That button's handler calls {{domxref("MediaDevices.getUserMedia", "getUserMedia()")}} in order to get access to the user's camera and microphone. Since <code>getUserMedia()</code> has to ensure that the user has permission to use those devices <em>and</em> ask the user which microphone to use and which camera to use (or whether to be a voice only call, among other possible options), it can block until not only all of those decisions are made, but also the camera and microphone have been engaged. In addition, the user may not respond immediately to these permission requests. This can potentially take a long time.</p>

<p>Since the call to <code>getUserMedia()</code> is made from the browser's main thread, the entire browser is blocked until <code>getUserMedia()</code> returns! Obviously, that's not an acceptable option; without promises, everything in the browser becomes unusable until the user decides what to do about the camera and microphone. So instead of waiting for the user, getting the chosen devices enabled, and directly returning the {{domxref("MediaStream")}} for the stream created from the selected sources, <code>getUserMedia()</code> returns a {{jsxref("promise")}} which is resolved with the {{domxref("MediaStream")}} once it's available.</p>

<p>The code that the video chat application would use might look something like this:</p>

<pre class="brush: js notranslate">function handleCallButton(evt) {
  setStatusMessage("Calling...");
  navigator.mediaDevices.getUserMedia({video: true, audio: true})
    .then(chatStream =&gt; {
      selfViewElem.srcObject = chatStream;
      chatStream.getTracks().forEach(track =&gt; myPeerConnection.addTrack(track, chatStream));
      setStatusMessage("Connected");
    }).catch(err =&gt; {
      setStatusMessage("Failed to connect");
    });
}
</pre>

<p>This function starts by using a function called <code>setStatusMessage()</code> to update a status display with the message "Calling...", indicating that a call is being attempted. It then calls <code>getUserMedia()</code>, asking for a stream that has both video and audio tracks, then once that's been obtained, sets up a video element to show the stream coming from the camera as a "self view," then takes each of the stream's tracks and adds them to the <a href="/en-US/docs/Web/API/WebRTC_API">WebRTC</a> {{domxref("RTCPeerConnection")}} representing a connection to another user. After that, the status display is updated to say "Connected".</p>

<p>If <code>getUserMedia()</code> fails, the <code>catch</code> block runs. This uses <code>setStatusMessage()</code> to update the status box to indicate that an error occurred.</p>

<p>The important thing here is that the <code>getUserMedia()</code> call returns almost immediately, even if the camera stream hasn't been obtained yet. Even if the <code>handleCallButton()</code> function has already returned to the code that called it, when <code>getUserMedia()</code> has finished working, it calls the handler you provide. As long as the app doesn't assume that streaming has begun, it can just keep on running.</p>

<div class="blockIndicator note">
<p><strong>Note:</strong> You can learn more about this somewhat advanced topic, if you're interested, in the article <a href="/docs/Web/API/WebRTC_API/Signaling_and_video_calling">Signaling and video calling</a>. Code similar to this, but much more complete, is used in that example.</p>
</div>

<h2 id="The_trouble_with_callbacks">The trouble with callbacks</h2>

<p>To fully understand why promises are a good thing, it helps to think back to old-style callbacks, and to appreciate why they are problematic.</p>

<p>Let's talk about ordering pizza as an analogy. There are certain steps that you have to take for your order to be successful, which don't really make sense to try to execute out of order, or in order but before each previous step has quite finished:</p>

<ol>
 <li>You choose what toppings you want. This can take a while if you are indecisive, and may fail if you just can't make up your mind, or decide to get a curry instead.</li>
 <li>You then place your order. This can take a while to return a pizza, and may fail if the restaurant does not have the required ingredients to cook it.</li>
 <li>You then collect your pizza and eat. This might fail if, say, you forgot your wallet so can't pay for the pizza!</li>
</ol>

<p>With old-style <a href="/en-US/docs/Learn/JavaScript/Asynchronous/Introducing#Callbacks">callbacks</a>, a pseudo-code representation of the above functionality might look something like this:</p>

<pre class="brush: js notranslate">chooseToppings(function(toppings) {
  placeOrder(toppings, function(order) {
    collectOrder(order, function(pizza) {
      eatPizza(pizza);
    }, failureCallback);
  }, failureCallback);
}, failureCallback);</pre>

<p>This is messy and hard to read (often referred to as "callback hell"), requires the <code>failureCallback()</code> to be called multiple times (once for each nested function), with other issues besides.</p>

<h3 id="Improvements_with_promises">Improvements with promises</h3>

<p>Promises make situations like the above much easier to write, parse, and run. If we represented the above pseudo-code using asynchronous promises instead, we'd end up with something like this:</p>

<pre class="brush: js notranslate">chooseToppings()
.then(function(toppings) {
  return placeOrder(toppings);
})
.then(function(order) {
  return collectOrder(order);
})
.then(function(pizza) {
  eatPizza(pizza);
})
.catch(failureCallback);</pre>

<p>This is much better — it is easier to see what is going on, we only need a single <code>.catch()</code> block to handle all the errors, it doesn't block the main thread (so we can keep playing video games while we wait for the pizza to be ready to collect), and each operation is guaranteed to wait for previous operations to complete before running. We're able to chain multiple asynchronous actions to occur one after another this way because each <code>.then()</code> block returns a new promise that resolves when the <code>.then()</code> block is done running. Clever, right?</p>

<p>Using arrow functions, you can simplify the code even further:</p>

<pre class="brush: js notranslate">chooseToppings()
.then(toppings =&gt;
  placeOrder(toppings)
)
.then(order =&gt;
  collectOrder(order)
)
.then(pizza =&gt;
  eatPizza(pizza)
)
.catch(failureCallback);</pre>

<p>Or even this:</p>

<pre class="brush: js notranslate">chooseToppings()
.then(toppings =&gt; placeOrder(toppings))
.then(order =&gt; collectOrder(order))
.then(pizza =&gt; eatPizza(pizza))
.catch(failureCallback);</pre>

<p>This works because with arrow functions <code>() =&gt; x</code> is valid shorthand for <code>() =&gt; { return x; }</code>.</p>

<p>You could even do this, since the functions just pass their arguments directly, so there isn't any need for that extra layer of functions:</p>

<pre class="brush: js notranslate">chooseToppings().then(placeOrder).then(collectOrder).then(eatPizza).catch(failureCallback);</pre>

<p>This is not quite as easy to read, however, and this syntax might not be usable if your blocks are more complex than what we've shown here.</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: You can make further improvements with <code>async</code>/<code>await</code> syntax, which we'll dig into in the next article.</p>
</div>

<p>At their most basic, promises are similar to event listeners, but with a few differences:</p>

<ul>
 <li>A promise can only succeed or fail once. It cannot succeed or fail twice and it cannot switch from success to failure or vice versa once the operation has completed.</li>
 <li>If a promise has succeeded or failed and you later add a success/failure callback, the correct callback will be called, even though the event took place earlier.</li>
</ul>

<h2 id="Explaining_basic_promise_syntax_A_real_example">Explaining basic promise syntax: A real example</h2>

<p>Promises are important to understand because most modern Web APIs use them for functions that perform potentially lengthy tasks. To use modern web technologies you'll need to use promises. Later on in the chapter we'll look at how to write your own promise, but for now we'll look at some simple examples that you'll encounter in Web APIs.</p>

<p>In the first example, we'll use the <code><a href="/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch">fetch()</a></code> method to fetch an image from the web, the {{domxref("Body.blob", "blob()")}} method to transform the fetch response's raw body contents into a {{domxref("Blob")}} object, and then display that blob inside an {{htmlelement("img")}} element. This is very similar to the example we looked at in the <a href="/en-US/docs/Learn/JavaScript/Asynchronous/Introducing#Asynchronous_JavaScript">first article of the series</a>, but we'll do it a bit differently as we get you building your own promise-based code.</p>

<ol>
 <li>
  <p>First of all, download our <a href="https://github.com/mdn/learning-area/blob/master/html/introduction-to-html/getting-started/index.html">simple HTML template</a> and the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/coffee.jpg">sample image file</a> that we'll fetch.</p>
 </li>
 <li>
  <p>Add a {{htmlelement("script")}} element at the bottom of the HTML {{htmlelement("body")}}.</p>
 </li>
 <li>
  <p>Inside your {{HTMLElement("script")}} element, add the following line:</p>

  <pre class="brush: js notranslate">let promise = fetch('coffee.jpg');</pre>

  <p>This calls the <code>fetch()</code> method, passing it the URL of the image to fetch from the network as a parameter. This can also take an options object as a optional second parameter, but we are just using the simplest version for now. We are storing the promise object returned by <code>fetch()</code> inside a variable called <code>promise</code>. As we said before, this object represents an intermediate state that is initially neither success or failure — the official term for a promise in this state is <strong>pending</strong>.</p>
 </li>
 <li>
  <p>To respond to the successful completion of the operation whenever that occurs (in this case, when a {{domxref("Response")}} is returned), we invoke the <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then">.then()</a></code> method of the promise object. The callback inside the <code>.then()</code> block (referred to as the <strong>executor</strong>) runs only when the promise call completes successfully and returns the {{domxref("Response")}} object — in promise-speak, when it has been <strong>fulfilled</strong>. It is passed the returned {{domxref("Response")}} object as a parameter.</p>

  <div class="blockIndicator note">
  <p><strong>Note</strong>: The way that a <code>.then()</code> block works is similar to when you add an event listener to an object using <code>AddEventListener()</code>. It doesn't run until an event occurs (when the promise fulfills). The most notable difference is that a .then() will only run once for each time it is used, whereas an event listener could be invoked multiple times.</p>
  </div>

  <p>We immediately run the <code>blob()</code> method on this response to ensure that the response body is fully downloaded, and when it is available transform it into a <code>Blob</code> object that we can do something with. The result of this is returned like so:</p>

  <pre class="brush: js notranslate">response =&gt; response.blob()</pre>

  <p>which is shorthand for</p>

  <pre class="brush: js notranslate">function(response) {
  return response.blob();
}</pre>

  <p>OK, enough explanation for now. Add the following line below your first line of JavaScript.</p>

  <pre class="brush: js notranslate">let promise2 = promise.then(response =&gt; response.blob());</pre>
 </li>
 <li>
  <p>Each call to <code>.then()</code> creates a new promise. This is very useful; because the <code>blob()</code> method also returns a promise, we can handle the <code>Blob</code> object it returns on fulfillment by invoking the <code>.then()</code> method of the second promise. Because we want to do something a bit more complex to the blob than just run a single method on it and return the result, we'll need to wrap the function body in curly braces this time (otherwise it'll throw an error).</p>

  <p>Add the following to the end of your code:</p>

  <pre class="brush: js notranslate">let promise3 = promise2.then(myBlob =&gt; {

})</pre>
 </li>
 <li>
  <p>Now let's fill in the body of the executor function. Add the following lines inside the curly braces:</p>

  <pre class="brush: js notranslate">let objectURL = URL.createObjectURL(myBlob);
let image = document.createElement('img');
image.src = objectURL;
document.body.appendChild(image);</pre>

  <p>Here we are running the {{domxref("URL.createObjectURL()")}} method, passing it as a parameter the <code>Blob</code> returned when the second promise fulfills. This will return a URL pointing to the object. Then we create an {{htmlelement("img")}} element, set its <code>src</code> attribute to equal the object URL and append it to the DOM, so the image will display on the page!</p>
 </li>
</ol>

<p>If you save the HTML file you've just created and load it in your browser, you'll see that the image is displayed in the page as expected. Good work!</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: You will probably notice that these examples are somewhat contrived. You could just do away with the whole <code>fetch()</code> and <code>blob()</code> chain, and just create an <code>&lt;img&gt;</code> element and set its <code>src</code> attribute value to the URL of the image file, <code>coffee.jpg</code>. We did however pick this example because it demonstrates promises in a nice simple fashion, rather than for its real world appropriateness.</p>
</div>

<h3 id="Responding_to_failure">Responding to failure</h3>

<p>There is something missing — currently there is nothing to explicitly handle errors if one of the promises fails (<strong>rejects</strong>, in promise-speak). We can add error handling by running the <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch">.catch()</a></code> method of the previous promise. Add this now:</p>

<pre class="brush: js notranslate">let errorCase = promise3.catch(e =&gt; {
  console.log('There has been a problem with your fetch operation: ' + e.message);
});</pre>

<p>To see this in action, try misspelling the URL to the image and reloading the page. The error will be reported in the console of your browser's developer tools.</p>

<p>This doesn't do much more than it would if you just didn't bother including the <code>.catch()</code> block at all, but think about it — this allows us to control error handling exactly how we want. In a real app, your <code>.catch()</code> block could retry fetching the image, or show a default image, or prompt the user to provide a different image URL, or whatever.</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: You can see <a href="https://mdn.github.io/learning-area/javascript/asynchronous/promises/simple-fetch.html">our version of the example live</a> (see the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/simple-fetch.html">source code</a> also).</p>
</div>

<h3 id="Chaining_the_blocks_together">Chaining the blocks together</h3>

<p>This is a very longhand way of writing this out; we've deliberately done this to help you understand what is going on clearly. As shown earlier on in the article, you can chain together <code>.then()</code> blocks (and also <code>.catch()</code> blocks). The above code could also be written like this (see also <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/simple-fetch-chained.html">simple-fetch-chained.html</a> on GitHub):</p>

<pre class="brush: js notranslate">fetch('coffee.jpg')
.then(response =&gt; response.blob())
.then(myBlob =&gt; {
  let objectURL = URL.createObjectURL(myBlob);
  let image = document.createElement('img');
  image.src = objectURL;
  document.body.appendChild(image);
})
.catch(e =&gt; {
  console.log('There has been a problem with your fetch operation: ' + e.message);
});</pre>

<p>Bear in mind that the value returned by a fulfilled promise becomes the parameter passed to the next <code>.then()</code> block's executor function.</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: <code>.then()</code>/<code>.catch()</code> blocks in promises are basically the async equivalent of a <code><a href="/en-US/docs/Web/JavaScript/Reference/Statements/try...catch">try...catch</a></code> block in sync code. Bear in mind that synchronous <code>try...catch</code> won't work in async code.</p>
</div>

<h2 id="Promise_terminology_recap">Promise terminology recap</h2>

<p>There was a lot to cover in the above section, so let's go back over it quickly to give you a <a href="/en-US/docs/Learn/JavaScript/Asynchronous/Promises#Promise_terminology_recap">short guide that you can bookmark</a> and use to refresh your memory in the future. You should also go over the above section again a few more time to make sure these concepts stick.</p>

<ol>
 <li>When a promise is created, it is neither in a success or failure state. It is said to be <strong>pending</strong>.</li>
 <li>When a promise returns, it is said to be <strong>resolved</strong>.
  <ol>
   <li>A successfully resolved promise is said to be <strong>fulfilled</strong>. It returns a value, which can be accessed by chaining a <code>.then()</code> block onto the end of the promise chain. The executor function inside the <code>.then()</code> block will contain the promise's return value.</li>
   <li>An unsuccessful resolved promise is said to be <strong>rejected</strong>. It returns a <strong>reason</strong>, an error message stating why the promise was rejected. This reason can be accessed by chaining a <code>.catch()</code> block onto the end of the promise chain.</li>
  </ol>
 </li>
</ol>

<h2 id="Running_code_in_response_to_multiple_promises_fulfilling">Running code in response to multiple promises fulfilling</h2>

<p>The above example showed us some of the real basics of using promises. Now let's look at some more advanced features. For a start, chaining processes to occur one after the other is all fine, but what if you want to run some code only after a whole bunch of promises have <em>all</em> fulfilled?</p>

<p>You can do this with the ingeniously named <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/all">Promise.all()</a></code> static method. This takes an array of promises as an input parameter and returns a new <code>Promise</code> object that will fulfill only if and when <em>all</em> promises in the array fulfill. It looks something like this:</p>

<pre class="brush: js notranslate">Promise.all([a, b, c]).then(values =&gt; {
  ...
});</pre>

<p>If they all fulfill, then chained <code>.then()</code> block's executor function will be passed an array containing all those results as a parameter. If any of the promises passed to <code>Promise.all()</code> reject, the whole block will reject.</p>

<p>This can be very useful. Imagine that we’re fetching information to dynamically populate a UI feature on our page with content. In many cases, it makes sense to receive all the data and only then show the complete content, rather than displaying partial information.</p>

<p>Let's build another example to show this in action.</p>

<ol>
 <li>
  <p>Download a fresh copy of our <a href="https://github.com/mdn/learning-area/blob/master/html/introduction-to-html/getting-started/index.html">page template</a>, and again put a <code>&lt;script&gt;</code> element just before the closing <code>&lt;/body&gt;</code> tag.</p>
 </li>
 <li>
  <p>Download our source files (<a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/coffee.jpg">coffee.jpg</a>, <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/tea.jpg">tea.jpg</a>, and <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/description.txt">description.txt</a>), or feel free to substitute your own.</p>
 </li>
 <li>
  <p>In our script we'll first define a function that returns the promises we want to send to <code>Promise.all()</code>. This would be easy if we just wanted to run the <code>Promise.all()</code> block in response to three <code>fetch()</code> operations completing. We could just do something like:</p>

  <pre class="brush: js notranslate">let a = fetch(url1);
let b = fetch(url2);
let c = fetch(url3);

Promise.all([a, b, c]).then(values =&gt; {
  ...
});</pre>

  <p>When the promise is fulfilled, the <code>values</code> passed into the fullfillment handler would contain three <code>Response</code> objects, one for each of the <code>fetch()</code> operations that have completed.</p>

  <p>However, we don't want to do this. Our code doesn't care when the <code>fetch()</code> operations are done. Instead, what we want is the loaded data. That means we want to run the <code>Promise.all()</code> block when we get back usable blobs representing the images, and a usable text string. We can write a function that does this; add the following inside your <code>&lt;script&gt;</code> element:</p>

  <pre class="brush: js notranslate">function fetchAndDecode(url, type) {
  return fetch(url).then(response =&gt; {
    if (type === 'blob') {
      return response.blob();
    } else if (type === 'text') {
      return response.text();
    }
  })
  .catch(e =&gt; {
    console.log('There has been a problem with your fetch operation: ' + e.message);
  });
}</pre>

  <p>This looks a bit complex, so let's run through it step by step:</p>

  <ol>
   <li>First of all we define the function, passing it a URL and a string representing the type of resource it is fetching.</li>
   <li>Inside the function body, we have a similar structure to what we saw in the first example — we call the <code>fetch()</code> function to fetch the resource at the specified URL, then chain it onto another promise that returns the decoded (or "read") response body. This was always the <code>blob()</code> method in the previous example.</li>
   <li>However, two things are different here:
    <ul>
     <li>First of all, the second promise we return is different depending on what the <code>type</code> value is. Inside the executor function we include a simple <code>if ... else if</code> statement to return a different promise depending on what type of file we need to decode (in this case we've got a choice of <code>blob</code> or <code>text</code>, but it would be easy to extend this to deal with other types as well).</li>
     <li>Second, we have added the <code>return</code> keyword before the <code>fetch()</code> call. The effect this has is to run the entire chain and then run the final result (i.e. the promise returned by <code>blob()</code> or <code>text()</code>) as the return value of the function we've just defined. In effect, the <code>return</code> statements pass the results back up the chain to the top.</li>
    </ul>
   </li>
   <li>
    <p>At the end of the block, we chain on a <code>.catch()</code> call, to handle any error cases that may come up with any of the promises passed in the array to <code>.all()</code>. If any of the promises reject, the catch block will let you know which one had a problem. The <code>.all()</code> block (see below) will still fulfill, but just won't display the resources that had problems. If you wanted the <code>.all</code> to reject, you'd have to chain the <code>.catch()</code> block on to the end of there instead.</p>
   </li>
  </ol>

  <p>The code inside the function body is async and promise-based, therefore in effect the entire function acts like a promise — convenient.</p>
 </li>
 <li>
  <p>Next, we call our function three times to begin the process of fetching and decoding the images and text, and store each of the returned promises in a variable. Add the following below your previous code:</p>

  <pre class="brush: js notranslate">let coffee = fetchAndDecode('coffee.jpg', 'blob');
let tea = fetchAndDecode('tea.jpg', 'blob');
let description = fetchAndDecode('description.txt', 'text');</pre>
 </li>
 <li>
  <p>Next, we will define a <code>Promise.all()</code> block to run some code only when all three of the promises stored above have successfully fulfilled. To begin with, add a block with an empty executor inside the <code>.then()</code> call, like so:</p>

  <pre class="brush: js notranslate">Promise.all([coffee, tea, description]).then(values =&gt; {

});</pre>

  <p>You can see that it takes an array containing the promises as a parameter. The executor will only run when all three promises resolve; when that happens, it will be passed an array containing the results from the individual promises (i.e. the decoded response bodies), kind of like [coffee-results, tea-results, description-results].</p>
 </li>
 <li>
  <p>Last of all, add the following inside the executor. Here we use some fairly simple sync code to store the results in separate variables (creating object URLs from the blobs), then display the images and text on the page.</p>

  <pre class="brush: js notranslate">console.log(values);
// Store each value returned from the promises in separate variables; create object URLs from the blobs
let objectURL1 = URL.createObjectURL(values[0]);
let objectURL2 = URL.createObjectURL(values[1]);
let descText = values[2];

// Display the images in &lt;img&gt; elements
let image1 = document.createElement('img');
let image2 = document.createElement('img');
image1.src = objectURL1;
image2.src = objectURL2;
document.body.appendChild(image1);
document.body.appendChild(image2);

// Display the text in a paragraph
let para = document.createElement('p');
para.textContent = descText;
document.body.appendChild(para);</pre>
 </li>
 <li>
  <p>Save and refresh and you should see your UI components all loaded, albeit in a not particularly attractive way!</p>
 </li>
</ol>

<p>The code we provided here for displaying the items is fairly rudimentary, but works as an explainer for now.</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: If you get stuck, you can compare your version of the code to ours, to see what it is meant to look like — <a href="https://mdn.github.io/learning-area/javascript/asynchronous/promises/promise-all.html">see it live</a>, and see the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/promise-all.html">source code</a>.</p>
</div>

<div class="blockIndicator note">
<p><strong>Note</strong>: If you were improving this code, you might want to loop through a list of items to display, fetching and decoding each one, and then loop through the results inside <code>Promise.all()</code>, running a different function to display each one depending on what the type of code was. This would make it work for any number of items, not just three.</p>

<p>In addition, you could determine what the type of file is being fetched without needing an explicit <code>type</code> property. You could for example check the {{HTTPHeader("Content-Type")}} HTTP header of the response in each case using <code><a href="/en-US/docs/Web/API/Headers/get">response.headers.get("content-type")</a></code>, and then react accordingly.</p>
</div>

<h2 id="Running_some_final_code_after_a_promise_fulfillsrejects">Running some final code after a promise fulfills/rejects</h2>

<p>There will be cases where you want to run a final block of code after a promise completes, regardless of whether it fulfilled or rejected. Previously you'd have to include the same code in both the <code>.then()</code> and <code>.catch()</code> callbacks, for example:</p>

<pre class="brush: js notranslate">myPromise
.then(response =&gt; {
  doSomething(response);
  runFinalCode();
})
.catch(e =&gt; {
  returnError(e);
  runFinalCode();
});</pre>

<p>In more recent modern browsers, the <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/finally">.finally()</a></code> method is available, which can be chained onto the end of your regular promise chain allowing you to cut down on code repetition and do things more elegantly. The above code can now be written as follows:</p>

<pre class="brush: js notranslate">myPromise
.then(response =&gt; {
  doSomething(response);
})
.catch(e =&gt; {
  returnError(e);
})
.finally(() =&gt; {
  runFinalCode();
});</pre>

<p>For a real example, take a look at our <a href="https://mdn.github.io/learning-area/javascript/asynchronous/promises/promise-finally.html">promise-finally.html demo</a> (see the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/promise-finally.html">source code</a> also). This works exactly the same as the <code>Promise.all()</code> demo we looked at in the above section, except that in the <code>fetchAndDecode()</code> function we chain a <code>finally()</code> call on to the end of the chain:</p>

<pre class="brush: js notranslate">function fetchAndDecode(url, type) {
  return fetch(url).then(response =&gt; {
    if(type === 'blob') {
      return response.blob();
    } else if(type === 'text') {
      return response.text();
    }
  })
  .catch(e =&gt; {
    console.log(`There has been a problem with your fetch operation for resource "${url}": ` + e.message);
  })
  .finally(() =&gt; {
    console.log(`fetch attempt for "${url}" finished.`);
  });
}</pre>

<p>This logs a simple message to the console to tell us when each fetch attempt has finished.</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: <code>finally()</code> allows you to write async equivalents to try/catch/finally in async code.</p>
</div>

<h2 id="Building_your_own_custom_promises">Building your own custom promises</h2>

<p>The good news is that, in a way, you've already built your own promises. When you've chained multiple promises together with <code>.then()</code> blocks, or otherwise combined them to create custom functionality, you are already making your own custom async promise-based functions. Take our <code>fetchAndDecode()</code> function from the previous examples, for example.</p>

<p>Combining different promise-based APIs together to create custom functionality is by far the most common way you'll do custom things with promises, and shows the flexibility and power of basing most modern APIs around the same principle. There is another way, however.</p>

<h3 id="Using_the_Promise_constructor">Using the Promise() constructor</h3>

<p>It is possible to build your own promises using the <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise">Promise()</a></code> constructor. The main situation in which you'll want to do this is when you've got code based on an an old-school asynchronous API that is not promise-based, which you want to promis-ify. This comes in handy when you need to use existing, older project code, libraries, or frameworks along with modern promise-based code.</p>

<p>Let's have a look at a simple example to get you started — here we wrap a <code><a href="/en-US/docs/Web/API/WindowOrWorkerGlobalScope/setTimeout">setTimeout()</a></code> call with a promise — this runs a function after two seconds that resolves the promise (using the passed <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve">resolve()</a></code> call) with a string of "Success!".</p>

<pre class="brush: js notranslate">let timeoutPromise = new Promise((resolve, reject) =&gt; {
  setTimeout(function(){
    resolve('Success!');
  }, 2000);
});</pre>

<p><code>resolve()</code> and <code>reject()</code> are functions that you call to fulfill or reject the newly-created promise. In this case, the promise fulfills with a string of "Success!".</p>

<p>So when you call this promise, you can chain a <code>.then()</code> block onto the end of it and it will be passed a string of "Success!". In the below code we simply alert that message:</p>

<pre class="brush: js notranslate">timeoutPromise
.then((message) =&gt; {
   alert(message);
})</pre>

<p>or even just</p>

<pre class="brush: js notranslate">timeoutPromise.then(alert);
</pre>

<p>Try <a href="https://mdn.github.io/learning-area/javascript/asynchronous/promises/custom-promise.html">running this live</a> to see the result (also see the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/custom-promise.html">source code</a>).</p>

<p>The above example is not very flexible — the promise can only ever fulfill with a single string, and it doesn't have any kind of <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject">reject()</a></code> condition specified (admittedly, <code>setTimeout()</code> doesn't really have a fail condition, so it doesn't matter for this simple example).</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: Why <code>resolve()</code>, and not <code>fulfill()</code>? The answer we'll give you for now is <em>it's complicated</em>.</p>
</div>

<h3 id="Rejecting_a_custom_promise">Rejecting a custom promise</h3>

<p>We can create a promise that rejects using the <code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject">reject()</a></code> method — just like <code>resolve()</code>, this takes a single value, but in this case it is the reason to reject with, i.e., the error that will be passed into the <code>.catch()</code> block.</p>

<p>Let's extend the previous example to have some <code>reject()</code> conditions as well as allowing different messages to be passed upon success.</p>

<p>Take a copy of the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/custom-promise.html">previous example</a>, and replace the existing <code>timeoutPromise()</code> definition with this:</p>

<pre class="brush: js notranslate">function timeoutPromise(message, interval) {
  return new Promise((resolve, reject) =&gt; {
    if (message === '' || typeof message !== 'string') {
      reject('Message is empty or not a string');
    } else if (interval &lt; 0 || typeof interval !== 'number') {
      reject('Interval is negative or not a number');
    } else {
      setTimeout(function(){
        resolve(message);
      }, interval);
    }
  });
};</pre>

<p>Here we are passing two methods into a custom function — a message to do something with, and the time interval to pass before doing the thing. Inside the function we then return a new <code>Promise</code> object — invoking the function will return the promise we want to use.</p>

<p>Inside the Promise constructor, we do a number of checks inside <code>if ... else</code> structures:</p>

<ol>
 <li>First of all we check to see if the message is appropriate for being alerted. If it is an empty string or not a string at all, we reject the promise with a suitable error message.</li>
 <li>Next, we check to see if the interval is an appropriate interval value. If it is negative or not a number, we reject the promise with a suitable error message.</li>
 <li>Finally, if the parameters both look OK, we resolve the promise with the specified message after the specified interval has passed using <code>setTimeout()</code>.</li>
</ol>

<p>Since the <code>timeoutPromise()</code> function returns a <code>Promise</code>, we can chain <code>.then()</code>, <code>.catch()</code>, etc. onto it to make use of its functionality. Let's use it now — replace the previous <code>timeoutPromise</code> usage with this one:</p>

<pre class="brush: js notranslate">timeoutPromise('Hello there!', 1000)
.then(message =&gt; {
   alert(message);
})
.catch(e =&gt; {
  console.log('Error: ' + e);
});</pre>

<p>When you save and run the code as is, after one second you'll get the message alerted. Now try setting the message to an empty string or the interval to a negative number, for example, and you'll be able to see the promise reject with the appropriate error messages! You could also try doing something else with the resolved message rather than just alerting it.</p>

<div class="blockIndicator note">
<p><strong>Note</strong>: You can find our version of this example on GitHub as <a href="https://mdn.github.io/learning-area/javascript/asynchronous/promises/custom-promise2.html">custom-promise2.html</a> (see also the <a href="https://github.com/mdn/learning-area/blob/master/javascript/asynchronous/promises/custom-promise2.html">source code</a>).</p>
</div>

<h3 id="A_more_real-world_example">A more real-world example</h3>

<p>The above example was kept deliberately simple to make the concepts easy to understand, but it is not really very async. The asynchronous nature is basically faked using <code>setTimeout()</code>, although it does still show that promises are useful for creating a custom function with sensible flow of operations, good error handling, etc.</p>

<p>One example we'd like to invite you to study, which does show a useful async application of the <code>Promise()</code> constructor, is <a href="https://github.com/jakearchibald/idb/">Jake Archibald's idb library</a>. This takes the <a href="/en-US/docs/Web/API/IndexedDB_API">IndexedDB API</a>, which is an old-style callback-based API for storing and retrieving data on the client-side, and allows you to use it with promises. If you look at the <a href="https://github.com/jakearchibald/idb/blob/master/lib/idb.js">main library file</a> you'll see the same kind of techniques we discussed above being used there. The following block converts the basic request model used by many IndexedDB methods to use promises:</p>

<pre class="brush: js notranslate">function promisifyRequest(request) {
  return new Promise(function(resolve, reject) {
    request.onsuccess = function() {
      resolve(request.result);
    };

    request.onerror = function() {
      reject(request.error);
    };
  });
}</pre>

<p>This works by adding a couple of event handlers that fulfill and reject the promise at appropriate times:</p>

<ul>
 <li>When the <code><a href="/en-US/docs/Web/API/IDBRequest">request</a></code>'s <a href="/en-US/docs/Web/API/IDBRequest/success_event"><code>success</code> event</a> fires, the <code><a href="/en-US/docs/Web/API/IDBRequest/onsuccess">onsuccess</a></code> handler fulfills the promise with the request <code><a href="/en-US/docs/Web/API/IDBRequest/result">result</a></code>.</li>
 <li>When the <code><a href="/en-US/docs/Web/API/IDBRequest">request</a></code>'s <a href="/en-US/docs/Web/API/IDBRequest/error_event"><code>error</code> event</a> fires, the <code><a href="/en-US/docs/Web/API/IDBRequest/onerror">onerror</a></code> handler rejects the promise with the request <code><a href="/en-US/docs/Web/API/IDBRequest/error">error</a></code>.</li>
</ul>

<h2 id="Conclusion">Conclusion</h2>

<p>Promises are a good way to build asynchronous applications when we don’t know the return value of a function or how long it will take to return. They make it easier to express and reason about sequences of asynchronous operations without deeply nested callbacks, and they support a style of error handling that is similar to the synchronous <code>try...catch</code> statement.</p>

<p>Promises work in the latest versions of all modern browsers; the only place where promise support will be a problem is in Opera Mini and IE11 and earlier versions.</p>

<p>We didn't touch on all promise features in this article, just the most interesting and useful ones. As you start to learn more about promises, you'll come across further features and techniques.</p>

<p>Most modern Web APIs are promise-based, so you'll need to understand promises to get the most out of them. Among those APIs are <a href="/en-US/docs/Web/API/WebRTC_API">WebRTC</a>, <a href="/en-US/docs/Web/API/Web_Audio_API">Web Audio API</a>, <a href="/en-US/docs/Web/API/Media_Streams_API">Media Capture and Streams</a>, and many more. Promises will be more and more important as time goes on, so learning to use and understand them is an important step in learning modern JavaScript.</p>

<h2 id="See_also">See also</h2>

<ul>
 <li><code><a href="/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise">Promise()</a></code></li>
 <li><a href="/en-US/docs/Web/JavaScript/Guide/Using_promises">Using promises</a></li>
 <li><a href="https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html">We have a problem with promises</a> by Nolan Lawson</li>
</ul>

<p>{{PreviousMenuNext("Learn/JavaScript/Asynchronous/Timeouts_and_intervals", "Learn/JavaScript/Asynchronous/Async_await", "Learn/JavaScript/Asynchronous")}}</p>

<h2 id="In_this_module">In this module</h2>

<ul>
 <li><a href="/en-US/docs/Learn/JavaScript/Asynchronous/Concepts">General asynchronous programming concepts</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Asynchronous/Introducing">Introducing asynchronous JavaScript</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Asynchronous/Timeouts_and_intervals">Cooperative asynchronous JavaScript: Timeouts and intervals</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Asynchronous/Promises">Graceful asynchronous programming with Promises</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Asynchronous/Async_await">Making asynchronous programming easier with async and await</a></li>
 <li><a href="/en-US/docs/Learn/JavaScript/Asynchronous/Choosing_the_right_approach">Choosing the right approach</a></li>
</ul>
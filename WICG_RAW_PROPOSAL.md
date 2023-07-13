[Proposal] API for first synchronous access to the window proxy object of every newborn same origin realm
APIs
weizmanDecember 7, 2022, 1:44pm#1
Proposal for enabling web apps full and first access and control over all same origin realms
tl;dr
Web applications as of today have no control/visibility over newborn same origin realms in runtime.
When loaded, same origin realms should be processed primarily by the app and only afterwards by their creator, to allow the app to decide how same origin realms should behave as their host, just like how the app has the right to set its own CSP.
This is not new as a concept, as this exact ability is granted by the browser to extensions and plugins in the form of the manifest field all_frames, we just think it makes sense to grant such power over child frames to the web app too.
We have a working shim called Snow :snowflake: of how we imagine a solution to this problem should behave, which is already integrated into MetaMask :fox_face: (merged) and is used to enhance its protection.
The main use case for this idea is for 1st/3rd party services who wish to apply certain dynamic rules to a web app which are very incomplete when are only applied to the top main realm but not to all realms automatically.
To further explore the different use cases jump over to Motivation.
To further explore this idea, its motivation, the realms security problem and Snow - jump over to the Resources section at the bottom of the page.

Introduction
We believe a web app should have full and first control over newborn realms within itself before any other entity does, including the creator of any certain realm.

In other words, if a certain script (whether of the app or of a 3rd party) lifts up a new realm for whatever reason, the window proxy representing that realm should first be processed by the app itself and only afterwards by that certain script.

Clarification
So for example, if web app https://my-app.com loads a 3rd party service JS script from https://external-service/sdk.js and that service uses iframes as part of its logic, the iframe naturally will load with its own realm.

This is very common for many reasons, a common example would be ad services such as Google Ads that optionally load ads within iframes.

In today’s landscape, the context of web apps such as https://my-app.com has no control over such rising realms and therefore cannot limit/define their power/abilities/behaviour.

To solve this problem, we:
suggest a proposal for a new API;
would like to present a shim that implements fully this proposed API.
Proposal
We imagine support for a new window owned property "observe" (probably should come up with a better name) that its descriptor is as follows:

{ value: observe, writable: true, enumerable: false, configurable: true }
Where observe is a function that gets a callback which will be called with every new realm that comes to life within the app:

observe((win) => console.log('Here is a new window:', win));
Listing the callback by calling observe should only be possible:

Once (per page load);
By the web app itself (meaning, by a script that originates in the same origin as the app, similar to how CSP is also a privilege that only belongs to the web app);
By the top most window (aka top).
Important notes
The callback should be called synchronously after the creation of the realm, so that the registrer of the callback assured to be getting first access to the realm before any other entity in the app.
If the same realm was already introduced to the page, the callback will fire again anyway. This is because there are cases in which the same realm representation - the WindowProxy object - is loaded more than once (e.g. iframe src relocation).
The browser should pass to the registered callback same origin realms only, as those are the true potential danger to the web app. Cross origin realms cannot synchronously access the web app and therefore cannot be leveraged maliciously against the app.
Shim
I created Snow :snowflake: which is a shim that when is applied to the web app at the beginning of its execution, fully applies the above proposal to the execution runtime.

Once applied, a new property SNOW is exported via the globalThis which accepts the described callback and calls it with every new realm that comes to life within the app.

Being a shim, Snow achieves its goal by hooking into all the different points where new realms are formed in JavaScript and makes sure to pass all new realms to the callback before returning them to their creator.

However, being a shim also means Snow comes with clear disadvantages such as impacting performance, potentially not being bulletproof and more.

Therefore, Implementing Snow as part of the browser will allow us to get the advantages it brings with it without these disadvantages.

Motivation
The motivation here is to simply endow the web app with the capabilities that it deserves to have, which is governance and control over same origin realms within itself.

This isn’t a new concept - this exact ability is granted by the browser to extensions and plugins in the form of the manifest field all_frames, we just think it makes sense to grant such power over child frames to the web app too.
With the minor exception in which extensions get primer access to all child frames whether cross or same origin whereas the web app should only have power over the realms under its own origin.
Here we’ll explore the different reasons and use cases of why we might want to realize this proposal.
1. Third Party Vendors For Client Side Security Products
Part A - The problem with not having builtin browser support:

As earlier described and demonstrated in the Clarification section, iframes are being used today in all sorts of legitimate ways.

However, in addition to the legitimate usage of them, iframes can also easily be leveraged maliciously in web security context.

This is far more relevant today than before due to the major shift web apps builders took towards fully relying on 3rd party dependencies when building their apps.

That is because securing a supply chain, which was less of a thing a couple of years back, is currently a very difficult problem to solve.

With the rise of supply chain attacks against web apps, different protection solutions (see “Part C” for examples) came to life, many of them around securing the app and browser builtin sensitive APIs in runtime.

The motivation was basically “It’s hard to tell what part of the supply chain will be breached and how, so let’s apply protection to the app itself in realtime, so that when an attacker finds a breach, we’ll already be defending the app”.

It’s hard to tell how an attacker will find its way into a supply chain, but might be easier to predict what they’ll try to do once they’re in.

Here’s an example: if the attacker attempts to steal the contents of document.cookie from the web page and exfiltrate it to its malicious domain, it might be smart to apply protection around access to the JavaScript API of document.cookie. Such protection can be applied by what is called “monkey patching” - a technique that is used to override builtin APIs provided by the browser with custom hooks:

/* defender */
const desc = Object.getOwnPropertyDescriptor(Document.prototype, 'cookie');
desc.get = () => ''; // disable document.cookie API getter
Object.defineProperty(Document.prototype, 'cookie', desc);
That is of course an over simplification, but in that manner it is possible to apply robust protection around different sensitive APIs and by that encounter an attacker more prepared in the battlefield.

However, such protections are basically worthless due to the power that realms inherently bring with them, because the attacker can easily bypass the protection demonstrated above by simply leveraging another instance of the document.cookie API from within the realm of an iframe:

/* attacker */
document.body.appendChild(document.createElement('iframe')).contentWindow.document.cookie;
While the example of protecting the top main realm’s document.cookie API is simple, dynamically defending every new document.cookie API that might be brought up to life within newborn realms is not possible as of today.

Part B - How builtin browser support can help solve the problem:

If the proposal described above makes it through, we could use the new suggested API to automatically apply our protection to all child realms with no effort:

/* defender */
top.observe((win) => {
    const desc = win.Object.getOwnPropertyDescriptor(win.Document.prototype, 'cookie');
    desc.get = () => ''; // disable document.cookie API getter
    win.Object.defineProperty(win.Document.prototype, 'cookie', desc);
});
The registered callback will process every new same origin realm that comes to life first and will apply our document.cookie API protection to it, thus leaving no option to leverage new realms to retrieve the original behaviour of the protected API.

Part C - Actors benefiting from this:

As mentioned in “Part A” above, there are a number of actors in the client side supply chain security industry that would highly benefit from being able to apply their security mechanizms automatically to all realms, some of them (but not all) are:

PerimeterX’s Code Defender
Akamai’s Page Integrity Manager
Imperva’s Client Side Protection
2. Client Side Monitoring Tools
Part A - The problem with not having builtin browser support:

There are third party services that provide monitoring tools for web apps to have better insights into how their apps function and behave “in the wild”.

To achieve that those usually apply monkey patches to different builtin APIs in the web app’s runtime to have visibility into how, why and how many times are they being used.

For example, the popular among those - Sentry.io - patches different APIs such as console to have real visibility into the console API usage of the web app in realtime.

Like before, this is a common practice among services like this, and they hook into many APIs than just console.

However, their visibility is incomplete, because in web apps where there is some to a lot of iframes usage, such services do not extend their visibility into such iframes and therefore miss a big part of the picture.

Part B - How builtin browser support can help solve the problem:

If the proposal described above makes it through, we could use the new suggested API to automatically apply those visibility-purposed patches into all iframes within the page too, thus significantly enhancing the quality of the service such third party vendors come to supply.

Part C - Actors benefiting from this:

As mentioned in “Part A” above, there are a number of actors in the client side visibility and monitoring industry that would highly benefit from being able to apply their tracking abilities automatically to all realms, some of them (but not all) are:

Sentry.io
LogRocket.com
3. Importance For Technological Innovation In JS
Part A - The problem with not having builtin browser support:

There are innovative projects (different shims mostly) that would benefit from being able to apply themselves to all same origin realms automatically, partly because they’re trying to apply technology to the page that needs to be deterministic and hermetic, which sometimes can be hard to achieve with multiple realms.

Part B - How builtin browser support can help solve the problem:

The ability to automatically apply such logic to all realms can help enhance its purpose and keep it fully secure.

Part C - Actors benefiting from this:

Some examples in that context:

Part of the SES shim initiative is a strong capability called lockdown which when is called freezes all platform intrinsics so that they could not be redefined and therefore trusted to function purely. However, calling lockdown only applies to the top main realm, and having the ability to apply it to all same origin realms automatically would have been beneficial for apps that wish to defend themselves using lockdown for their inner iframe activities too.
Across is a shim that introduces a way for two scripts within a single DOM to exchange messages with each other securely based on their origins. In order for this shim to fulfil its purpose safely, it must apply itself to all child realms, otherwise a pure realm can be leveraged to bypass its mechanizm. To achieve that, currently Across is built on top of Snow shim, but obviously replacing Snow with a builtin API would be far better.
4. More
We believe there are more examples for use cases, and there will be more in the future if this proposal goes through.

TBDs
Although the idea and the need are quite clear to us, there are minor questions to still answer:

Should the API simply receive a callback? Or should the callback be registered to a new event perhaps?
Should the API receive only one callback once? Or should it allow multiple callbacks registration chronologically?
What should be the name of the official API/event? (Not Snow obviously)
Should the descriptor of the API be configurable/writable?
Should the callback be called only with same origin realms? or with cross origin realms too? (even though the callback code won’t have sync access to newborn cross origin realms in such a case).
Resources
This proposal derives from a long term research around the security aspect of realms, and hard work on bringing Snow to life mostly to learn that it’s better off natively in the browser - here are some resources in this context:

awesome javascript realms security :star: - Long term realms security research documentation which includes references to:
Snow :snowflake: - Official shim repo.
Introduction to Snow - further reading on the rise of supply chain attacks and how it all lead to creating Snow.
Integrating Snow into MetaMask - Going deeper on why Snow is an important component for enhancing the defense of MetaMask and any other web app basically.
Live demo - a super simple demo to demonstrate Snow in action.
chaalsDecember 19, 2022, 8:11am#2
If browsers implement this, does it makes sense to integrate with something like CSP - so the web app actually asks for it in the first instance, and then (for example) gets to run the code it applies on your example observe…

Where would this fit in with e.g. extensions that run such code already? I.e. both of these things want to be first - so which one should be?

It seems there are security-enhancing benefits to doing this, but are there implications for privacy beyond the security aspect? I don’t think so, given that most of what has happened is that the top-level server is just making it harder for random intervention.

Beyond providing protection against malicious attacks (in the “supply chain” of code), are there things that the Web doesn’t do at the moment because of security/privacy implications that this would enable? I don’t think that’s necessary to justify the proposal (which makes sense to me), but it would be interesting to know.

naugturJanuary 4, 2023, 11:02am#3
The point of the mechanism is to let the app protect each realm in its origin. I believe there might be an alternative to exposing a JavaScript API.

Applications already use CSP to express their security restrictions. Pointing to code that needs to bootstrap each realm could become a part of that.

CSP header is served to all of those realms, because it’s set on the server. Let a new CSP directive point to a same origin JS file with the code that needs to run when window is created but document parsing did not start yet.

Also, if a realm is considered sameorigin because it didn’t load anything - the location of the code to load and run as bootstrap should be inherited from opener realm and opener should not receive the reference to the new window until the code runs.

That way we’re removing the necessity of defining a callback function and passing it to an API. As well as the question of what to do if another call is made to the API to register a callback because the same setup code is embedded in all pages. Overriding the protection during the original page load because developer made a mistake of running other code earlier or that code was XSSed in place is no longer a concern.

Pointing to a JavaScript url in the same origin bares similarities to Service Workers and content scripts from extensions. Defining the code declaratively and not bootstrapping it in an existing JavaScript context makes it even more similar to what extensions do than the original proposal.

weizmanJanuary 10, 2023, 1:23pm#4
Yea that actually sounds like a great idea @naugtur, integrating this with CSP while leveraging the already existing ability of the browser to run code in new realms using the manifest.json all_frames directive is an interesting choice.

To be honest I’m fine either way, as long as we get to ship this ability to browsers so different vendors can build on top of this, but agree your idea might be more accurate.

Where would this fit in with e.g. extensions that run such code already? I.e. both of these things want to be first - so which one should be?

Extensions always get to be first, that is their given right being highly privileged software in comparison with web apps code.

It seems there are security-enhancing benefits to doing this, but are there implications for privacy beyond the security aspect? I don’t think so, given that most of what has happened is that the top-level server is just making it harder for random intervention.

Can’t think of any privacy concerns.

Beyond providing protection against malicious attacks (in the “supply chain” of code), are there things that the Web doesn’t do at the moment because of security/privacy implications that this would enable? I don’t think that’s necessary to justify the proposal (which makes sense to me), but it would be interesting to know.

I think Across is a perfect example of a really interesting use case that could have been implemented securely on top of such a solution. Across allows two scripts to communicate with each other safely based on their origin which is a state that cannot be achieved as of today. The problem is that attackers can bypass this mechanizm by leveraging a new realm, and that is being eliminated in Across by applying its logic and defense mechanizm to all same origin realms automatically using Snow.

Across being brought up here as an example of a security/privacy concept that could be unlocked once we have this proposal integrated into the browser. But that’s all very theoretical for now.

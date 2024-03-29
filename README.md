# Awesome Javascript Realms Security [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)]()

> Curated list of Javascript Realms Security resources (also known as [The Same Origin Concern](https://weizmangal.com/2023/09/28/the-same-origin-concern/))

## Abstract

Due to the rise of dependencies based development, the javascript ecosystem (and the browser javascript ecosystem in particular) is far more vulnerable to the rising major problem we know as “supply chain attacks”.

Therefore, many different supply chain security solutions were introduced to the industry as well, focusing on different ends of it, ranging from build time to runtime protection.

However, runtime browser based protections usually lack a major component in their solutions, one that mostly leaves such solutions completely vulnerable, almost as if they were never there.

Realms (aka iframes in the browser) is an ancient and legitimate concept that goes through a horrific spinoff in the context of bypassing browser based supply chain security attempts.

And the worst part is that carrying out attacks is so easy with realms, but defending realms is so complicated.

It's time to dive into the so important yet ignored layer in securing against unwanted code execution - it's time to talk about the javascript realms blank spot and its offensive/defensive security aspects.

> It is important to note that the scope here is specifically around how unwanted code execution in the top main realm of a web app can bypass protections applied to that realm by leveraging another same origin child realm created by the attacker. There are other attacks involving iframes to be aware of (e.g. iframe injection, clickjacking, phishing and more), but those are out of the scope of this repository.

## Realms

### Spec & Docs

* [tc39 Realms](https://tc39.es/ecma262/#sec-code-realms) ⭐️
 * [tc39 Agents](https://tc39.es/ecma262/#sec-agents) (parent entity of realms)
 * [tc39 Agent Clusters](https://tc39.es/ecma262/#sec-agent-clusters) (parent entity of agents)

### Content

* [What is a realm in javascript?](https://weizman.github.io/page-what-is-a-realm-in-js/) ⭐️
* [The Same Origin Concern](https://weizmangal.com/2023/09/28/the-same-origin-concern/) ⭐️

## iFrames

### Spec & Docs

* [mdn iFrames](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)
* [html iFrames](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element)

## Realms(/iFrames) Security

### Content

* [Realms role in supply chain attacks](https://twitter.com/WeizmanGal/status/1576942106156810240) ⭐️
* [Introduction - What is realms security?](https://twitter.com/WeizmanGal/status/1593879906928074753?s=20&t=JCfpu5-ZIAFgLKC1Wrvd4g) ⭐️

### In the wild

* [Skimmer acts as payment service provider via rogue iframe](https://www.malwarebytes.com/blog/news/2019/05/skimmer-acts-as-payment-service-provider-via-rogue-iframe) - by [malwarebytes](https://malwarebytes.com)

### Tools

* [Snow JS ❄️](https://github.com/lavamoat/snow) ⭐️ - the most secure tool out there for hermatic realms ownership 
  * [Introduction to Snow](https://github.com/lavamoat/snow/wiki/Introducing-Snow) - the rise of supply chain attacks and how it all lead to creating Snow
  * [Integrating Snow into MetaMask](https://weizman.github.io/page-snow-into-metamask/) ⭐️ - understanding how supply chain attacks affect web apps such as [MetaMask 🦊](https://github.com/metamask), how MetaMask develops [LavaMoat](https://github.com/lavamoat) to defend against those, and why it also needs Snow 
  * [Live demo](https://lavamoat.github.io/snow/demo/) ⭐️ - can you bypass snow? 
  * [Technical explanation](https://github.com/lavamoat/snow/wiki/Introducing-Snow#why-snow-solves-a-non-trivial-problem) ⭐️
  * [Source code](https://github.com/lavamoat/snow)

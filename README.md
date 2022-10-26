# Awesome Javascript Realms Security [![Awesome](https://cdn.rawgit.com/sindresorhus/awesome/d7305f38d29fed78fa85652e3a63e154dd8e8829/media/badge.svg)]()

> Curated list of Javascript Realms Security resources

Due to the rise of dependencies based development, the javascript ecosystem (and the browser javascript ecosystem in particular) is far more vulnerable to what we know as “supply chain attacks” - and the ability to create new realms in javascript is being leveraged to successfully carry out such attacks against web apps.

It's time to research and learn more about javascript realms and offensive/defensive security around them.

## Realms

* [What is a realm in javascript?](https://weizman.github.io/page-what-is-a-realm-in-js/)

## Realms Security

* [Realms role in supply chain attacks](https://twitter.com/WeizmanGal/status/1576942106156810240)

### Tools

* [Snow JS ❄️](https://github.com/lavamoat/snow) - the most secure tool out there for hermatic realms ownership
  * [Introduction to Snow](https://github.com/lavamoat/snow/wiki/Introducing-Snow) - the rise of supply chain attacks and how it all lead to creating Snow
  * [Integrating Snow into MetaMask 🦊](https://gist.github.com/weizman/2a67fb182924676285773be44138d52c) - Understanding how supply chain attacks affect web apps such as [MetaMask](https://github.com/metamask), how MetaMask develops [LavaMoat](https://github.com/lavamoat) to defend against those, and why it also needs Snow
  * [Live demo](https://lavamoat.github.io/snow/demo/) - can you bypass snow?
  * [Technical explanation](https://github.com/lavamoat/snow/wiki/Introducing-Snow#why-snow-solves-a-non-trivial-problem)
  * [Source code](https://github.com/lavamoat/snow)

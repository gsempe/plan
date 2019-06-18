---
date: 2019-06-15
tags: localization i18n
---

In 2015, the Golang core team added the [localization support](https://github.com/golang/go/issues/12750) in the x/text package. They produced a complete and quite complicated [internationalization system](https://phraseapp.com/blog/posts/internationalization-i18n-go/).

This was around one year after the creation of the [go-i18n](https://github.com/nicksnyder/go-i18n) project that was as well trying to solve the lack of localization tools/libraries.

Consequently, most of the Golang projects use [go-i18n](https://github.com/nicksnyder/go-i18n) for their localization. [Hugo](https://gohugo.io/), the fastest proclamed fastest framework for building website, is one of them.


How `Hugo` is using `go-i18n`?

Hugo is currently trying the v2 of go-i18n. cf. the [WIP PR](https://github.com/gohugoio/hugo/pull/6012/files)

In the file [i18n.go](https://github.com/gohugoio/hugo/blob/2838d58b1daa0f6a337125c5a64d06215901c5d6/langs/i18n/i18n.go), a Translator struct is defined. This struct embeds a map

**...TBC...**

`srcN` Polyfills
==============

I’m thinking through some of the ways we might be able to make use of this pattern without doing any damage in current browsers—avoiding a double download and eliminating as much potential as possible for a broken/hidden image.

Polyfilling this pattern would be exceedingly simple, on the surface: evaluate the `srcN` sources and, if an appropriate match is found, replace the `src` with the new source.

Unfortunately, in using a real `src` we open ourselves up to potential for an extraneous request in current browsers. As much as it would be nice to say “browsers will catch up soon and this won’t be an issue,” many of us still find ourselves fixing bugs specific to Android 2.3—despite it being released in 2008. In relying entirely on JavaScript to manage which source is displayed, we open up the possibility for a broken user script or browser plugin, third-party sharing widget, or advertising script to render content images inaccessible.

We know that including an `src` in current browsers means that `src` image is subject to prefetching, well prior to the application of any custom scripting.

```
<img src="fallbk-sm.jpg"
     src2="(min-width: 400px) pic-md.jpg"
     src1="(min-width: 1000px) pic-lg.jpg">
```

The above code leaves us in a position where 400px+ contexts that don’t have native support for this new pattern will prefetch the `src`, then replace it with the appropriate source by way of the polyfill. A non-JS or broken-JS environment will be provided with the fallback `src` alone which—as a representative not not ideally suited image—seems like a solid compromise from a progressive enhancement standpoint. 

Pros:
* Native fallback provided for non/broken JS experiences.

Cons:
* Double-download in unsupported browsers that qualify for an alternate `src`.

To avoid the double-download, we could make use of something along these lines:

```
<img data-src="fallbk-sm.jpg"
     src2="(min-width: 400px) pic-md.jpg"
     src1="(min-width: 1000px) pic-lg.jpg">
```

The polyfill would swap the contents of the `data-src` into the `src` attribute. This would avoid the double-download—and the preparser altogether—but provide no image in non/broken JS environments. This also relies on an `img` with no actual `src`, which may result in a broken `img` rendered on the page, and potential for additional oddities—including a request (see: http://www.nczonline.net/blog/2010/07/13/empty-string-urls-browser-update ).

Pros:
* Avoids redundant request _in browsers that don’t make a request for empty/omitted `src`_

Cons:
* Relies on an `img` with no native `src` 
* Forgoes prefetching
* No image provided in non/broken-JS environments

To work around the lack of an image in the above example, we could add something along these lines:

```
<img data-src="fallbk-sm.jpg"
     src2="(min-width: 400px) pic-md.jpg"
     src1="(min-width: 1000px) pic-lg.jpg">
<noscript><img src="fallbk-sm.jpg"></noscript>
```

As the DOM within a `noscript` is inert, the `src` of the fallback image won’t be prefetched. This accounts for JavaScript being wholesale disabled, but not for any issues that might prevent JavaScript from running as expected.

Pros:
* Avoids redundant request _in browsers that don’t make a request for empty/omitted `src`_

Cons:
* Relies on an `img` with no native `src` 
* Forgoes prefetching
* Still no image provided in broken-JS environments, which is far more common than “JS is disabled” environments.

I mostly wanted to hash out a few potential polyfill patterns for the sake of my own edification. While it isn’t technically a requirement that a proposed standard be easily polyfilled, that difficulty is likely to act as a huge barrier to adoption, and result in sites developed during the transition between the initial proposal and widespread adoption relying on hacks and “uncanny valley” versions of the proposed pattern.

None of the above patterns are particularly appealing, but this is just my first pass at thinking a polyfill through. Considering that we’re looking at a number of years before the native pattern is widely adopted, a polyfill that causes a redundant download or runs the risk of breaking content could cause harm to users for some time.

What concerns me is that the developer community has been waiting on a solution to this issue for a very long time now—any appealing standard is likely to see rapid adoption and polyfilling, as we’ve seen with the proposed `picture` element and Picturefill. While it’s easy to say that browsers will be quick to “catch up” and that polyfilling concerns are apt to be short-lived, any web developer can attest to the amount of time and effort we expend day-to-day for the sake of supporting widely-used legacy browsers.

Any solution that causes extraneous requests or limits access to content during the years it may take to see widespread native support for this pattern could put a serious burden on users. While it may be easy enough for us to discount the cost of an extraneous request for the sake of a thought exercise: when used several times on a page and multiplied across an entire site—by tens of thousands of developers in their daily work—we’ve just put a tremendous burden on users in developing countries, on underpowered devices, or stuck using legacy browsers. These are the very places where wasted bandwidth stands to do the most damage.

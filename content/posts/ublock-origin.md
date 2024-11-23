+++
date = '2024-11-23T17:43:33+01:00'
draft = false
title = 'Ublock Origin'
tags = ['uBlock-Origin', 'Cheatsheet'  ]
+++

[uBlock Origin](https://github.com/gorhill/uBlock) is a powerful ad blocker that allows you to create custom block lists to remove unwanted elements from web pages. This cheat sheet focuses on removing elements based on CSS id, class, and text content. It shows you how to create CSS selectors for various scenarios, including parent, sibling and child elements containing specific text or attributes. It is not only useful for removing unwanted ads, you can also use it to remove other elements from the web. Although some sites will have reader mode enabled, a heavily underused featire by many people, sometime you just want moar blocking. Browser Developer tools (i.e. "Inspect Element") are your friend to figure out the correct selectors. Here is the basic syntax for CSS selectors:


| Selector | Syntax | Description |
|--------------|--------|-------------|
| Element ID | `example.com##[id="some-id"]` | Blocks element with specific ID |
| Class | `example.com##.some-class` | Blocks elements with specific class |
| Multiple Classes | `example.com##.class1.class2` | Blocks elements with both classes |
| Attribute | `example.com##[data-ad="true"]` | Blocks elements with specific attribute |
| Contains Text | `example.com##div:has-text(Advertisement)` | Blocks div containing "Advertisement" |
| Exact Text Match | `example.com##div:has-text(/^Advertisement$/)` | Blocks div with exact text match |
| Case Insensitive | `example.com##div:has-text(/advertisement/i)` | Blocks div with case-insensitive match |
| Parent | `example.com##div:has(> .ad-class)` | Blocks parent div containing child with class "ad-class" |
| Previous Sibling | `example.com##div:has(+ .ad-class)` | Blocks div followed by element with class "ad-class" |
| Next Sibling | `example.com##.ad-class + div` | Blocks div that follows element with class "ad-class" |

### Complex Examples

* Block div containing "sponsored" with parent class "content":
```
example.com##.content div:has-text(/sponsored/i)
```

* Block article with image and sponsored text:
```
example.com##article:has(img):has-text(/sponsored/i)
```

* Remove parent container with multiple ad indicators:
```
example.com##div:has(:scope > div:has-text(/sponsored/i)):has(img[src*="ad"])
```

* Block element and its siblings when containing specific text:
```
example.com##div:has-text(Promoted) + *
```

### Going Beyond Basic Filters

Consider a common scenario: you're browsing a content site that cleverly disguises sponsored content. Instead of just looking for the word "sponsored," you might need to target a specific container with multiple indicators. One of my favorite approaches is combining parent and child selectors like this:

```css
example.com##.content div:has-text(/sponsored/i)
```

This tells uBlock to look for any div within the "content" class that contains the word "sponsored," regardless of case. But we can go even deeper. Sometimes sponsored content is identified by both text and images. Here's a more comprehensive filter:

```
example.com##article:has(img):has-text(/sponsored/i)
```

### Real-World Applications: Cleaning Up Popular Platforms

YouTube's Shorts feature is a prime example. While some users enjoy this TikTok-style content, others prefer the traditional YouTube experience. If you're in the latter camp, you can use filters from [gijsdev's repository](https://github.com/gijsdev/ublock-hide-yt-shorts).

Social media platforms like Twitter require particularly clever filtering due to their dynamic content loading. For Twitter, this combination has proven reliable:

```css
twitter.com##article:has(span:has-text(/Promoted/))
twitter.com##article:has([data-testid="promotedIndicator"])
```

Or alternatively:

```
x.com##[data-testid="tweet"]:has-text(/Promoted$/)
```

I keep my filterlist very small although I should probably add more to remove distractions from the web. You can serve the list as a text file from your webserver and load it into uBlock as a custom list. This way you can maintain it under version control and have it on all of your devices, updated automatically.

Another (hyperlocal) example is for the small town newspaper website. With some out of the box filter lists enabled, it removes the ads, but it leaves an enormous amount of whitespace. It turns out the containing div is not removed by the default block list, so I added a simple entry to remove it, and added a line to remove some silly "most read" bar on the side:

```
voorburgsdagblad.nl##.advrow
voorburgsdagblad.nl##.sidebar-warp
```

Thank you Raymond Hill for keeping the web bearable.

### Using Browser Developer Tools

While uBlock Origin's element picker is great for simple cases, understanding how to use browser Developer Tools will significantly improve your filtering abilities. The element inspector (usually opened with F12 or right-click > Inspect) helps you understand the structure of web pages and create more precise selectors. 

There are several excellent resources to get you started:
- Mozilla has an excellent [guide to browser developer tools](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Tools_and_setup/What_are_browser_developer_tools)
- Chrome's [DevTools DOM documentation](https://developer.chrome.com/docs/devtools/dom/) is comprehensive and applies to all Chromium-based browsers
- The uBlock Origin creator has written a detailed guide on [using the element picker](https://github.com/gorhill/uBlock/wiki/Element-picker)

The key is to look for unique identifiers or patterns that consistently appear around the elements you want to block. Sometimes, what looks like a simple ad might be nested several layers deep in the DOM, and developer tools help you navigate this hierarchy effectively.

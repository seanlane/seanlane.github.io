---
layout: post
title: Filtering domain names on Domcomp.com with regex
commentIssueId: 3
tags: domcomp, domain registration, filter domain names
---

Lately, I've been looking around for another domain name that's a little shorter and more flexible than seanlane.net. My purposes for that can be explained another time, but I have been trying to search domain names that have top-level domains (the .com part of google.com, for example) that are shorter in length, alongside some other attributes. There are a number of good tools to use for searching for available domain names, but I have not found any that allow for searching by the length of top-level domain. 

One of the tools I am a huge fan of is [DomComp](domcomp.com), but I feel like it could be a bit refined. It shows whether a domain is registered or not, but I would rather that the registered ones were hidden. With that, and the other goals I had, I wrote a JavaScript function that filters out the results from DomComp's search functionality:

``` javascript
// Filter domcomp.com results to show what I'm interested in
function clr() {
    // Google Global TLDs as of Jan 2017
    var g_tlds = ['ad', 'as', 'bz', 'cc', 'cd', 'co', 'dj', 'fm', 'io', 
                  'la', 'me', 'ms', 'nu', 'sc', 'sr', 'su', 'tv', 'tk', 'ws']
    var rows = document.getElementsByClassName('price_table')[0].rows;
    var re = /^[a-z0-9]+$/i;
    for (var i = 1; i < rows.length; i++) {
        if (rows[i].classList.contains('registered')    ||      // Don't show registered TLDs
            rows[i].attributes['tld'].value.length > 2  ||      // Don't show TLDs logner than 2 chars
            // $.inArray(rows[i].attributes['tld'].value, g_tlds) == -1 // Only show Google Global TLDs
            re.exec(rows[i].attributes['tld'].value) == null)   // Don't TLDs with non-English chars
        {
            rows[i].style.display = 'None';
        }
    }
}
```

The process for using the function is a little raw, but it gets the job done. The steps are as follows:

1. Go to [domcomp.com](domcomp.com)
2. Under 'New' domains, search for a domain name. Like 'seanlane' for example
3. Scroll down once or twice to make sure enough results load
4. Open a developer console in your browser
    * [How to open a console in Google Chrome](https://developers.google.com/web/tools/chrome-devtools/console/#open_as_panel)
    * [How to open a console in Mozilla Firefox](https://developer.mozilla.org/en-US/docs/Tools/Browser_Console#Opening_the_Browser_Console)
5. Copy and paste in the script above
    * **NOTE:** This is dangerous to do with code you do not completely understand, as this allows for malicious code to potentially be ran on your browser. I would hope that the script above is plain enough, but you should be very wary of doing this without making sure you understand what is being ran
6. In the console, call the function by entering `clr()` and pressing enter.
7. Repeat step 6 as necessary to filter out the results

In the script above, I have filtered out registered domain names, any domain names with TLDs longer than 2 characters, and domains that contain non-English characters. Commented out, but still there is a clause to only retain domains with TLDs that are within the list of what Google considers [Generic Country Code Top Level Domains (ccTLDs)](https://support.google.com/webmasters/answer/62399?hl=en). 

There is probably a much better way to go about this, the least of which would be running it as the results load in automatically, but it works well enough for the time being.
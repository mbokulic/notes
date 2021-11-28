@level=3

# single page applications
Single page vs server-side rendering

> Traditionally, the browser receives HTML from the server and renders it. When the user navigates to another URL, a full-page refresh is required and the server sends fresh new HTML for the new page. This is called server-side rendering.

> However in modern SPAs, client-side rendering is used instead. The browser loads the initial page from the server, along with the scripts (frameworks, libraries, app code) and stylesheets required for the whole app. When the user navigates to other pages, a page refresh is not triggered. The URL of the page is updated via the HTML5 History API. New data required for the new page, usually in JSON format, is retrieved by the browser via AJAX requests to the server. The SPA then dynamically updates the page with the data via JavaScript, which it has already downloaded in the initial page load.

## benefits and drawbacks of SPAs
> benefits:
>  - feels more responsive and users do not see the flash between page navigations due to full-page refreshes
>  - fewer HTTP requests to the server
>  - clear separation of the concerns between the client and the server; you can easily build new clients for different platforms (e.g. mobile, chatbots, smart watches) without having to modify the server code. You can also modify the technology stack on the client and server independently, as long as the API contract is not broken
>  - lazy loading: load content when necessary (eg, on scroll)
>  - hosting static files is cheaper than running server-side code

> drawbacks:
>  - heavier initial page load
>  - have to configure server to route all requests to a single entry point and have the client-side take over
>  - search engine crawlers that do not execute Javascript will think you have an empty page

## important tech concepts

### changing URLs
Use `window.history` to manually edit the URLs or use hashtags.

[SO link](https://stackoverflow.com/a/21862736)
*With this approach the exact state of the web app is embedded in the page URL. If copied and pasted to other browser window can open the exact same state. With HTML5 pushState() you can eliminate the #hash and use completely classic URLs which can resolve on the server on the first request and then load via ajax on subsequent requests.*



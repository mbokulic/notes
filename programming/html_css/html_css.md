# common patterns

## sizing img
When the height of an image or video is set, then its width can be set to auto so that the media scales proportionally. Reversing these two properties and values will also achieve the same result. 

```css
/*. Setting overflow to hidden ensures that any content with dimensions larger than the container will be hidden from view.*/
.container {
  width: 50%;
  height: 200px;
  overflow: hidden;
}

/*this ensures that images scale with the width of the container. height  is set to auto, meaning an image's height will automatically scale proportionally with the width. Finally, the last line will display images as block level elements, this will prevent images from attempting to align with other content on the page (like text), which can add unintended margin to the images.*/
.container img {
  max-width: 100%;
  height: auto;
  display: block;
}
```

## background image sizing
A background image of an HTML element will scale proportionally when its background-size property is set to cover.

# responsive design

## breakpoints
> Rather than set breakpoints based on specific devices, the best practice is to resize your browser to view where the website naturally breaks based on its content. The dimensions at which the layout breaks or looks odd become your media query breakpoints. Within those breakpoints, we can adjust the CSS to make the page resize and reorganize.

## mobile first
> A principle where you target the smallest screen with the base CSS and add media queries for larger screens. That way on mobile devices you do not have to download the media queries.

#  flexbox model
https://medium.freecodecamp.com/an-animated-guide-to-flexbox-d280cf6afc35#.45zz4lni5

# head tags

## viewport
Used to set the viewport (assumed width of the device) and the initial zoom level.

https://responsivedesign.is/develop/responsive-html/viewport-meta-element/
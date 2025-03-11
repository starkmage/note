## Characteristics of Responsive Layout

Responsive Web Design allows a website to adapt to multiple devices and screens simultaneously, enabling the website's layout and functionality to change according to the user's environment (screen size, input method, device/browser capabilities).

### Advantages

- High flexibility for devices with different viewport sizes
- Quick solution for multi-device display adaptation

### Disadvantages

- Only suitable for webpages with simple layouts, information, and frameworks
- High workload for device compatibility, low efficiency
- Code redundancy, hidden unused elements, increased loading time
- Actually a compromise design solution, optimal results hard to achieve due to multiple factors

## Media Queries

Using `@media` queries allows defining different styles for different media types, especially useful for responsive pages where multiple style sets can be written for different screen sizes to achieve adaptive effects.

```css
@media screen and (max-width: 960px){
    body{
      background-color:#FF6699
    }
}

@media screen and (max-width: 768px){
    body{
      background-color:#00FF66;
    }
}
```

## Fluid Layout (Percentage Layout, Flexbox Layout)

When browser width or height changes, using percentage units allows components' width and height to change with the browser, achieving responsive effects.

### Value References

- When child elements use percentage for width, it refers to the percentage of parent element's width
- When using percentage for height, it refers to the percentage of parent element's height
- When using percentage for margin/padding, it refers to the percentage of parent element's width, regardless of horizontal or vertical padding/margin
- Cannot use percentage for border width
- border-radius is different, if set as percentage, it's relative to its own width
- Child elements' top and bottom percentages are relative to the height of the first non-static positioned parent element; similarly, left and right percentages are relative to the width
- `font-size` percentage is relative to the font-size of the first outer layer containing the font-size property

## vw and vh Layout

CSS3 introduced new units vw/vh related to the viewport. vw represents width relative to viewport, vh represents height relative to viewport. For elements at any level, when using vw units, 1vw equals one percent of the viewport width.

Similar to percentage layout but more convenient.

## rem Layout

rem: Elements' rem unit style values are dynamically calculated based on the HTML element's font-size value.

em: Multiples of parent element's font size.

By default, html element's font-size is 16px. So 1rem = 16px.

Two common application methods:

1. Combine with media queries to set different root element font sizes, relatively rigid

   ```css
   @media screen and (max-width: 375px) {
     html {
       font-size: 16px
     }
   }
   
   @media screen and (max-width: 320px) {
     html {
       font-size: 12px
     }
   }
   ```

2. Using JS

   ```js
   //Dynamically set root element font size
   function init () {
       // Get screen width
       var width = document.documentElement.clientWidth
       // Set root element font size. Now divided into 10 parts
       document.documentElement.style.fontSize = width / 10 + 'px'
   }
   
   //First load application, set once
   init()
   // Listen for phone rotation events, reset
   window.addEventListener('orientationchange', init)
   // Listen for window changes, reset
   window.addEventListener('resize', init)
   ```

## Common Frontend Layouts

* Document Layout (Most Basic)
* Float Layout
* Position Layout
* Fluid Layout
* Flexible Layout (flex)
* Responsive Layout
  * Media Queries
  * Percentages
  * rem
  * vw,vh
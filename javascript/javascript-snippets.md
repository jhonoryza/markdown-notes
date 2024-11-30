auto load more if we reach bottom of the page

```js
window.onscroll = function(ev) { 
    if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) { 
        @this.dispatch('load-more'); 
    } 
}
```

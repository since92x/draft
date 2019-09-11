# 交叉观察者IntersectionObserver

## tl;dr
* 目的: 观察DOM元素的可见性和位置加以处理。可用于preload、lazyload等等

## API
```javascript
  const handler = (changes) => {
    for (const change of changes) {
      console.log(change.time);               // Timestamp when the change occurred
      console.log(change.rootBounds);         // Unclipped area of root
      console.log(change.boundingClientRect); // target.boundingClientRect()
      console.log(change.intersectionRect);   // boundingClientRect, clipped by its containing block ancestors, and intersected with rootBounds
      console.log(change.intersectionRatio);  // Ratio of intersectionRect area to boundingClientRect area
      console.log(change.target);             // the Element target
    }
  }
  const options = {
    root: null, // 相对于谁
    rootMargin: "0px", // 和css和模型的margin一回事
    threshold: [0, 1.0], // 所观察元素可见可见成程度(0完全消失，50%出现一半)
  }
  const observer = new IntersectionObserver(handler, options);
  // Watch for intersection events on a specific target Element.
  observer.observe(target);
  // Stop watching for intersection events on a specific target Element.
  observer.unobserve(target);
  // Stop observing threshold events on all target elements.
  observer.disconnect();
```
## examples
### lazyload
 ```javascript
    // view
    <img 
      class="lazy"
      src="placeholder-image.jpg"
      data-src="image-to-load-1x.jpg"
    />
    // script
    const fetchImage = (url) => { // It is also named 'preloadImage'
      return new Promise((resolve, reject) => {
        const image = document.createElement('img');
        image.src = url;
        image.onload = resolve;
        image.onerror = reject;
      });
    };
    const loadImage = (image) => {
      const src = image.dataset.src;
      fetchImage(src).then(() => {
        image.src = src;
      })
    };
    document.addEventListener("DOMContentLoaded", function() {
      if ("IntersectionObserver" in window) {
        const observer = new IntersectionObserver(handler, options);
        const handler = (entries, observer) => {
          entries.forEach(entry => {
            if(entry.intersectionRatio > 0) {
              observer.unobserve(entry.target); // 加载则停止观察
              loadImage(entry.target);
            }
          })
        };
        const options = {
          root: null,
          rootMargin: '0px', // this is lazyload
          // rootMargin: '-100px 0px', // this is preload 
          threshold: 0.01, // 当img在root即视口达到{{1%*el.height}}时，开始加载图片
        };
        document.querySelectorAll('img.lazy').forEach(img => observer.observe(img))
      } else {
        // fall back
      }
    })
   ```
### Infinite Scroll 
```vue
  <!-- Observer.vue -->
  <template>
    <div class="observer"/>
  </template>
  <script>
  export default {
    props: ['options'],
    data: () => ({
      observer: null,
    }),
    mounted() {
      const options = this.options || {};
      this.observer = new IntersectionObserver(([entry]) => {
        if (entry && entry.isIntersecting) {
          this.$emit("loadMore");
        }
      }, options);
      this.observer.observe(this.$el);
    },
    destroyed() {
      this.observer.disconnect();
    },
  };
  </script>
```

## 参考
[Intersection Observer](https://w3c.github.io/IntersectionObserver/)

[Intersection Observer v2](https://w3c.github.io/IntersectionObserver/v2/)

[Trust is Good, Observation is Better—Intersection Observer v2](https://developers.google.com/web/updates/2019/02/intersectionobserver-v2)

[延迟加载图像和视频](https://developers.google.com/web/fundamentals/performance/lazy-loading-guidance/images-and-video)

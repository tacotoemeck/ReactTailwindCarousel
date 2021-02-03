## Next JS/ React / Tailwind simple image slider

Recently a part of the ticket I was working on, I needed to build an image carousel. Over the course of my developer journey I’ve done quite a few of them. Having said that, I think there are as many approaches to building those as there are developers doing them.

We only just started using Tailwind recently. I have to say I absolutly love it! It can save you a lot of time writing your css classes and worrying about clashes. On top of that, making your apps responsive gets so much easier.

### Create component

Let's start by creating a functional component. We will be passing on an array of image URLs as a prop. At this point we can also create a state which we'll use to store the index of the currently showing image.

```javascript=
const images = ['/img/img1.png', '/img/img2.png', '/img/img3.png']
```

```javascript=
const Carousel = ({images}) => {
 const [currentImage, setCurrentImage] = React.useState(0);

 return (
     <div className="p-12 flex justify-center w-screen md:w-1/2 items-center">
      <div className="relative w-full">
        <div className="carousel">
          {images.map(img => (
            <div className="w-full flex-shrink-0" key={img}>
              <img src={img} className="w-full object-contain" />
            </div>
          ))}
        </div>
      </div>
    </div>
 )
};

export default Carousel;

```

And a little css that we need. We want our images inlline, scrollable and lastly we want to hide the scrollbar. Sadly last part requires -webkits.

```css=
.carousel {
  display: inline-flex;
  overflow-x: hidden;
/*  scroll snap is a great feature which will center the image on snap on touch screen devices  */
  scroll-snap-type: x mandatory;
/* all below will hide the scrollbar on all browsers.    */
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none; /* For Firefox */
  -ms-overflow-style: none; /* For Internet Explorer and Edge */
  &::-webkit-scrollbar {
    width: 0px;
  }
}
```

Images are placed using inline flex. We then wrap an image in a div with flex-shrink-0 to stop it from 'shrinking' to fit the outer div. Finally the image itself will be 100% of a parent div. Outer div is set with position relative, so we can later place our control buttons using absolute positioning on each side of the image. We can also add a breakpoint for non-mobile screens.

### Create ref for images

Time for some React magic now. We will need to 'reference' our images. For this we will use `React.createRef()` With help of `.reduce` , we can create an object with numbered keys, where value will be an object representing each image. We will use those to later access a ref of a specific image in this array. If unfamiliar with useRef and createRef : https://reactjs.org/docs/refs-and-the-dom.html

```javascript=
const refs = images.reduce((acc, val, i) => {
    acc[i] = React.createRef();
    return acc;
  }, {});
```

Now let's add those refs to our images. Let's add it to our mapping function we created earlier:

```javascript=
  {images.map((img, i) => (
            <div className="w-full flex-shrink-0" key={img} ref={refs[i]}>
              <img src={img} className="w-full object-contain" />
            </div>
          ))}
```

### ScrollIntoView

Now that we referenced each image in the array with createRef. We can then use built-is scrollIntoView API to do exactly what it says on the box - scroll it into your current view! To do so we pass an index of the image, which is then used to identify our current image's ref in 'refs' object. In a normal DOM, you could use your element's id to do the same. More on scrollIntoView

```javascript=
  const scrollToImage = i => {
    // First let's set the index of the image we want to see next
    setCurrentImage(i);

    refs[i].current.scrollIntoView({
      //     Defines the transition animation.
      behavior: 'smooth',
      //      Defines vertical alignment.
      block: 'nearest',
      //      Defines horizontal alignment.
      inline: 'start',
    });
  };
```

And lastly our buttons:

```javascript=
  // Some validation for checking the array length could be added if needed
  const totalImages = images.length;

  // Below functions will assure that after last image we'll scroll back to the start,
  // or another way round - first to last in previousImage method.
  const nextImage = () => {
    if (currentImage >= totalImages - 1) {
      scrollToImage(0);
    } else {
      scrollToImage(currentImage + 1);
    }
  };

  const previousImage = () => {
    if (currentImage === 0) {
      scrollToImage(totalImages - 1);
    } else {
      scrollToImage(currentImage - 1);
    }
  };
```

Before we finish. Let's add those to the carousel:

```javascript=
     <div className="carousel">
          {sliderControl(true)}
          {images.map((img, i) => (
            <div className="w-full flex-shrink-0" key={img} ref={refs[i]}>
              <img src={img} className="w-full object-contain" />
            </div>
          ))}
          {sliderControl()}
        </div>
```

And that's it. It's not perfect as currently scrollIntoView's 'smooth' transition will not work as well on Safari and will jump to the next image instead or scrolling. It would be great to see if we could eliminate the need of external css to keep it all inline using Tailwind only.

I am keen to hear your opinions. I will soon be looking at rebuilding this slider so images 'loop' rather than scroll to the front/back.

Here's the working example https://codepen.io/tacotoemeck/pen/Jjbjgpy?editors=0010

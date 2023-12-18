## Links Needed For the Project

1. [SketchLab](https://sketchfab.com/)
2. [Pixotronix](https://webgi.pixotronics.com/)

## Setting Up the Template

First Run
```shell
npm install
```
Now Use 
```shell
npm run dev
```
to run the local server made with `webpack` . The website will be visible in this  [localhost:5173](http://localhost:5173/index.html) 
for testing the development.


### Replacing the 3d model

Replace the model in the path `/assets` 
Then add the correct path to `load` in the **`index.ts`** file.Modify the following line
```ts
await viewer.load("./assets/scene.glb")
```

> Run the system to see if everything is working as expected or not

## Resizing the webgl component

As you can see the render is not properly covering the entire webpage as we expect. So do some changes in `style.css` .

- change the height and width to 100 vh and vw respectively, in the `webgi-canvas-container`
- Remove the padding , margin , box-shadow and border radius as well.
- Add a margin and padding of 0 to the body.
- Remove the overflow hidden to overflow-x hidden.

The final CSS should look like this 
```css
body {
  font-family: sans-serif;
  margin: 0;
  padding: 0;
  overflow-x:'hidden';
}

#webgi-canvas {
  width: 100%;
  height: 100%;
}

#webgi-canvas-container{
  width: 100vw;
  height: 100vh;
}
```



### Bonus: Want to get rid of the tooltip in the website 
Just remove the `TweakpaneUiPlugin` plugin from the `index.ts` file . And remove this or comment out this
```ts
const uiPlugin = await viewer.addPlugin(TweakpaneUiPlugin)
// Add plugins to the UI to see their settings.
uiPlugin.setupPlugins<IViewerPlugin>(TonemapPlugin, CanvasSnipperPlugin)
```




## Add The HTML and CSS for your Website 

>You can use a  wireframing tool like **figma** or **Abobe Xd** for faster execution .

Add your font in the `styles.css` if you like.



### Can not see any changes or any elements in my page

So this happens due to the weired behavior of webgl or threejs canvas behaviour, you can use 
```css
position: 'fixed';
```
This works but you have to keep in mind that the webgl canvas is on top of everything , you can use a z-index, but the **absolute position is must because we want to place the canvas at the center every time** .
So much this and that, you can use this css property and it should work fine (in my case I will edit it out if any change is needed).
```css
#webgi-canvas {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
}
```


> So to make things clear we gotta place the text below the 3d element , but to do that we need the position as well as the transperency of the canvas to see the things behind it clearly.

### Making the canvas transperent

Go to the `index.ts` file, then modify the following code
```ts
const viewer = new ViewerApp({
	canvas: document.getElementById('webgi-canvas') as HTMLCanvasElement,
    useRgbm:false // add this property
    })
```


### Scrolling Problem 
As you can see you can not scroll the website using the default scroll , you can use the up and down arrow to do this, so what we have to do is we have to **disable the pointer events in the canvas** so that the scroll does not trigger the zoom in zoom out thing.
So in the `styles.css` modify it like
```css
#webgi-canvas-container{
  width: 100vw;
  height: 100vh;
  pointer-events: none; /* add this */ 
}
```
Yeah, now you can not move the model too, but do not worry about that as we will use gsap for that.


## Install GSAP

To install gsap, we will use this
```shell
npm install gsap
```
now add this line to the ts file after importing the css and other stuff
```ts
import gsap from 'gsap'
```
and then import the `ScrollTrigger` plugin
```ts
import { ScrollTrigger } from "gsap/all"
```
Register the plugin by 
```ts
gsap.registerPlugin(ScrollTrigger)
```

### Set Up the Function for Animation 

The function you can declare **inside** the `setupViewer` function declaration. Something like `setupScrollAnimation` to keep the naming convention. 

For the animation we will need a **timeline** , so we will make one using 
```ts
const tl = gsap.timeline()
```

At this point we need camera and object access. To do this go under the `AssetManagerPlugin` line
```ts
const manager = await viewer.addPlugin(AssetManagerPlugin)
```
access the camera by
```ts
const camera = viewer.scene.activeCamera
```
This selects the active camera from the scene. Now to get the position of the camera
```ts
const position = camera.position
```
We will be using some kind of control , and to control that we need a target to control. Let us access the target by adding 
```ts
const target = camera.target
```
So the final structure of the code after the ``AssetManagerPlugin`` will be
```ts
const camera = viewer.scene.activeCamera
const position = camera.position
const target = camera.target
```

Now lets get back to our function, we will change the position of the camera using gsap. 
```ts
    function setupScrollAnimation()
    {
        const tl = gsap.timeline()

        // 1st part
        tl.to(position , { x: 5 , duration:4})
    }
```
Just a simple one to test , now call the function inside the ``setupViewer`` function (We were already inside that function) . 


### No animation?

This happens because we can not directly change ThreeJs property by using gsap directly, we have to use a **WEBGI update function**
Let's do it like this 
```ts
    let needsUpdate = true
    function onUpdate()
    {
        needsUpdate = true
        viewer.renderer.resetShadows()
    }
```
We have to rerender the entire webGI as we progress.
Now we need to add a **Hook** in order to see if the trigger for the redender has ended or not.
Do it like this
```ts
    viewer.addEventListener('preFrame' , ()=>{
        if(needsUpdate)
        {
            camera.positionUpdated(false)
            camera.targetUpdated(true)
            needsUpdate = false
        }
    })
```
Now we have to pass the WEBGI update function into the GSAP code or animation function
```ts
    function setupScrollAnimation()
    {
        const tl = gsap.timeline()
        // 1st part
        tl.to(position , { x: 5 , duration:4 , onUpdate}) // <- here
    }
```

Now all left is to set up the scrolltrigger part.

```ts
scrollTrigger:{
                trigger: '#two', // your second section 
                markers:true, // to see the things better
                scrub:2 // you can see why
            } ,
```













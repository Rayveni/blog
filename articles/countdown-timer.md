
# How to create Countdown Timer on Pure CSS

Welcome to a short tutorial on how to create a **circle** countdown timer.
Here's how to create a countdown using conic-gradient (Pure **CSS**) to display progress and requestAnimationFrame (**JS**) for animation without using 3rd party libraries.

<iframe height="500" width="500" style="width: 100%;" scrolling="no" title="countdown-timer" src="https://codepen.io/Rayveni/embed/RwYgwra?default-tab=result" frameborder="no" loading="lazy" allowtransparency="true" allowfullscreen="true">
  See the Pen <a href="https://codepen.io/Rayveni/pen/RwYgwra">
  countdown-timer</a> by Rayveni (<a href="https://codepen.io/Rayveni">@Rayveni</a>)
  on <a href="https://codepen.io">CodePen</a>.
</iframe>

## Example code download

[![https://github.com/Rayveni/countdown-timer](https://github.com/Rayveni/blog/blob/main/articles/img/octocat_mini.png?raw=true)](https://github.com/Rayveni/countdown-timer)[countdown-timer](https://github.com/Rayveni/countdown-timer)

## Solution Explanation
Our task is divided into two parts:.
- displaying progress by drawing a rotating circle.
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/countdown-timer/circles.jpg?raw=true)
- progress animation, ideally with fluid transitions.

### Drawing circle rotation
I have discovered the following variants:
- Layer composition: 
   *  **bottom layer**- solid circle for 100% progress
   * **middle layer**- a half-circle rotation. (e.g.CSS:```transform: rotate(136deg);```)
   * **Top layer**- static half circle ,hidden for 0-50% progress and visible for 50%-100%

![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/countdown-timer/layer.jpg?raw=true)

To make job done, half circles must be made; this is possible with CSS **Clip** attribute. See the [article](https://www.tiagoalmeida.dev/posts/making-a-pure-css-circular-progress-bar.html) with an explanation if you have an interest. **Note**: Clip is deprecated in the current version of CSS; use [clip-path](https://developer.mozilla.org/en-US/docs/Web/CSS/clip-path) instead ([clip-path generator](https://www.cssportal.com/css-clip-path-generator/) would be helpful)
- JS's paintings on Canvas or SVG  J- nice option 
- conic gradient (css) - I found this the simplest and most laconic.
 
## Conic-gradient

Simply using a border-radius of 50% will result in a circle in CSS. Setting up the gradient is the next step. 
```html
<html>
   <body>
      <style>
         .timer {
		         width:30vmin;
				 height:30vmin; 
				 border:1vh  solid green;
				 border-radius:50%;
				 background:conic-gradient(red 44%, 0, green )
				 }
      </style>
      <div class="timer">
      </div>
   </body>
</html>
```
We can pass values ​​in percent or degrees to the conical gradient to show progress. 
```CSS 
background:conic-gradient(red 44%, 0, green ) 
```
At this point our circles should look like a pie chart ,so we need to add a circle overlay.
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/countdown-timer/pie_chart.jpg?raw=true)

There are different ways to do this, my approach is to manipulate the thickness of the border and  fill the background with other gradients (e. g. linear or radial).
```CSS
         .timer {
		         width:30vmin;
				 height:30vmin; 
				 border:1vh  solid transparent;
				 border-radius:50%;
				 background:linear-gradient(white, white) content-box no-repeat,conic-gradient(red 44%, 0, green ) border-box;
				 
				 }
```

![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/countdown-timer/donut_chart.jpg?raw=true)
## JS code

We just have to implement:
 - countdown
 - timer animation

### Countdown
For provided example I use 3 minutes countdown, and its value is stored in global variable  **distance_minutes** (you are free to change it)
Algorithm:
 1. Calculate distance_date (add  **distance_minutes**  to current timestamp and store in global variable
 2. Calculate  **countdown_value**=current timestamp  -distance_date 
 3. Use integer division and remainder division to extract remain second and minutes from the **countdown_value**
 4. Pass values to DOM directly or use CSS variables.

### Timer animation
requestAnimationFrame to start animation:
```javascript
animation_id = window.requestAnimationFrame()
```
and cancelAnimationFrame to stop animation
```javascript
 window.cancelAnimationFrame(animation_id);
 ```

## Layout
To compose  all the elements we will use CSS flexbox.
First, create a structure:
```html
      <div class="clock_container">
		<div class="timer">
		   <div class="donat outer-circle">
			  <p class="global_percent center_text" ></p>
			  <div class="donat inner-circle">
				 <p class="local_percent center_text"></p>
				 <p class="clock-time clock-timer"></p>
			  </div>
		   </div>
		</div>
         <div class="timer_controls">
            <button onclick="start_timer()" class="player-control">
               <svg   fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="feather">
                  <polygon points="5 3 19 12 5 21 5 3"></polygon>
               </svg>
            </button>
            <button onclick="stop_timer()" class="player-control">
               <svg  class="feather" fill="white" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" >
                  <rect x="3" y="3" width="18" height="18" rx="2" ry="2"></rect>
               </svg>
            </button>
		
         </div>
      </div>
```
Next ,create  flex layout in CSS(I applied temporary background colors for layout to make design more visual):
```CSS
  .clock_container {
     display: flex;
     flex-wrap: wrap;
     justify-content: space-between;
     background:pink;
     width:50vmin;
     height:auto;
     aspect-ratio: 1 / 1.2 ;
     flex-direction: column;
     justify-content: flex-start 
}
 .timer {
     width: 100%;
     aspect-ratio: 1 / 1 ;
     background:red;
}
 .timer_controls{
     width: 100%;
     flex-grow: 1;
     position: relative;
     display:flex;
     background:black;
     flex-direction: row;
     justify-content: center;
     align-items: center;
     z-index:2;
}
 .player-control{
     position: relative;
     height:70%;
     aspect-ratio: 1 / 1 ;
     border:1vmin solid green;
     margin: auto 5%;
     border-radius: 50%;
}
```
![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/countdown-timer/layout1.jpg?raw=true)

Next, draw circles and add text:
```CSS
 .clock-time:before {
     content: var(--timer-minutes-seconds,"03:00");
}
 .center_text:before{
     position:absolute;
     top:3%;
     left:50%;
     transform: translate(-50% , 0%);
     z-index: 5;
}
 .global_percent:before {
     content: var(--global_percent_val,"100%");
}
 .local_percent:before {
     content: var(--local_percent_val,"0%");
}
 .clock_container {
     display: flex;
     flex-wrap: wrap;
     justify-content: space-between;
     background:pink;
     width:50vmin;
     height:auto;
     aspect-ratio: 1 / 1.2 ;
     flex-direction: column;
     justify-content: flex-start ;
     margin-left:10vh;
     margin-top:10vh;
     color:white;
     font-size: 2.5vmin;
}
 .timer {
     width: 100%;
     aspect-ratio: 1 / 1 ;
     background:red;
     display: flex;
     align-items: center;
     justify-content: center;
}
 .timer_controls{
     width: 100%;
     flex-grow: 1;
     position: relative;
     display:flex;
     background:black;
     flex-direction: row;
     justify-content: center;
     align-items: center;
     z-index:2;
}
 .player-control{
     position: relative;
     height:70%;
     aspect-ratio: 1 / 1 ;
     border:1vmin solid green;
     margin: auto 5%;
     border-radius: 50%;
}
 .donat {
     aspect-ratio: 1 / 1 ;
     border-radius: 50%;
     border:1.5vmin solid transparent;
}
 .outer-circle{
     position: relative;
     width:90%;
     background: linear-gradient(grey, grey) content-box no-repeat, conic-gradient(yellow 70%, 0, green ) border-box;
     z-index:0;
     display:flex;
     justify-content: center;
     align-items: center;
}
 .inner-circle{
     position: relative;
     width:70%;
     z-index:1;
     background: linear-gradient(grey, grey) content-box no-repeat, conic-gradient(blue 25%, 0, green ) border-box;
     display:flex;
     justify-content: center;
}
 .clock-timer{
     font-size: 10vh;
     color:purple;
}
 
```


![enter image description here](https://github.com/Rayveni/blog/blob/main/articles/countdown-timer/layout2.jpg?raw=true)

Voilà! Circle countdown timer created! The next step is to link the JS file and replace static values ​​with reliable CSS variables for displaying progress. This step is quite trivial and not be considered.

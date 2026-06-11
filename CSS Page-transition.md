```css
@view-transition{
	navigation: auto;
}

::view-transition-group(root){
	animation-duration: 1s; 
	animation-timing-function: ease;
	/*Define the duration of animation, root marks the name of element to be transitioned, root will cause whole page to transition.*/
}
/*Group determines the properties of transition of both new and old*/

::view-transition-old(root){
	animation-name: slide-out;
}
::view-transition-new(root){
	animation-name: slide-in;
}
/*old slides out, new slides in*/
/*old refers to the page transitioning from*/
/*new refers to the page transitioning to*/
@keyframes slide-out{
	to{translate: -100vw;
	}
}
@keyframes slide-in{
	from{translate: 100vw;
	}
}

```

# Transition on an element rather than whole page

```css
::view-transition-group(page-content){
	animation-duration: 1s; 
	animation-timing-function: ease;
	/*Define the duration of animation, root marks the name of element to be transitioned, root will cause whole page to transition.*/
}
/*Group determines the properties of transition of both new and old*/

::view-transition-old(page-content){
	animation-name: slide-out;
}
::view-transition-new(page-content){
	animation-name: slide-in;
}

main{
	view-transition-name: page-content;
}

/*Now transition only happens in main element*/
```


# Transitioning of element that exists on both pages
```css
.card image, .hero-image{
	view-transition-name: article-image;
}
```


# Gotta respect if user has prefers-reduced-motion
```css
@media(prefers-rediced-motion: no-preference){
::view-transition-group(page-content){
	animation-duration: 1s; 
	animation-timing-function: ease;
}

::view-transition-old(page-content){
	animation-name: slide-out;
}
::view-transition-new(page-content){
	animation-name: slide-in;
}

main{
	view-transition-name: page-content;
}
/*If user prefers reduced-motion this transition wont take place.*/
}
```
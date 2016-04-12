WoW
===

Before you dive into details, you can try [animate]() (also [wow](https://github.com/matthieua/WOW) to trigger it on scroll):

~~~
<html>
  <head>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/3.4.0/animate.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/wow/1.1.2/wow.js">
  </script>
  <script>
    new WOW().init();
  </script>
</head>
  <body>
    <div class="animated fadeInRightBig">
      Content to Reveal Here
    </div>
  </body>
</html>
~~~

Transform property
===

~~~
div {
  transform: scale(2,3);
}
~~~

2D transforms:

* `translate(50px, 100px)` is 50px right and 100px down
* `rotate(20deg)` clockwise
* `scale(2,3)` width 2 times and height 3 times bigger
* `skewX(20deg)` `skewY(-10deg)` and `skew(20deg,-10deg)`
* `matrix(6 params)`

another usefull property is `transform-origin: 50% 50% 0` which
move the base of transform, on x, y and z axis (for 3D).

3D transforms:

* `rotateX(20deg)` around X axis (horizontal)
* `rotateY(20deg)` around Y axis (vertical)
* `rotateZ(20deg)` around Z axis (toward user)
* translate3d scale3d rotate3d

usefull property is `perspective: 400px` is how many pixes is
3D element placed from the view and `perspective-origin: 10% 10%` bootom
position on x and y axis.

Transition property
===

Transition allows that element change property smoothly, over given duration.

~~~
div {
  transition: width 2s ease-out 1s;
  // or
  transition-property: width;
  transition-duration: 2s;
  transition-timing-function: ease-out;
  transition-delay: 1s;
}
~~~

usually we change properties on hover.

Animation property
===

~~~
div {
  animation: example 5s linear 2s infinite alternate;
  // or
  animation-name: example;
  animation-duration: 5s;
  animation-timing-function: linear;
  animation-delay: 2s;
  animation-iteration-count: infinite;
  animation-direction: alternate;

  animation-fill-mode: forwards;
}

@keyframes example {
  0% { width: 10px; }
  25% { width: 100px; }
  100% { width: 110px; }
}

so you don't need user action to animate...
~~~

# Angular

Include `ngAnimate` and for `ngRepeat` use `ng-enter` and `ng-enter-active` [old
doc](https://code.angularjs.org/1.2.22/docs/api/ngAnimate/service/$animate). We
could just also watch `ng-leave`, `ng-move` with ngRepeat, `ng-add` and
`ng-remove` with ngClass, `ng-hide` with ngShow.

There is nice [ng-fx](https://github.com/AngularClass/ng-fx)
[egghead](https://egghead.io/lessons/angularjs-introduction-to-ngfx-for-angular-animations)
so you can use their animations. To install just run `bower instal ng-fx --save`
and include `ngFx` in your module file. Just add some of [the
supported](https://github.com/AngularClass/ng-fx/blob/master/animationList.txt)
animations like `div(ng-repeat="item in items" class="fx-zoom-left fx-speed-500
fx-ease-sin fx-stagger-10")`

https://24ways.org/2010/intro-to-css-3d-transforms/

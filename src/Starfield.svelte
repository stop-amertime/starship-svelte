<script lang="ts">
import jq from 'jquery';

var sf = {
	parralax: true,
	parralaxSens: 0.01,
	starDensity: 0.6,
	canvas: null,
	container: null,
	direction: 'right', // left || right
	dx: 0.05, // horizontal velocity
	cw: null, // canvasWidth
	ch: null, // canvasHeight
	ctx: null, // context
	parralaxVal: null,
	numStars: null, // stars per 100px square
	stars: [], // Stars in the scene, starts as an empty array
	layers: [],
};

//start on
jq(document).ready(function () {
	sfSetup();
	if (sf.parralax == true) {
		parralax();
	}
	let requestID = requestAnimationFrame(starfieldAnimate);
});

// redraw on resize
///////////////// REMOVED REDRAW ON RESIZE DUE TO MOBILE ISSUE.
/*
jq(window).resize(function(){
  waitForFinalEvent(function(){
    //alert('fire');
    sfSetup();
  },400);
});
*/

function starfieldAnimate() {
	let requestID = requestAnimationFrame(starfieldAnimate);
	clearCanvas();
	animatePanel();
}

function parralax() {
	var distanceFromTop;
	sf.parralaxVal = jq(window).scrollTop();
	jq(window).on('scroll', function () {
		sf.parralaxVal = jq(window).scrollTop() * -1;
	});
}

function sfSetup() {
	// Set / reset values since we may initialize several times
	sf.stars = [];
	sf.layers = [];

	sf.canvas = jq('.js-canvas');
	sf.container = jq('.js-container');

	sf.ctx = sf.canvas[0].getContext('2d');
	sf.cw = sf.container.outerWidth();
	sf.ch = sf.container.outerHeight();

	sf.canvas.attr({
		width: sf.cw,
		height: sf.ch,
	});

	// Caculate how many stars should be on screen per 40px grid.
	var area = (sf.cw * sf.ch) / (40 * 40);
	sf.numStars = area * sf.starDensity;

	// randomly calculate the positions and sf.sizes of the stars
	// and store the positions in an array to be called / modified later.

	var max = 4;
	var med = 2;
	var min = 1;

	var starCount = 0;
	var starFreqs = [0, 50, 12, 4, 1];

	// create stars
	while (starCount <= sf.numStars) {
		var size = rand(1, 4);
		let count = starFreqs[size];

		// create the instances of the stars
		createStar(size, count);
		starCount += count;
	}

	// create a panel for each star set
	for (var i = min; i <= max + 1; i++) {
		var buffer = document.createElement('canvas');
		buffer.width = sf.cw;
		buffer.height = sf.ch;
		var bufferContext = buffer.getContext('2d');

		// render stars to a layer
		renderStars(i, bufferContext);

		sf.layers.push({ x: 0, s: i, buffer: buffer });
	}
}

// star creation
function createStar(size, numberToCreate) {
	for (var i = 0; i < numberToCreate; i++) {
		var x = rand(4, sf.cw - 4);
		var y = rand(4, sf.ch - 4);

		sf.stars.push({
			x: x,
			y: y,
			s: size,
		});
	}
}

function renderStars(size, bufferContext) {
	for (var i = 0; i < sf.numStars; i++) {
		// So that we don't have to do a sf.stars[i] a million times below
		// we just grab and hold a reference to the star we're working on
		// and use the x, y and s properties throughout the code
		var star = sf.stars[i];

		// skip the star if it's not the appropriate size
		if (star.s != size) {
			continue;
		}

		var color;

		// big star color
		if (star.s == 4) {
			color = 'rgba(255, 255, 255, 0.9)';
		} else if (star.s == 3) {
			color = 'rgba(255, 255, 255, 0.7)';
		} else if (star.s == 2) {
			color = 'rgba(255, 255, 255, 0.5)';
		} else if (star.s == 1) {
			color = 'rgba(255, 255, 255, 0.35)';
		}

		bufferContext.beginPath();
		bufferContext.arc(star.x, star.y, star.s, 0, Math.PI * 2, true);
		bufferContext.closePath();
		bufferContext.fillStyle = color;
		bufferContext.fill();
	}
}

// animate panels of stars
// this grabs the generated star panels and animates them
// every frame
function animatePanel() {
	// draw each layer on screen
	for (var i = 0; i < sf.layers.length; i++) {
		var layer = sf.layers[i];
		var size = layer.s;

		// calculate parallax ypos
		if (sf.parralax == true) {
			layer.y = sf.parralaxVal * (size * sf.parralaxSens);
		}

		// calculate xpos
		// if left
		if (sf.direction == 'left') {
			layer.x = layer.x <= -sf.cw ? 0 : layer.x + -(layer.s * sf.dx);
			layer.x2 = layer.x + sf.cw;
		}
		// if right
		if (sf.direction == 'right') {
			layer.x = layer.x >= sf.cw ? 0 : layer.x + layer.s * sf.dx;
			layer.x2 = layer.x - sf.cw;
		}

		// check if layer is on screen / move it to other side
		// + move based on panels star size
		sf.ctx.drawImage(layer.buffer, layer.x, layer.y);
		sf.ctx.drawImage(layer.buffer, layer.x2, layer.y);

		// if parallax, draw a second set of tiles under the canvas
		// that become visible when scrolling
		if (sf.parralax == true) {
			sf.ctx.drawImage(layer.buffer, layer.x, layer.y + sf.ch);
			sf.ctx.drawImage(layer.buffer, layer.x2, layer.y + sf.ch);
		}
	}
}

// clear canvas every frame
function clearCanvas() {
	sf.ctx.clearRect(0, 0, sf.cw, sf.ch);
} // should only clear the animated space

//-----------------------------------------------------------
// General Functions

// Random Number function
function rand(from, to) {
	return Math.floor(Math.random() * (to - from + 1) + from);
}

// Resize Delay Function
var waitForFinalEvent = (function () {
	var timers = {};
	return function (callback, ms, uniqueId) {
		if (!uniqueId) {
			uniqueId = "Don't call this twice without a uniqueId";
		}
		if (timers[uniqueId]) {
			clearTimeout(timers[uniqueId]);
		}
		timers[uniqueId] = setTimeout(callback, ms);
	};
})();
</script>

<img src="starbg.jpg" alt="Starfield" />
<div class="canvas-container fixed-top js-container" id="starfield">
	<canvas class="100 canvas-container__canvas js-canvas">Starfield Canvas</canvas>
</div>

<style>
img {
	position: fixed;
	width: 100vw;
	height: 100vh;
	filter: grayscale(0.9) brightness(1) contrast(1.4);
	z-index: -30;
	object-fit: cover;
}

.canvas-container {
	background: rgba(0, 0, 0, 0);
	height: 120vh;
	width: 100%;
	z-index: -20;
	position: fixed;
}

.canvas-container__canvas {
	position: absolute;
	height: 100%;
	width: 100%;
	top: 0;
	left: 0;
}
</style>

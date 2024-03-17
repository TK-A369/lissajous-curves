<script lang="ts">
	import { onMount } from 'svelte';

	let paramAAmp: number = 150;
	let paramBAmp: number = 150;
	let paramAAngFreq: number = 2;
	let paramBAngFreq: number = 3;
	let paramDelta: number = 0;

	let paramVelocityVectorSize: number = 100.0;
	let paramAccelerationVectorSize: number = 100.0;

	let paramT: number = 0;
	let paramStep: number = 0.05;
	let paramSpeed: number = 1;
	$: console.log(`t = ${paramT}`);
	let timeRunning: boolean = false;
	let timeInterval: number | null = null;

	let drawCurve: ((time: number) => void) | null = null;
	onMount(() => {
		const canvas: HTMLCanvasElement = document.getElementById(
			'lissajous-canvas'
		) as HTMLCanvasElement;
		const ctx: CanvasRenderingContext2D = canvas.getContext('2d') as CanvasRenderingContext2D;

		function drawVector(
			startX: number,
			startY: number,
			sizeX: number,
			sizeY: number,
			color: string
		) {
			ctx.strokeStyle = color;
			ctx.lineWidth = 2;
			ctx.lineCap = 'round';
			ctx.beginPath();
			ctx.moveTo(startX, startY);
			ctx.lineTo(startX + sizeX, startY + sizeY);

			const arrowTipLength = 4;
			const vectorDirection = Math.atan2(sizeY, sizeX);
			const arrowTip1X = Math.cos(Math.PI + vectorDirection - Math.PI / 4) * arrowTipLength;
			const arrowTip1Y = Math.sin(Math.PI + vectorDirection - Math.PI / 4) * arrowTipLength;
			const arrowTip2X = Math.cos(Math.PI + vectorDirection + Math.PI / 4) * arrowTipLength;
			const arrowTip2Y = Math.sin(Math.PI + vectorDirection + Math.PI / 4) * arrowTipLength;

			ctx.lineTo(startX + sizeX + arrowTip1X, startY + sizeY + arrowTip1Y);
			ctx.moveTo(startX + sizeX, startY + sizeY);
			ctx.lineTo(startX + sizeX + arrowTip2X, startY + sizeY + arrowTip2Y);
			ctx.stroke();
		}

		drawCurve = (time: number) => {
			const width = canvas.width;
			const height = canvas.height;

			ctx.clearRect(0, 0, width, height);

			// Generate Lissajous curve points
			let coords: Array<{ x: number; y: number }> = [];
			for (let currTime = 0; currTime <= time; currTime += paramStep) {
				coords.push({
					x: paramAAmp * Math.sin(paramAAngFreq * currTime + paramDelta),
					y: paramBAmp * Math.sin(paramBAngFreq * currTime)
				});
			}
			let position: { x: number; y: number } = {
				x: paramAAmp * Math.sin(paramAAngFreq * time + paramDelta),
				y: paramBAmp * Math.sin(paramBAngFreq * time)
			};
			let velocity: { x: number; y: number } = {
				x: paramAAmp * paramAAngFreq * Math.cos(paramAAngFreq * time + paramDelta),
				y: paramBAmp * paramBAngFreq * Math.cos(paramBAngFreq * time)
			};
			let acceleration: { x: number; y: number } = {
				x: -paramAAmp * paramAAngFreq * paramAAngFreq * Math.sin(paramAAngFreq * time + paramDelta),
				y: -paramBAmp * paramBAngFreq * paramBAngFreq * Math.sin(paramBAngFreq * time)
			};

			ctx.save();
			ctx.translate(width / 2, height / 2);

			// Draw Lissajous curve
			ctx.strokeStyle = '#000000';
			ctx.lineWidth = 2;
			ctx.lineCap = 'round';
			ctx.beginPath();
			coords.forEach((point) => {
				ctx.lineTo(point.x, point.y);
			});
			ctx.stroke();

			// Draw current position
			ctx.fillStyle = '#FF0000';
			ctx.lineWidth = 1;
			ctx.beginPath();
			ctx.arc(position.x, position.y, 3, 0, 2 * Math.PI, false);
			ctx.fill();

			//Draw velocity vector
			const velocityVectorDivider = Math.max(paramAAmp * paramAAngFreq, paramBAmp * paramBAngFreq);
			drawVector(
				position.x,
				position.y,
				(velocity.x * paramVelocityVectorSize) / velocityVectorDivider,
				(velocity.y * paramVelocityVectorSize) / velocityVectorDivider,
				'#00FF00'
			);

			//Draw acceleration vector
			const accelerationVectorDivider = Math.max(
				paramAAmp * paramAAngFreq * paramAAngFreq,
				paramBAmp * paramBAngFreq * paramBAngFreq
			);
			drawVector(
				position.x,
				position.y,
				(acceleration.x * paramAccelerationVectorSize) / accelerationVectorDivider,
				(acceleration.y * paramAccelerationVectorSize) / accelerationVectorDivider,
				'#0000FF'
			);

			ctx.restore();
		};
	});

	$: if (drawCurve !== null) {
		drawCurve(paramT);
	}
</script>

<canvas id="lissajous-canvas" width="500" height="500"> </canvas>

<h1>Krzywe Lissajous</h1>

<div>
	<label for="param-a-amp">A = </label>
	<input
		type="number"
		id="param-a-amp"
		name="param-a-amp"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramAAmp}
	/>
	<br />
	<label for="param-a-ang-freq">a = </label>
	<input
		type="number"
		id="param-a-ang-freq"
		name="param-a-ang-freq"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramAAngFreq}
	/>
	<br />

	<label for="param-b-amp">B = </label>
	<input
		type="number"
		id="param-b-amp"
		name="param-b-amp"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramBAmp}
	/>
	<br />
	<label for="param-b-ang-freq">b = </label>
	<input
		type="number"
		id="param-b-ang-freq"
		name="param-b-ang-freq"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramBAngFreq}
	/>
	<br />

	<label for="param-delta">ẟ = </label>
	<input
		type="number"
		id="param-delta"
		name="param-delta"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramDelta}
	/>
	<br />

	<label for="param-time">t = </label>
	<input type="number" id="param-t" name="param-t" contenteditable="true" bind:value={paramT} />
	<br />
	<input type="range" id="param-t-slider" min={0} max={10} step={paramStep} bind:value={paramT} />
	<input
		type="button"
		id="param-time-running"
		value={timeRunning ? 'Stop' : 'Start'}
		on:click={() => {
			timeRunning = !timeRunning;
			if (timeRunning) {
				if (timeInterval === null) {
					timeInterval = setInterval(
						() => {
							paramT += paramStep;
						},
						(1000 * paramStep) / paramSpeed
					);
				}
			} else {
				if (timeInterval !== null) {
					clearInterval(timeInterval);
					timeInterval = null;
				}
			}
		}}
	/>
	<br />

	<label for="param-speed">prędkość = </label>
	<input
		type="number"
		id="param-speed"
		name="param-speed"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramSpeed}
	/>
	<br />

	<label for="param-step">krok = </label>
	<input
		type="number"
		id="param-step"
		name="param-step"
		min="0"
		max=""
		step="any"
		contenteditable="true"
		bind:value={paramStep}
	/>
	<br />
</div>

<p>x(t) = A*sin(a*t+ẟ)</p>
<p>y(t) = B*sin(b*t)</p>

<p>Powyżej można modyfikować parametry krzywej Lissajous.</p>
<p>Zmienna prękość wpływa na to, jak szybko rysowana jest krzywa.</p>
<p>
	Natomiast zmienna krok reguluje co jaki czas obliczane są współrzędne krzywej. Im mniejsza
	wartość, tym dokładniejsza krzywa. Niska jakość może być szczególnie zauważalna gdy a i/lub b są
	wysokie, a krok niski.
</p>
<p>Przycisk start/stop włącza i wyłącza rysowanie krzywej o danych parametrach.</p>
<p>
	Przed modyfikacją parametrów zaleca się zatrzymanie rysowania, a po niej ustawienie t na 0
	(używając pola tekstowego lub suwaka) i następnie wznowienie rysowania.
</p>

<p>Czerwony punkt wskazuje gdzie obecnie jest rysowana krzywa.</p>
<p>Wektor prędkości jest zaznaczony na zielono.</p>
<p>Wektor przyspieszenia jest zaznaczony na niebiesko.</p>

<style>
	p,
	h1,
	label {
		font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
	}

	#lissajous-canvas {
		border: 1px solid black;
		width: min(60vw, 95vh);
		height: min(60vw, 95vh);
		float: right;
	}
</style>

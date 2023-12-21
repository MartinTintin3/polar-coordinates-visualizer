<script>
	import { onDestroy, onMount } from "svelte";

	/** @type {HTMLCanvasElement} */
	let canvas;
	/** @type {CanvasRenderingContext2D} */
	let ctx;

	let width;
	let height;

	let grid_color = "#BBBBBB";
	let axis_color = "#000000";
	let point_color = "#FF0000";

	let point = { r: 2, theta: 0 };

	let interval = 10;

	const addZeroes = num => num.toFixed(Math.max(num.toString().split('.')[1]?.length, 2) || 2)


	const decimals = (n, d) => {
		const factor = 10 ** d;
		const result = addZeroes(Math.round(n * factor) / factor);
		return result;
	};

	function pad(num, size) {
		if (num >= 0) {
			while (num.length < size) num = "0" + num;
			return num;
		} else {
			num = num.slice(1);
			while (num.length < size) num = "0" + num;
			return "-" + num;
		}
	}

	const update = () => {
		ctx.clearRect(0, 0, width, height);

		const center = { x: width / 2, y: height / 2 };

		ctx.fillStyle = grid_color;

		for (let r = interval; r < Math.max(width, height); r += interval) {
			ctx.strokeStyle = grid_color;
			
			ctx.beginPath();
			ctx.arc(center.x, center.y, r, 0, 2 * Math.PI);
			ctx.stroke();
		}

		ctx.fillStyle = grid_color;

		for (let x = center.x; x < width; x += interval) {
			if (x === center.x) continue;
			ctx.fillRect(x, 0, 1, height);
			ctx.fillRect(width - x, 0, 1, height);
		}

		for (let y = center.y; y < height; y += interval) {
			if (y === center.y) continue;
			ctx.fillRect(0, y, width, 1);
			ctx.fillRect(0, height - y, width, 1);
		}

		ctx.fillStyle = axis_color;
		ctx.fillRect(center.x, 0, 1, height);
		ctx.fillRect(0, center.y, width, 1);

		ctx.fillStyle = point_color;

		const point_x = point.r * Math.cos(point.theta * Math.PI / 180);
		const point_y = point.r * Math.sin(point.theta * Math.PI / 180);

		ctx.beginPath();
		ctx.arc(center.x + point_x * interval, center.y - point_y * interval, 2, 0, 2 * Math.PI);
		ctx.fill();
	};

	onMount(() => {
		interval = 20;

		canvas = document.getElementById("canvas");
		ctx = canvas.getContext("2d");

		width = canvas.width;
		height = canvas.height;

		if (window.devicePixelRatio > 1) {
			canvas.width = width * window.devicePixelRatio;
			canvas.height = height * window.devicePixelRatio;
			canvas.style.width = width + "px";
			canvas.style.height = height + "px";
			ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
		}

		update();

		let dragging = false;

		canvas.addEventListener("mousedown", e => {
			dragging = true;
		});

		canvas.addEventListener("mouseup", e => {
			dragging = false;
		});

		canvas.addEventListener("mousemove", e => {
			if (!dragging) return;
			const rect = canvas.getBoundingClientRect();
			const x = e.clientX - rect.left;
			const y = e.clientY - rect.top;

			const center = { x: width / 2, y: height / 2 };

			const dx = x - center.x;
			const dy = y - center.y;

			point.r = Math.sqrt(dx * dx + dy * dy) / interval;
			point.theta = Math.atan2(-dy, dx) * 180 / Math.PI;

			update();
		});
	});
</script>

<div class="content">
	<h1>Polar Coordinate Visualizer</h1>

	<div class="info">
		<p>Polar: ({decimals(point.r, 2)}, {decimals(point.theta, 2)}°)</p>
		<p>Cartesian: ({decimals(point.r * Math.cos(point.theta * Math.PI / 180), 2)}, {decimals(point.r * Math.sin(point.theta * Math.PI / 180), 2)})</p>
	</div>

	<canvas id="canvas" width="400" height="400" style="border: 1px solid black" />

	<div class="setting">
		<label for="interval">Interval: </label>
		<input type="range" name="interval" id="interval" bind:value={interval} on:input={update} min=2 max=200>
		<span>{interval}px</span>
	</div>

	<div class="setting">
		<label for="grid_color">Grid Color: </label>
		<input type="color" name="grid_color" id="grid_color" bind:value={grid_color} on:input={update}>
	</div>

	<div class="setting">
		<label for="axis_color">Axis Color: </label>
		<input type="color" name="axis_color" id="axis_color" bind:value={axis_color} on:input={update}>
	</div>

	<div class="setting">
		<label for="point_color">Point Color: </label>
		<input type="color" name="point_color" id="point_color" bind:value={point_color} on:input={update}>
	</div>

	<div class="setting">
		<label for="point_r">Point R: </label>
		<input type="range" name="point_r" id="point_r" bind:value={point.r} on:input={update} min=0 max={200/interval} step=0.01>
		<span>{decimals(point.r, 2)}</span>
	</div>

	<div class="setting">
		<label for="point_theta">Point Theta: </label>
		<input type="range" name="point_theta" id="point_theta" bind:value={point.theta} on:input={update} min=-180 max=180 step=0.01>
		<span>{point.theta >= 0 ? "+" : ""}{pad(decimals(point.theta, 2), 6)}°</span>
	</div>
</div>

<style>
	* {
		text-align: center;
		display: block;
		margin: auto;
		font-family: 'Courier New', Courier, monospace;
		font-variant-numeric: tabular-nums
	}
	
	.content {
		margin: 15px;
		display: flex;
		flex-direction: column;
		gap: 12px;
	}

	.setting {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.setting > * {
		flex-grow: 0;
		margin: 8px
	}
</style>
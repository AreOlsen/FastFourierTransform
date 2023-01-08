<script>
	/*
		FFT FORMULAS.
			Recommended reads:
			https://www.iba.edu.pk/IBALibraries/ORC/onlinebooks/int_2005_emer_tech/Files/41.pdf
			https://people.scs.carleton.ca/~maheshwa/courses/5703COMP/16Fall/FFT_Report.pdf
			https://www.ee.iitm.ac.in/~csr/teaching/pg_dsp/lecnotes/fft.pdf
	*/

	function FFT(x, y) {
		const N = x.length;
		if (N == 1) return [[1, x[0], y[0]]];

		const X_even = [],
			X_odd = [],
			Y_even = [],
			Y_odd = [];
		for (let i = 0; i < N; i++) {
			if (i % 2 == 0) {
				X_even.push(x[i]);
				Y_even.push(y[i]);
			} else {
				X_odd.push(x[i]);
				Y_odd.push(y[i]);
			}
		}

		const X_even_FFT = FFT(X_even, Y_even);
		const X_odd_FFT = FFT(X_odd, Y_odd);

		const X_FFT = [];
		for (let i = 0; i < N / 2; i++) {
			const omega_n = (-2 * Math.PI * i) / N;
			const x_even = X_even_FFT[i][1],
				y_even = X_even_FFT[i][2];
			const x_odd = X_odd_FFT[i][1],
				y_odd = X_odd_FFT[i][2];

			const x_re = x_even + Math.cos(omega_n) * x_odd - Math.sin(omega_n) * y_odd;
			const x_im = y_even + Math.sin(omega_n) * x_odd + Math.cos(omega_n) * y_odd;
			X_FFT.push([i, x_re, x_im]);
			X_FFT.push([N - i, x_re, -x_im]);
		}

		return X_FFT;
	}
	/*
		DEFAULT CHART SETTINGS.
	*/
	import Chart from './LineChart.svelte';
	let dataCsv;
	let csvArray = [
		[-1, 1],
		[1, -1],
		[-1, 1],
		[1, -1]
	];
	let csvDelimiter = ',';
	let csvDecimalSymbol = '.';
	let csvCommentSymbol = '#';

	let csvReader = () => {
		let fr = new FileReader();
		fr.readAsText(dataCsv.files[0]);
		fr.onload = () => {
			let values = [];
			let lines = fr.result.split('\n');
			for (let line of lines) {
				if (line.trim() === '' || line.startsWith(csvCommentSymbol)) {
					continue;
				}
				let lineValues = [];
				line.split(csvDelimiter).map((value) => {
					let trimmed = value.trim().replace(csvDecimalSymbol, '.');
					if (!isNaN(parseFloat(trimmed)) && isFinite(trimmed)) {
						lineValues.push(Number(trimmed));
					}
				});
				if (lineValues.length === 1) {
					lineValues[1] = 0;
				}
				values.push(lineValues);
			}
			csvArray = values;
		};
	};

	$: transformedCsvArray = FFT(
		csvArray.map((val) => val[0]),
		csvArray.map((val) => val[1])
	);

	/* These two are only used in the fourier graph.*/
	$: transformedCsvArrayPhase = [];
	$: transformedCsvArrayAmplitude = [];

	/*
		transformedCsvArray.map((val) => {
			transformedCsvArrayPhase.push(val[2]);
			transformedCsvArrayAmplitude.push(val[1]);
		});
	*/
	/*	
		GENERATE AN OUTPUT CSV FILE.
	*/

	let csvWriter = (csvData, csvDelimit, csvDecimal) => {
		let csvString = '';
		csvData.map((i) => {
			if (i[1] !== 0) {
				i.map((j) => {
					csvString += `${j.toString().replace('.', csvDecimal)}${csvDelimit}`;
				});
				csvString += '\n';
			}
		});
		return csvString;
	};

	let downloadButton;
	let filename = 'fft-results.csv';
	let csvDownloader = () => {
		let text = `${csvCommentSymbol}Frequency${csvDelimiter} Phaseshift${csvDelimiter} Amplitude${csvDelimiter} \n`;
		/*text+=csvWriter(transformedCsvArray, csvDelimiter, csvDecimalSymbol);*/
		let blob = new Blob([text], { type: 'text/plain' });
		downloadButton.download = filename;
		downloadButton.href = window.URL.createObjectURL(blob);
	};
</script>

<input type="checkbox" id="my-modal" class="modal-toggle" />
<div class="modal">
	<div class="modal-box">
		<h3 class="font-bold text-lg">The .CSV File has to be formatted!</h3>
		<p class="py-4">
			The way that the website reads in the values requires that the file is formatted correctly.
			Please follow the following format if only using real values.
		</p>
		<div class="mockup-code flex flex-col items-start">
			<pre data-prefix="1"><code>{csvCommentSymbol}Real Values.</code></pre>
			<pre data-prefix="2"><code>1{csvDecimalSymbol}1{csvDelimiter}</code></pre>
			<pre data-prefix="3"><code>2{csvDecimalSymbol}1{csvDelimiter}</code></pre>
			<pre data-prefix="4"><code>3{csvDecimalSymbol}75{csvDelimiter}</code></pre>
			<pre data-prefix="5"><code>4{csvDecimalSymbol}5{csvDelimiter}</code></pre>
		</div>
		<p class="py-4">
			One can change the delimiter, decimal and the comment symbol. There can either be one column
			of real values, or two columns where the first represents the real values, and the second
			imaginary. Format it on such occasions the following method:
		</p>
		<div class="mockup-code flex flex-col items-start">
			<pre data-prefix="1"><code>{csvCommentSymbol}Real Values{csvDelimiter} Imaginary Values</code
				></pre>
			<pre data-prefix="2"><code>1{csvDecimalSymbol}1{csvDelimiter} 2{csvDelimiter}</code></pre>
			<pre data-prefix="3"><code>2{csvDecimalSymbol}1{csvDelimiter} 4{csvDelimiter}</code></pre>
			<pre data-prefix="4"><code
					>3{csvDecimalSymbol}75{csvDelimiter} 4{csvDecimalSymbol}2{csvDelimiter}</code
				></pre>
			<pre data-prefix="5"><code>4{csvDecimalSymbol}5{csvDelimiter} 5{csvDelimiter}</code></pre>
		</div>
		<p class="py-4">
			One should take note that when one inputs with imaginary values and there are missing values
			from the imaginary values column, it will assume that the missing values are zero in value.
		</p>
		<div class="modal-action">
			<label for="my-modal" class="btn">Okay!</label>
		</div>
	</div>
</div>

<main class="font-mono flex flex-col justify-center items-center gap-y-9 m-24">
	<h1 class="text-5xl">Fourier Transformer!</h1>
	<h2 class="text-xl">Use this simple tool to do a fourier transform!</h2>

	<div class="flex flex-col justify-center gap">
		<span>Inputted graph:</span>
		<Chart
			points={csvArray.map((val) => val[0])}
			yAxisLabel="y"
			xAxisLabel="x"
			points2={csvArray.map((val) => val[1])}
		/>
	</div>
	<div class="flex flex-col items-center gap-y-4">
		<label for="dataInput" class="label">Upload the data (.csv)</label>
		<input
			style="color:white"
			class="file-input file-input-bordered file-input-accent bg-neutral"
			type="file"
			bind:this={dataCsv}
			on:change={csvReader}
			id="dataInput"
			name="dataInput"
			accept=".csv"
		/>
		<div class="flex gap-2">
			<div class="flex flex-col">
				<label for="csvDecimal">CSV Decimal</label>
				<input
					style="color:white"
					id="csvDecimal"
					name="csvDecimal"
					type="text"
					class="input input-accent bg-neutral"
					bind:value={csvDecimalSymbol}
					maxlength="1"
				/>
			</div>
			<div class="flex flex-col">
				<label for="csvDelimiter">CSV Delimiter</label>
				<input
					style="color:white"
					id="csvDelimiter"
					name="csvDelmiter"
					type="text"
					class="input input-accent bg-neutral"
					bind:value={csvDelimiter}
					maxlength="1"
				/>
			</div>
			<div class="flex flex-col">
				<label for="csvComment">CSV Delimiter</label>
				<input
					style="color:white"
					id="csvComment"
					name="csvComment"
					type="text"
					class="input input-accent bg-neutral"
					bind:value={csvCommentSymbol}
					maxlength="1"
				/>
			</div>
		</div>
	</div>
	<div class="flex flex-col justify-center gap">
		<span>Frequency graph:</span>
		<Chart
			points={transformedCsvArrayAmplitude}
			points2={transformedCsvArrayPhase}
			yAxisLabel="Amplitude & Phase Shift"
			xAxisLabel="Frequency"
		/>
	</div>
	<a class="btn" bind:this={downloadButton} on:keydown={csvDownloader}
		>Download .CSV Fourier Values</a
	>
</main>
<button
	class="btn absolute top-[5%] right-[5%]"
	data-toggle-theme="dark,light"
	data-act-class="ACTIVECLASS">🌞Dark🌑</button
>

<label for="my-modal" class="absolute top-[15%] right-[5%] btn">
	<span>Info</span>
</label>

<script>
	import { create, all, complex } from 'mathjs';

	/*
		Create a Math.JS instance with configuration.
	*/
	const config = {
		epsilon: 1e-12,
		matrix: 'Matrix',
		number: 'number',
		precision: 64,
		predictable: false,
		randomSeed: null
	};
	const math = create(all, config);

	/*	
		Array To Complex.
		TODO:
		Optimize the code to get rid of all standard number arrays.
	*/

	function arrayToComplex(array) {
		let complexArray = [];
		for (let i = 0; i < array.length; i++) {
			let comp = math.complex(array[i][0], array[i][1]);
			complexArray.push(comp);
		}
		return complexArray;
	}

	function isPowerOfTwo(n) {
		return n > 0 && (n & (n - 1)) == 0;
	}

	/*	
	FFT.
		Recommended reads:
			https://www.iba.edu.pk/IBALibraries/ORC/onlinebooks/int_2005_emer_tech/Files/41.pdf
			https://people.scs.carleton.ca/~maheshwa/courses/5703COMP/16Fall/FFT_Report.pdf
			https://www.ee.iitm.ac.in/~csr/teaching/pg_dsp/lecnotes/fft.pdf
	*/
	function FFT(complexsignal) {
		let Y = [],
			N = complexsignal.length;
		if (N === 1) {
			return complexsignal;
		}

		let WN = math.pow(math.e, math.divide(math.multiply(2 * math.pi, math.i), N)),
			W = 1;
		let AEVEN = [],
			AODD = [];
		for (let i = 0; i < N; i++) {
			if (i % 2 === 0) {
				AEVEN.push(complexsignal[i]);
				continue;
			}
			AODD.push(complexsignal[i]);
		}
		let YEVEN = FFT(AEVEN),
			YODD = FFT(AODD);
		for (let j = 0; j < N / 2; j++) {
			Y[j] = math.add(YEVEN[j], math.multiply(W, YODD[j]));
			Y[j + N / 2] = math.subtract(YEVEN[j], math.multiply(W, YODD[j]));
			W = math.multiply(W, WN);
		}
		return Y;
	}
	/*
		DEFAULT CHART SETTINGS.
	*/
	import Chart from './LineChart.svelte';
	let dataCsv;

	/*
		csvArray[x][0] is real, csvArray[x][1] is imaginary.
	*/
	let csvArray = [
		[-1, 1],
		[1, -1],
		[-1, 1],
		[1, -1]
	];

	let csvDelimiter = ',';
	let csvDecimalSymbol = '.';
	let csvCommentSymbol = '#';

	let showError = false;
	/*
		READ IN CSV FILES OF DATA.
	*/

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
			showError = false;
			if (!isPowerOfTwo(values.length)) {
				showError = true;
				let amountZeroPad = math.pow(2, math.ceil(math.log(values.length, 2))) - values.length;
				for (let i = 0; i < amountZeroPad; i++) {
					values.push([0, 0]);
				}
			}
			csvArray = values;
		};
	};

	$: transformedCsvArray = FFT(arrayToComplex(csvArray));

	/*	
		GENERATE AN OUTPUT CSV FILE.
	*/

	let csvWriter = (csvData, csvDelimit, csvDecimal) => {
		let csvString = '';
		for (let i = 0; i < csvData.length; i++) {
			csvString += `${i}${csvDelimiter} ${csvData[i].re
				.toString()
				.replace('.', csvDecimal)}${csvDelimit} ${csvData[i].im
				.toString()
				.replace('.', csvDecimal)}\n`;
		}
		return csvString;
	};

	/*
		${csvDelimit} ${math.sqrt(
					math.pow(csvData[i].re, 2) + math.pow(csvData[i].im, 2)
				)}${csvDelimit} ${math.atan(math.divide(csvData[i].im, csvData[i].re))}
		${csvDelimiter} Magnitude${csvDelimiter} PhaseShift
	*/

	let downloadButton;
	let filename = 'fft-results.csv';
	let csvDownloader = () => {
		let text = `${csvCommentSymbol}Frequency${csvDelimiter} Real${csvDelimiter} Complex\n`;
		text += csvWriter(transformedCsvArray, csvDelimiter, csvDecimalSymbol);
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
			<pre data-prefix="1"><code>{csvCommentSymbol}Real Values</code></pre>
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
			Likewise, due to the nature of the Cooley Turkey FFT algorithm, if the input length is not a
			power of 2, the output will be zero-padded untill it is. This may create outputs which are not
			inherently incorrect, but one may have to be careful with the interpretation and selection of
			the basis sinusoidal functions.
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
				<label for="csvComment">CSV Comment</label>
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
			points={transformedCsvArray.map((val) => val.re)}
			points2={transformedCsvArray.map((val) => val.im)}
			yAxisLabel="Real And Imaginary Values"
			xAxisLabel="x"
		/>
	</div>
	<a class="btn" bind:this={downloadButton} on:click={csvDownloader}>Download .CSV Fourier Values</a
	>
	{#if showError}
		<div class="alert alert-warning shadow-lg">
			<div>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="stroke-current flex-shrink-0 h-6 w-6"
					fill="none"
					viewBox="0 0 24 24"
					><path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z"
					/></svg
				>
				<span>Warning: Input has been zero-padded due to not being 2^N in length!</span>
			</div>
		</div>
	{/if}
	<a href="https://github.com/AreOlsen" class="flex flex-row gap-5"
		><span>Created By Are Olsen - 10.02.2023 - V.0.1</span><img
			src="github.png"
			style="height:25px"
			alt="Github logo"
		/></a
	>
</main>
<button
	class="btn absolute top-[5%] right-[5%]"
	data-toggle-theme="dark,light"
	data-act-class="ACTIVECLASS">Dark</button
>
<label for="my-modal" class="absolute top-[15%] right-[5%] btn">
	<span>Info</span>
</label>

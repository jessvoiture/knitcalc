<script>
	import { fade } from 'svelte/transition';
	import {
		gaugeSts,
		gaugeRows,
		gaugeWidth,
		gaugeHeight,
		neckToArmpit,
		garmentWidth,
		neckCircumference,
		underarmCO,
		raglanCount
	} from '../stores';

	let stsIncrPerRound = 8;
	let selectedUnit = 'in';

	let unitAdjustmentFactor = 1;

	$: stsPerUnit = $gaugeSts / ($gaugeWidth * unitAdjustmentFactor); // sts/unit
	$: unitPerSts = ($gaugeWidth * unitAdjustmentFactor) / $gaugeSts; // unit/sts

	$: rowsPerUnit = $gaugeRows / ($gaugeHeight * unitAdjustmentFactor); // rows/unit

	$: raglanWidth = $raglanCount * unitPerSts; // sts / unit
	$: stitchesToCORaw = ($neckCircumference * unitAdjustmentFactor - raglanWidth) * stsPerUnit;
	$: stitchesToCO = roundToNearestDivisibleByNum(stitchesToCORaw, 6) + $raglanCount; // in * sts/in

	$: sleeveSts = (stitchesToCO - $raglanCount) / 6;
	$: frontSts = (stitchesToCO - $raglanCount) / 3;

	$: underarmStsWidth = $underarmCO * 2 * unitPerSts; // width (in) for sts CO underarm

	$: totalRowsNeeded = $neckToArmpit * unitAdjustmentFactor * rowsPerUnit; // in * rows/in = rows
	$: totalStsNeeded = ($garmentWidth * unitAdjustmentFactor * 2 - underarmStsWidth) * stsPerUnit; // in * sts/in = sts

	$: numStsToIncreaseRaw = totalStsNeeded - stitchesToCO; // sts
	$: numStsToIncrease = roundToNearestDivisibleByNum(numStsToIncreaseRaw, stsIncrPerRound);
	$: numIncrRoundNeeded = numStsToIncrease / stsIncrPerRound; // need to repeat increase round x many times
	$: IncrRoundFreq = Math.round(totalRowsNeeded / numIncrRoundNeeded);

	$: inputsEntered =
		$gaugeSts !== undefined &&
		$gaugeSts !== null &&
		$gaugeRows !== undefined &&
		$gaugeRows !== null &&
		$gaugeWidth !== undefined &&
		$gaugeWidth !== null &&
		$gaugeHeight !== undefined &&
		$gaugeHeight !== null &&
		$neckToArmpit !== undefined &&
		$neckToArmpit !== null &&
		$garmentWidth !== undefined &&
		$garmentWidth !== null &&
		$neckCircumference !== undefined &&
		$neckCircumference !== null &&
		$underarmCO !== undefined &&
		$underarmCO !== null &&
		$raglanCount !== undefined &&
		$raglanCount !== null;

	function roundToNearestDivisibleByNum(num, diviser) {
		return Math.round(num / diviser) * diviser;
	}

	function changeUnit(selection) {
		if (selection === 'inches') {
			unitAdjustmentFactor = 1;
		} else if (selection === 'cm') {
			unitAdjustmentFactor = 2.54;
		}

		selectedUnit = selection;
	}

	$: console.log(selectedUnit);
</script>

<div class="wrapper">
	<h1>Knitting Calculator!</h1>

	<!-- <div class="buttons">
		<button
			class="unit-option"
			class:active={selectedUnit == 'in'}
			on:click={() => changeUnit('in')}>inches</button
		>

		<button
			class="unit-option"
			class:active={selectedUnit == 'cm'}
			on:click={() => changeUnit('cm')}>cm</button
		>
	</div> -->

	<div class="container">
		<div class="card" id="inputs">
			<div class="section">
				<h3>Gauge</h3>

				<div style="display: flex; align-items: center;">
					<input type="number" bind:value={$gaugeSts} />
					<p>sts per</p>
					<input type="number" bind:value={$gaugeWidth} />
					<p>{selectedUnit}</p>
				</div>

				<div style="display: flex; align-items: center;">
					<input type="number" bind:value={$gaugeRows} />
					<p>rows per</p>
					<input type="number" bind:value={$gaugeHeight} />
					<p>{selectedUnit}</p>
				</div>
			</div>

			<div class="section">
				<h3>Garment Details</h3>
				<div style="display: flex; align-items: center;">
					<p>Desired width from armpit to armpit:</p>
					<input type="number" bind:value={$garmentWidth} />
					<p>{selectedUnit}</p>
				</div>

				<div style="display: flex; align-items: center;">
					<p>Desired length from the back of the neck to armpit:</p>
					<input type="number" bind:value={$neckToArmpit} />
					<p>{selectedUnit}</p>
				</div>

				<div style="display: flex; align-items: center;">
					<p>Desired neckband circumference:</p>
					<input type="number" bind:value={$neckCircumference} />
					<p>{selectedUnit}</p>
				</div>

				<div style="display: flex; align-items: center;">
					<p># of stitches cast on for underarm (one side):</p>
					<input type="number" bind:value={$underarmCO} />
					<p>sts</p>
				</div>

				<div style="display: flex; align-items: center;">
					<p># of raglan stitches (total):</p>
					<input type="number" bind:value={$raglanCount} />
					<p>sts</p>
				</div>
			</div>
		</div>

		{#if inputsEntered}
			<div class="card" id="results">
				<div class="section" transition:fade>
					<h3>Instructions</h3>
					<p>CO {stitchesToCO} sts</p>
					<p>Back / Front sts: {frontSts} sts</p>
					<p>Sleeve sts (each): {sleeveSts} sts</p>
					<p>Raglan sts: {$raglanCount} sts</p>

					<p>You need to increase to {totalStsNeeded} sts when you split for the body/sleeves.</p>
					<p>
						This means you will need {numIncrRoundNeeded} increase rounds, each with 8 sts added
					</p>

					<p>You need to work {totalRowsNeeded} rows to reach the desired length.</p>
					<p>Therefore, increase every {IncrRoundFreq} rows</p>
				</div>
			</div>
		{/if}
	</div>
</div>

<style lang="scss">
	.container {
		display: flex;
		flex-wrap: wrap;
		gap: 32px;
	}

	.card {
		border-radius: 16px;
		background-color: rgb(239, 244, 246);
		padding: 16px;
		width: 500px;
	}

	.wrapper {
		padding: 8px 32px 8px 32px;
	}

	.unit-option {
		border-radius: 4px;
		color: 'black';
		background-color: rgb(239, 244, 246);
		padding: 4px 12px;
		border: 0px;
		border-color: 'white';
		cursor: pointer;
	}

	.unit-option.active,
	.unit-option:hover {
		background-color: rgb(188, 226, 241);
		border: 0px;
	}

	.buttons {
		margin: 8px 0 8px 0px;
	}
</style>

<!-- src/routes/+page.svelte -->
<!-- This is the same UI component from the previous response. -->
<!-- It provides the mobile-first chart and signal display. -->

<script lang="ts">
	import { onMount } from 'svelte';
	import { createChart, type IChartApi, type ISeriesApi, type CandlestickData } from 'lightweight-charts';

	let chartContainer: HTMLElement;
	let chart: IChartApi;
	let candlestickSeries: ISeriesApi<'Candlestick'>;

	let currentSymbol = 'XAUUSD';
	let currentTimeframe = 'H1';
	let symbols = ['XAUUSD', 'EURUSD', 'GBPUSD', 'USDJPY'];
	let timeframes = ['M5', 'M15', 'H1', 'H4', 'D1'];
	let isLoading = true;
	let errorMessage = '';

	const dummySignals = [
		{ direction: 'Buy', entry: 1955.50, sl: 1945.20, tp1: 1965.80, symbol: 'XAUUSD', timeframe: 'H1', confidence: 85, rr: 1 },
        { direction: 'Sell', entry: 1.0850, sl: 1.0890, tp1: 1.0810, symbol: 'EURUSD', timeframe: 'H1', confidence: 78, rr: 1 }
	];

	onMount(() => {
		chart = createChart(chartContainer, {
			width: chartContainer.clientWidth,
			height: 300,
			layout: { background: { color: '#000000' }, textColor: '#d1d5db' },
			grid: { vertLines: { color: '#2a2a2a' }, horzLines: { color: '#2a2a2a' } },
            crosshair: { mode: 1 },
            timeScale: { borderColor: '#4b5563' },
            rightPriceScale: { borderColor: '#4b5563' },
		});

		candlestickSeries = chart.addCandlestickSeries({
			upColor: '#22c55e', downColor: '#ef4444',
			borderDownColor: '#ef4444', borderUpColor: '#22c55e',
			wickDownColor: '#ef4444', wickUpColor: '#22c55e',
		});

		loadChartData();

        const handleResize = () => chart.resize(chartContainer.clientWidth, 300);
        window.addEventListener('resize', handleResize);
        return () => window.removeEventListener('resize', handleResize);
	});

	async function loadChartData() {
        isLoading = true;
        errorMessage = '';
        try {
            const data = generateMockCandles();
            candlestickSeries.setData(data);
            chart.timeScale().fitContent();
        } catch (error) {
            errorMessage = 'Failed to load chart data.';
        } finally {
            isLoading = false;
        }
    }

    function changeSymbol(newSymbol: string) {
        currentSymbol = newSymbol;
        loadChartData();
    }

    function changeTimeframe(newTimeframe: string) {
        currentTimeframe = newTimeframe;
        loadChartData();
    }

    function generateMockCandles(): CandlestickData[] {
        let price = 1950;
        const data: CandlestickData[] = [];
        let time = Math.floor(Date.now() / 1000) - 200 * 3600;

        for (let i = 0; i < 200; i++) {
            const open = price;
            const close = open + (Math.random() - 0.5) * 10;
            const high = Math.max(open, close) + Math.random() * 5;
            const low = Math.min(open, close) - Math.random() * 5;
            data.push({ time: time as any, open, high, low, close });
            price = close;
            time += 3600;
        }
        return data;
    }
</script>

<svelte:head>
    <title>CRT Signal Platform</title>
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
</svelte:head>

<main class="bg-black text-gray-200 min-h-screen font-sans p-2 md:p-4">
	<div class="max-w-5xl mx-auto">
		<header class="flex justify-between items-center mb-4">
			<h1 class="text-xl font-bold text-yellow-400">CRT Signal Platform</h1>
		</header>

		<div class="fixed bottom-0 left-0 right-0 bg-gray-900 p-2 border-t border-gray-700 flex justify-around z-10 md:relative md:bg-transparent md:border-none md:justify-start md:space-x-2 md:mb-4">
			<div class="flex-1 md:flex-none">
				<select bind:value={currentSymbol} on:change={() => changeSymbol(currentSymbol)} class="w-full bg-gray-800 text-white rounded p-2 border border-gray-600 focus:outline-none focus:ring-2 focus:ring-yellow-400">
					{#each symbols as symbol}
						<option value={symbol}>{symbol}</option>
					{/each}
				</select>
			</div>
			<div class="flex-1 md:flex-none">
				<select bind:value={currentTimeframe} on:change={() => changeTimeframe(currentTimeframe)} class="w-full bg-gray-800 text-white rounded p-2 border border-gray-600 focus:outline-none focus:ring-2 focus:ring-yellow-400">
					{#each timeframes as timeframe}
						<option value={timeframe}>{timeframe}</option>
					{/each}
				</select>
			</div>
		</div>
		
		<div class="mb-4 bg-gray-900 rounded-lg p-1">
			<div bind:this={chartContainer} class="w-full h-[300px] relative">
                {#if isLoading}
                    <div class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-50"><p>Loading Chart...</p></div>
                {/if}
            </div>
		</div>

		<div class="bg-yellow-900 bg-opacity-30 text-yellow-300 text-xs p-2 rounded-md mb-4">
			<strong>Risk Warning:</strong> Not financial advice. Educational purposes only.
		</div>

		<div class="space-y-3 pb-16 md:pb-4">
			<h2 class="text-lg font-semibold">Active Signals</h2>
			{#each dummySignals.filter(s => s.symbol === currentSymbol) as signal}
				<div class="bg-gray-900 rounded-lg p-3 border-l-4 {signal.direction === 'Buy' ? 'border-green-500' : 'border-red-500'}">
                    <!-- Signal details here -->
				</div>
			{/each}
            {#if dummySignals.filter(s => s.symbol === currentSymbol).length === 0}
                <div class="text-center text-gray-500 py-8"><p>No active signals for {currentSymbol}.</p></div>
            {/if}
		</div>
	</div>
</main>

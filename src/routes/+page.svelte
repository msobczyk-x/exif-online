<script lang="ts">
	import exifr from 'exifr';
	type Exif = {
		key: string;
		value: string;
	};

	let exifParsed: Array<Exif> = [];
	let errorMsg: string = '';
	let file: FileList;
	let droppedFile: File;

	function dropHandler(event: DragEvent | any) {
		event.preventDefault();

		if (event.dataTransfer.items) {
			[...event.dataTransfer.items].forEach((item, i) => {
				if (item.kind === 'file' && item.type.startsWith('image/')) {
					droppedFile = item.getAsFile();
				}
			});
		} else {
			[...event.dataTransfer.files].forEach((file, i) => {
				console.log(`… file[${i}].name = ${file.name}`);
			});
		}
	}

	function dragOverHandler(event: DragEvent) {
		event.preventDefault();
	}

	function handleFile(file: File) {
		try {
			const reader = new FileReader();
			reader.readAsArrayBuffer(file);
			reader.onload = async () => {
				const buffer = reader.result as ArrayBuffer;
				exifr.parse(buffer).then(
					(exif: Exif) => {
						exifParsed = Object.entries(exif).map(([key, value]) => ({ key, value }));
					},
					(err) => {
						console.log(err);
					}
				);
			};
		} catch (err) {
			console.log(err);
			errorMsg = 'Error parsing file.';
		}
	}
	$: if (file) {
		handleFile(file[0]);
	}
	$: if (droppedFile) {
		handleFile(droppedFile);
	}
</script>

<main class="flex w-full min-h-screen justify-center items-center bg-slate-100">
	<div
		class="flex justify-center items-center flex-col gap-10 p-12 rounded-xl bg-white shadow-xl w-2/3 my-32 min-h-[35rem]"
	>
		<h1 class="text-6xl font-bold text-center">Exif info online</h1>
		<label for="images" class="drop-container" on:drop={dropHandler} on:dragover={dragOverHandler}>
			<span class="drop-title">Drop file here</span>
			or
			<input type="file" id="images" accept="image/*" required bind:files={file} />
		</label>
		{#if exifParsed.length > 0}
			<div class="border-t-2 border-blue-600 mt-8 pt-8 w-full">
				<table>
					<thead>
						<tr>
							<th>Key</th>
							<th>Value</th>
						</tr>
					</thead>
					<tbody>
						{#each exifParsed as item}
							<tr>
								<td>{item.key}</td>
								<td>{item.value}</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		{/if}
		{#if errorMsg}
			<p>{errorMsg}</p>
		{/if}
	</div>
</main>

<style lang="postcss">
	.drop-container {
		position: relative;
		display: flex;
		gap: 10px;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		height: 200px;
		padding: 20px;
		border-radius: 10px;
		border: 2px dashed #555;
		color: #444;
		cursor: pointer;
		transition: background 0.2s ease-in-out, border 0.2s ease-in-out;
	}

	.drop-container:hover {
		background: #eee;
		border-color: #111;
	}

	.drop-container:hover .drop-title {
		color: #222;
	}

	.drop-title {
		color: #444;
		font-size: 20px;
		font-weight: bold;
		text-align: center;
		transition: color 0.2s ease-in-out;
	}

	input[type='file'] {
		width: 350px;
		max-width: 100%;
		color: #444;
		padding: 5px;
		background: #fff;
		border-radius: 10px;
		border: 1px solid #555;
	}

	input[type='file']::file-selector-button {
		margin-right: 20px;
		border: none;
		background: #084cdf;
		padding: 10px 20px;
		border-radius: 10px;
		color: #fff;
		cursor: pointer;
		transition: background 0.2s ease-in-out;
	}

	input[type='file']::file-selector-button:hover {
		background: #0d45a5;
	}
</style>

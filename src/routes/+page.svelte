<script lang="ts">
	import { invoke } from '@tauri-apps/api/core';
	import {FileDropzone, ProgressRadial } from '@skeletonlabs/skeleton';
	import {event}from "@tauri-apps/api"

	let originalFile: FileList | undefined;
	let verifyFile: FileList | undefined;
	let result: { similarity: number; advice: string } | null = null;
	let isLoading = false;

	function onChangeHandler(e: Event,files:FileList|undefined): void {
		console.log('file data:', e);
		console.log('files:', files);
	}
  
	async function handleVerification() {
	  if (!originalFile || !verifyFile) return;
	  
	  isLoading = true;
	  try {
		// Convert Files to temporary paths
		const originalPath = await writeTempFile(originalFile[0]);
		const verifyPath = await writeTempFile(verifyFile[0]);
		
		result = await invoke('verify_signatures', { 
		  original: originalPath, 
		  verify: verifyPath 
		});
	  } catch (error) {
		console.error('Verification failed:', error);
	  }
	  isLoading = false;
	}

	async function writeTempFile(file: File): Promise<string> {
	  return invoke('write_temp_file', { file: { name: file.name, data: Array.from(new Uint8Array(await file.arrayBuffer())) }});
	}
  </script>
  
  <div class="card p-8 m-8 max-w-2xl mx-auto">
	<h1 class="text-2xl font-bold mb-6">Signature Verification</h1>
	
	<div class="grid grid-cols-2 gap-4 mb-6">
	  <FileDropzone
	  	name="orig-sign"
		accept="image/png,image/jpeg"
		label="Original Signature"
		bind:files={originalFile}
		multiple={false}
		on:change={(e)=>onChangeHandler(e,originalFile)}
		{isLoading}
	  >
	  <svelte:fragment slot="lead"><i class="justify-center flex p-8 "><svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 24 24"><path fill="currentColor" d="M14 2H6c-1.1 0-1.99.9-1.99 2L4 20c0 1.1.89 2 1.99 2H18c1.1 0 2-.9 2-2V8zm4 18H6V4h7v5h5zM8 15.01l1.41 1.41L11 14.84V19h2v-4.16l1.59 1.59L16 15.01L12.01 11z"/></svg></i></svelte:fragment>
	  <svelte:fragment slot="meta">(.png and .jpg alowed)</svelte:fragment>
		{#if originalFile}
		  <div class="text-primary-500">{originalFile[0].name}</div>
		{:else}
		  <span class="text-surface-500">Drop or select image</span>
		{/if}
	  </FileDropzone>
  
	  <FileDropzone
	  	name="verify-sign"
		accept="image/png,image/jpeg"
		label="Test Signature"
		bind:files={verifyFile}
		multiple={false}
		on:change={(e)=>onChangeHandler(e,verifyFile)}
		{isLoading}
	  >
	  <svelte:fragment slot="lead"><i class="justify-center flex p-8 "><svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 24 24"><path fill="currentColor" d="M14 2H6c-1.1 0-1.99.9-1.99 2L4 20c0 1.1.89 2 1.99 2H18c1.1 0 2-.9 2-2V8zm4 18H6V4h7v5h5zM8 15.01l1.41 1.41L11 14.84V19h2v-4.16l1.59 1.59L16 15.01L12.01 11z"/></svg></i></svelte:fragment>
	  <svelte:fragment slot="meta">(.png and .jpg alowed)</svelte:fragment>
		{#if verifyFile}
		  <div class="text-primary-500">{verifyFile[0].name}</div>
		{:else}
		  <span class="text-surface-500">Drop or select image</span>
		{/if}
	  </FileDropzone>
	</div>
  
	<button
	  on:click={handleVerification}
	  disabled={isLoading|| !originalFile || !verifyFile}
	  class="w-full btn variant-filled"
	>
	  {#if isLoading}
		<ProgressRadial size="sm" class="mr-2" intermediate />
		Verifying...
	  {:else}
		Verify Signatures
	  {/if}
	</button>
  
	{#if result}
	  <div class="mt-6 p-4 bg-surface-50 rounded-lg border border-surface-200">
		<div class="flex gap-4 items-center mb-3">
		  <span class="text-lg font-semibold">Similarity:</span>
		  <span class="text-2xl font-bold text-primary-600">
			<ProgressRadial value={result.similarity * 100}>{result.similarity*100}%</ProgressRadial>
		  </span>
		</div>
		<div class="space-y-2">
		  <span class="font-medium">AI Recommendation:</span>
		  <p class="text-surface-700">{result.advice}</p>
		</div>
	  </div>
	{/if}
	</div>
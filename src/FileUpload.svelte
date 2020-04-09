<script>
  import * as cocoSsd from '@tensorflow-models/coco-ssd';
  import { onMount } from 'svelte';
  import Spinner from './Spinner.svelte';

  const baseModel = 'mobilenet_v1';
  let model;
  let loading = false;

  onMount(async () => {
    loading = true;
    model = await cocoSsd.load({ base: baseModel });
    loading = false;
  });
  let fileInput;

  function convertToBase64(file) {
    return new Promise((resolve) => {
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.addEventListener('loadend', (event) => {
        resolve(event.target.result);
      });
    });
  }

  async function uploadFile(e) {
    loading = true
    const imageFile = fileInput.files[0] || e.dataTransfer.files.item(0);
    // Reset Dropzone
    if (!imageFile) return;
    const b64Image = await convertToBase64(imageFile);
    const detectTest = new Image();
    detectTest.src = b64Image;
    detectTest.width = 500;
    detectTest.height = 375;
    console.log(detectTest);
    const result = await model.detect(detectTest);
    loading = false;
    console.log('number of detections: ', result.length);
  }

</script>
  <input type="file"
    bind:this={fileInput}
    id="image_file" 
    name="image_file"
    on:change={uploadFile}
    accept="image/png, image/jpeg" />

    <label 
      for="image_file" 
      class="input-label"
    >
    Load Image
  </label>
  {#if loading}
  <Spinner />
  {/if}
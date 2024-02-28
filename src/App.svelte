<script type="module" lang="ts">
  import hotLogo from './assets/hot_logo.png';
  import qrcodeGenerator from 'qrcode-generator';
  import { deflate, inflate } from 'pako';
  import { HTMLCanvasElementLuminanceSource } from '@zxing/browser';
  import { HybridBinarizer, QRCodeReader, BinaryBitmap } from '@zxing/library';

  let qrCodeDataURL = '';
  let jsonData = '';
  let decodedJson = '';
  let qrInput;

  const base64zlibencode = (string) => {
    return btoa(String.fromCodePoint(...deflate(new TextEncoder().encode(string))));
  };

  const base64zlibdecode = (string) => {
    return new TextDecoder().decode(inflate(Uint8Array.from(atob(string), c => c.codePointAt(0))));
  };

  const generateQRCode = async () => {
    if (!jsonData) return;
    const code = qrcodeGenerator(0, 'L');
    code.addData(base64zlibencode(jsonData));
    code.make();
    qrCodeDataURL = code.createDataURL(3, 5);
  };

  const downloadQRCode = () => {
    const link = document.createElement('a');
    link.href = qrCodeDataURL;
    link.download = 'qrcode.png';
    link.click();
  };

  const decodeFromQR = async () => {
    const imageData = qrInput.files?.[0];
    if (!imageData) return;

    const reader = new FileReader();
    reader.onload = async (event) => {
      const img = new Image();
      img.onload = async () => {
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        canvas.width = img.width;
        canvas.height = img.height;
        ctx?.drawImage(img, 0, 0, img.width, img.height);

        const luminanceSource = new HTMLCanvasElementLuminanceSource(canvas);
        const binarizer = new HybridBinarizer(luminanceSource);
        const binaryBitmap = new BinaryBitmap(binarizer);
        const barcodeReader = new QRCodeReader();
        try {
          const result = barcodeReader.decode(binaryBitmap);
          const encodedData = result.getText();
          const decodedData = base64zlibdecode(encodedData);
          decodedJson = decodedData;
        } catch (error) {
          console.error('Could not decode QR code:', error);
          decodedJson = 'Error decoding QR code';
        }
      };
      img.src = event.target!.result;
    };
    reader.readAsDataURL(imageData);
  };

</script>

<main>
  <div>
    <a href="https://hotosm.org" target="_blank" rel="noreferrer">
      <img src={hotLogo} class="logo" alt="HOT Logo" />
    </a>
  </div>

  <h2>QR Code Encode and Decode</h2>

  <h3>Encode QR Code</h3>
  <textarea rows="10" cols="50" on:input={e => jsonData = e.target.value}></textarea><br>
  <button on:click={generateQRCode}>Generate QR Code</button>
  {#if qrCodeDataURL !== null}
    <button on:click={downloadQRCode}>Download QR Code</button>
  {/if}
  <br><br>
  {#if qrCodeDataURL !== null}
    <img src={qrCodeDataURL} alt="" />
  {/if}

  <h3>Decode QR Code</h3>
  <input type="file" id="qrInput" accept="image/*" bind:this={qrInput} on:change={decodeFromQR}>
  <br><br>
  {#if decodedJson}
    <div id="decodedJson">{decodedJson}</div>
  {/if}
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
</style>

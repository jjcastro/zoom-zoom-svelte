<script lang="ts">
  import { onMount } from "svelte";

  let filePicker: HTMLInputElement | null = null;
  let selectedImages: { url: string; name: string; file: File }[] = [];
  let presentationRequest: PresentationRequest | null = null;

  let presentationConnection: PresentationConnection | null = null;
  let searchTerm = "";
  let searchInput: HTMLInputElement | null = null;

  let canCast = false;

  function handleFileSelect(event: Event) {
    const input = event.target as HTMLInputElement;
    const files = Array.from(input.files || [])
      .filter((file) => file.type.startsWith("image/"))
      .map((file) => ({
        url: URL.createObjectURL(file),
        name: file.name,
        file: file,
      }));

    selectedImages = [...selectedImages, ...files];
  }

  // Function to handle file upload and send the image
  const sendImage = (file: File): void => {
    console.log("img bton callback");
    console.log(presentationConnection);
    let result = presentationConnection.send("test");
    console.log(result);

    const reader = new FileReader();

    reader.onload = () => {
      const base64Image = reader.result as string; // Base64 encoded image
      const message = { type: "image", data: base64Image }; // Message object
      presentationConnection.send(JSON.stringify(message)); // Send as JSON
    };

    if (file) {
      reader.readAsDataURL(file); // Convert the image to Base64
    }
  };

  function initCast(event) {
    console.log("inniting");
    presentationConnection = event.connection;
    presentationConnection.addEventListener("terminate", () => {
      console.log("bye");
      presentationConnection = null;
    });

    // presentationConnection.send(data);

    presentationConnection.addEventListener("message", ({ data }) => {
      console.log("hi,");
      // peer.signalingPort.postMessage(data);
    });
    presentationConnection.onmessage = (event: MessageEvent): void => {
      console.log("hi222");
      console.log("Message received:", event.data);
      try {
        const message = JSON.parse(event.data); // Parse incoming data as JSON

        if (message.type === "image") {
          const imageElement = document.createElement("img");
          imageElement.src = message.data; // Set Base64 string as image source
          imageElement.style.maxWidth = "100%";
          imageElement.style.height = "auto";
          document.body.appendChild(imageElement); // Add image to the display
        }
      } catch (error) {
        console.error("Error handling message:", error);
      }
    };

    console.log(presentationConnection);
  }

  // async function startPresentation() {
  //   try {
  //     const presentationRequest = new PresentationRequest([
  //       "index.html",
  //       // window.location.href,
  //     ]);
  //     presentationConnection = await presentationRequest.start();

  //     presentationConnection.addEventListener("message", ({ data }) => {
  //       console.log("Message received:", data);
  //     });

  //     // Listen for incoming messages
  //     presentationConnection.onmessage = (event: MessageEvent): void => {
  //       console.log("Message received:", event.data);
  //       try {
  //         const message = JSON.parse(event.data); // Parse incoming data as JSON

  //         if (message.type === "image") {
  //           const imageElement = document.createElement("img");
  //           imageElement.src = message.data; // Set Base64 string as image source
  //           imageElement.style.maxWidth = "100%";
  //           imageElement.style.height = "auto";
  //           document.body.appendChild(imageElement); // Add image to the display
  //         }
  //       } catch (error) {
  //         console.error("Error handling message:", error);
  //       }
  //     };

  //     // presentationConnection.send(JSON.stringify(selectedImages));
  //   } catch (error) {
  //     console.error("Presentation failed:", error);
  //     alert("Could not start presentation.");
  //   }
  // }

  function clearPresentation() {
    if (presentationConnection) {
      presentationConnection.terminate();
      presentationConnection = null;
    }
    selectedImages = [];
    if (filePicker) filePicker.value = "";
    searchTerm = "";
  }

  function handleSearchFocus() {
    searchInput?.focus();
  }

  function handleSearchKeyDown(event: KeyboardEvent) {
    if (event.key === "Enter") {
      searchInput?.focus();
    }
  }

  function toggleCast() {
    // if (video.readyState) {
    if (presentationConnection) {
      presentationConnection?.terminate();
    } else {
      console.log("start");
      presentationRequest.start();
    }
    // }
  }

  let isPresentationSupported = false;
  onMount(() => {
    isPresentationSupported = "PresentationRequest" in window;

    if ("PresentationRequest" in window) {
      const handleAvailability = (aval) => {
        canCast = !!aval;
      };
      console.log(window.location.href);
      presentationRequest = new PresentationRequest(
        "file:///Users/juanjo/Sites/zoom-zoom-svelte/dist/viewer.html",
      );
      console.log("ya");
      presentationRequest.addEventListener("connectionavailable", (e) =>
        initCast(e),
      );
      navigator.presentation.defaultRequest = presentationRequest;

      navigator.presentation.receiver.connectionList.then((list) => {
        list.connections.forEach((connection) => {
          connection.addEventListener("message", (event) => {
            console.log("Received message:", event.data);
          });
        });
      });

      presentationRequest.getAvailability().then((aval) => {
        console.log(aval);
        aval.onchange = (e) => {
          handleAvailability(e.target.value);
          console.log("aval changed " + e.target.value);
        };
        handleAvailability(aval.value);
      });
    }
  });
</script>

<div class="container">
  {#if isPresentationSupported}
    <h1>Multi-Image Presentation App</h1>

    <input
      type="file"
      multiple
      accept="image/*"
      bind:this={filePicker}
      on:change={handleFileSelect}
    />

    {#if selectedImages.length > 0}
      <div
        class="search-container"
        role="button"
        tabindex="0"
        on:click={handleSearchFocus}
        on:keydown={handleSearchKeyDown}
      >
        <input
          type="text"
          placeholder="Search images..."
          bind:value={searchTerm}
          bind:this={searchInput}
          on:blur={() => searchInput?.focus()}
        />
      </div>

      <div class="image-grid">
        {#each selectedImages as image}
          <button
            class="image-container"
            on:click={() => sendImage(image.file)}
            class:highlighted={searchTerm &&
              image.name.toLowerCase().includes(searchTerm.toLowerCase())}
          >
            <img src={image.url} alt={image.name} />
          </button>
        {/each}
      </div>

      <div class="controls">
        <button on:click={toggleCast}> Toggle Present on Second Screen </button>
        <button on:click={clearPresentation}> Clear All </button>
      </div>
    {/if}
  {:else}
    <p>
      Presentation API is not supported in this browser. Please use Google
      Chrome, Microsoft Edge or Opera browsers.
    </p>
  {/if}
</div>

<style>
  .container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    text-align: center;
  }

  .search-container {
    position: relative;
    width: 100%;
    cursor: pointer;
  }

  input[type="text"] {
    width: 100%;
    padding: 10px;
    margin: 10px 0;
    box-sizing: border-box;
  }

  .image-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
    gap: 10px;
    margin-top: 20px;
  }

  .image-container {
    aspect-ratio: 1 / 1;
    overflow: hidden;
  }

  .image-container img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .highlighted {
    outline: 3px solid #4caf50;
  }

  .controls button {
    margin: 0 10px;
    padding: 10px 15px;
    background-color: #4caf50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }

  input[type="file"] {
    margin: 20px 0;
  }
</style>

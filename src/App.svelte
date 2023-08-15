<script>
    import { onMount } from "svelte";
    import { writable } from "svelte/store";
    import axios from "axios";

    const openAIToken = writable(localStorage.getItem("openAIToken") || "");
    const file = writable(null);
    const model = "whisper-1";

    let result;

    async function handleSubmit() {
        const formData = new FormData();
        formData.append("file", $file);
        formData.append("model", model);

        try {
            result = "Uploading...";

            const response = await axios.post("https://api.openai.com/v1/audio/transcriptions", formData, {
                headers: {
                    Authorization: `Bearer ${$openAIToken}`,
                    "Content-Type": "multipart/form-data",
                },
            });

            if (response.status === 200) {
                console.log("Transcription result:", response.data.text);
                result = response.data.text;
            } else {
                console.error("Unexpected response:", response.status, response.statusText);
                result = "Error: Unexpected response from server";
            }
        } catch (error) {
            if (error.response && error.response.status === 400) {
                console.error("Bad request:", error.response.data);
                result = "Error: Bad request - " + error.response.data.error.message;
            } else {
                console.error("Error submitting transcription:", error);
                result = "Error submitting transcription: " + error.message;
            }
        }
    }

    // Save token to local storage when it changes
    openAIToken.subscribe((value) => {
        localStorage.setItem("openAIToken", value);
    });

    onMount(() => {
        // Populate token from local storage on component mount
        openAIToken.set(localStorage.getItem("openAIToken") || "");
    });
</script>

<main>
    <h1 class="header">Whisper UI</h1>
    <form class="form" on:submit|preventDefault={handleSubmit}>
        <div class="form-group">
            <label for="openaiToken">OpenAI Token (only saved on this machine):</label>
            <input id="openaiToken" type="password" class="input" bind:value={$openAIToken} placeholder="Enter your OpenAI token" />
        </div>

        <div class="form-group">
            <label for="fileInput">File Submission:</label>
            <input id="fileInput" type="file" class="input" accept=".m4a,.mp3,.webm,.mp4,.mpga,.wav,.mpeg,.ogg,.oga,.flac" on:change={(event) => ($file = event.target.files[0])} />
        </div>

        <button class="submit-button" type="submit">Submit</button>

        {#if result}
            <p class="result">{result}</p>
        {/if}
    </form>
</main>

<style>
    * {
        font-family: "Maven Pro", sans-serif;
        font-family: "Roboto Mono", monospace;
    }

    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        /* height: 100vh; */
    }

    .header {
        font-size: 2rem;
        margin-bottom: 1rem;
        color: #333;
    }

    .form {
        background-color: #fff;
        border-radius: 8px;
        padding: 2rem;
        box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        width: 100%;
        max-width: 400px;
        text-align: left;
    }

    .form-group {
        margin-bottom: 1rem;
    }

    label {
        font-size: 1rem;
        color: #555;
        margin-bottom: 0.5rem;
        display: block;
    }

    .input {
        width: 100%;
        padding: 0.5rem;
        border: 1px solid #ccc;
        border-radius: 4px;
        font-size: 1rem;
    }

    .submit-button {
        background-color: #007bff;
        color: #fff;
        border: none;
        border-radius: 4px;
        padding: 0.5rem 1rem;
        font-size: 1rem;
        cursor: pointer;
        transition: background-color 0.2s;
    }

    .submit-button:hover {
        background-color: #0056b3;
    }

    .result {
        margin-top: 1rem;
        font-size: 1rem;
        color: #333;
    }
</style>

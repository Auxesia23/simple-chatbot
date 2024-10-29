<script lang="ts">
    import { marked } from 'marked';
    const apiUrl = import.meta.env.VITE_API_URL;

    let query = "";
    let responseData = ""; // Reactive variable for streaming data

    async function fetchData() {
        responseData = ""; // Clear previous response

        try {
            const res = await fetch(apiUrl, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({ query }),
            });

            if (!res.ok) {
                throw new Error('Fetch failed!');
            }

            // If response is in stream form
            const reader = res.body?.getReader();
            const decoder = new TextDecoder("utf-8");

            // Read data from the stream
            while (true) {
                const { done, value } = await reader!.read();
                if (done) break;
                responseData += decoder.decode(value, { stream: true });
            }
        } catch (error) {
            console.error('Error:', error);
            responseData = "An error occurred while fetching data.";
        }
    }
</script>

<h1 class="text-2xl mt-5 font-bold text-center mb-6 dark:text-gray-100">What Can I Help With?</h1>
<div class="flex flex-col items-center justify-center max-h-screen">
    <div class="mt-10 w-full sm:w-1/2 shadow-md rounded-lg p-6 border border-gray-200 dark:border-gray-700 dark:bg-gray-800">
        <form on:submit|preventDefault={fetchData} class="flex flex-col space-y-4">
            <textarea
                bind:value={query}
                class="input textarea textarea-bordered textarea-lg w-full h-32 p-3 rounded-lg border-gray-300 dark:border-gray-600 dark:bg-gray-700 dark:text-gray-100 focus:ring-2 focus:ring-green-500 focus:outline-none resize-none"
                placeholder="Ask me anything..."
            ></textarea>
            <button type="submit" class="btn btn-success w-full py-3 text-lg font-semibold bg-green-500 text-white rounded-lg hover:bg-green-600 focus:ring-4 focus:ring-green-300">
                Send
            </button>
        </form>
    </div>

    <div class="mt-10 w-full sm:w-1/2 shadow-md rounded-lg p-6 border border-gray-200 dark:border-gray-700 dark:bg-gray-800">
        <h2 class="text-xl font-semibold mb-4 dark:text-gray-100">Response</h2>
        <div class="prose w-full h-96 overflow-scroll text-black dark:text-gray-100">
            {@html marked(responseData)}
        </div>
    </div>
</div>

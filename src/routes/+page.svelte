<script lang="ts">
    // import { onMount } from "svelte";

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    let socket: WebSocket;

    const squareSize = 20;

    function setupWebSocket() {
        socket = new WebSocket("ws://localhost:3312/ws");

        socket.addEventListener("open", () => {
            console.log("WebSocket connected");

            socket.send('sub');
        });

        socket.addEventListener("message", (event) => {
            const [x, y] = event.data.split(";").map(Number) as [
                number,
                number,
            ];
            if (!isNaN(x) && !isNaN(y)) {
                drawSquare(x, y);
            }
        });

        socket.addEventListener("close", () => {
            console.log("WebSocket disconnected");
            setTimeout(setupWebSocket, 1000);
        });

        socket.addEventListener("error", (error) => {
            console.error("WebSocket error:", error);
        });

        return socket;
    }

    function drawSquare(x: number, y: number) {
        if (ctx) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = "red";
            ctx.fillRect(x, y, squareSize, squareSize);
        }
    }

    $effect(() => {
        if (canvas) {
            ctx = canvas.getContext("2d")!;
            setupWebSocket();
        }
    });
</script>

<main>
    <canvas
        bind:this={canvas}
        width={500}
        height={300}
        style="border: 1px solid #ccc"
    ></canvas>
</main>

<style>
    main {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
    }
</style>

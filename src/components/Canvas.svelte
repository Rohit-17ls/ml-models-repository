<script>
    import {onMount} from 'svelte'
    export let makePrediction;
    let canvas;
    let wrapper;
    let context;
    let image;
    let isDrawing = false;
    let mouse = {x : 0, y : 0}

    const startPosition = (e) => {
        console.log("startPosition");
        isDrawing = true;
        mouse.x = e.offsetX;
        mouse.y = e.offsetY;
        draw(e);
    }
    
    const finishedPosition = (e) => {
        console.log("finishPosition");
        isDrawing = false;
        context.beginPath();
    }

    const getPositionX = (e) => {
        const rect = canvas.getBoundingClientRect();
        // const x = e.pageX - window.pageXOffset - rect.left;
        const x = e.pageX - canvas.offsetLeft;
        return x;
    }
    
    const getPositionY = (e) => {
        const rect = canvas.getBoundingClientRect();
        // const y = e.pageY - window.pageYOffset - rect.top;
        const y = e.pageY - canvas.offsetTop;
        return y;
    }

    
    
    const draw = (e) => {
        console.log("draw");
        if(!isDrawing) return;
        context.lineWidth = 8;
        context.lineCap = 'round';

        const newX = e.offsetX;
        const newY = e.offsetY;

        context.beginPath();
        context.moveTo(mouse.x, mouse.y);
        context.lineTo(newX, newY);
        context.stroke();
        mouse.x = newX;
        mouse.y = newY;
    }

    const canvasDrawHandler = () => {
        context = canvas.getContext("2d");
        canvas.addEventListener("mousedown", startPosition);
        canvas.addEventListener("mouseup", finishedPosition);
        canvas.addEventListener("mouseout", finishedPosition);
        canvas.addEventListener("mousemove", draw);
    }

    const clearCanvas = () => {
        context.clearRect(0, 0, canvas.width, canvas.height);
    }

    const resizeBase64Image = async (base64, width, height) => {
        // Create a canvas element
        const canvasElement = document.createElement('canvas');

        // Create an image element from the base64 string
        image = new Image();
        image.src = base64;

        // Return a Promise that resolves when the image has loaded
        return new Promise((resolve, reject) => {
            image.onload = () => {
            // Calculate the aspect ratio of the image
            const aspectRatio = image.width / image.height;

            // Calculate the best fit dimensions for the canvasElement
            if (width / height > aspectRatio) {
                canvasElement.width = height * aspectRatio;
                canvasElement.height = height;
            } else {
                canvasElement.width = width;
                canvasElement.height = width / aspectRatio;
            }

            // Draw the image to the canvasElement
            canvasElement.getContext('2d').drawImage(image, 0, 0, canvasElement.width, canvasElement.height);

            // Resolve the Promise with the resized image as a base64 string
            resolve(canvasElement.toDataURL());
            };

            image.onerror = reject;
        });
        };

    const initiatePrediction = async() => {
        const dataURI = canvas.toDataURL();
        const resizedDataURI = await resizeBase64Image(dataURI, 28, 28);
        console.log(resizedDataURI);
        makePrediction(resizedDataURI, image);
    }


    onMount(() => {
        canvas = wrapper.querySelector('#canvas');
        // canvas.width = '200px';
        // canvas.height = '200px';
        console.log(canvas);
        canvasDrawHandler();
    })



</script>

<div id="canvas-encloser" bind:this={wrapper}>
    <canvas id="canvas" width = "300" height = "300">
 
    </canvas>
</div>
<button type="button" on:click = {clearCanvas}>Clear Canvas</button>
<button type="button" on:click = {initiatePrediction}>Make Prediction</button>

<style>

#canvas-encloser{
    /* background: rgba(255, 255, 0, 0.115); */
    border-width: 2px;
    border-image : linear-gradient(to right,#ff009e,#8700ff) 1;
    border-style : solid;
    min-width: 350px;
    min-height: 350px;
    margin: 3em;
    background: white;
}

#canvas{
    /* background: rgba(12, 220, 12, 0.408); */
    /* width: 400px;
    height: 400px; */
}


</style>
<script>
    import {onMount} from 'svelte'
    import * as tf from '@tensorflow/tfjs'
    import Canvas from './Canvas.svelte';
    let imagePicker;
    // let bar;

    let prediction = '';

    const tfmodel = {
        model: null,
        status: false,
        predicting : false,
        predicted : false
    }
    
    const imgObject = {
        src : null,
        status : false,
        error: false,
        errorMessage: "No image selected for classification",
        imageTensor: null,
        imageElement: null,
        resizedImage: null,
        scaledImage: null,
        prediction : null
    }

    const MODEL_ENDPOINT = 'https://bitbucket.org/ml-projects-17/ml-models/raw/f5bbc2e263e13b637d08ada154f8e1189c22ac36/happy-sad-classifier/model.json'
    

    const getImage = () => {
        const image = imagePicker.files[0];
        if(image){
            const fileReader = new FileReader();
            fileReader.readAsDataURL(image);
            fileReader.addEventListener('load', (e) => {
                imgObject.src = fileReader.result
                imgObject.status = true
                imgObject.imageElement = new Image()
                // imgObject.imageElement = document.createElement('canvas');
                // imgObject.imageElement = imgObject.imageElement.drawImage(imgObject.src, 0, 0, 256, 256).getImageData(0, 0, 256, 256);
                imgObject.imageElement.onload = () => {
                    imgObject.imageElement.style.display = 'none'
                    document.body.append(imgObject.imageElement);
                    imgObject.error = false;
                    
                }
                imgObject.imageElement.src = fileReader.result
                // console.log(imgObject.src)
            })
        }
    }

    const classify = async() => {
        if(imgObject.imageElement === null){
            imgObject.error = true;
            return;
        }
        console.log(imgObject.imageElement);
        imgObject.imageTensor = tf.browser.fromPixels(imgObject.imageElement);
        console.log(imgObject.imageTensor.dataSync());
        imgObject.resizedImage = tf.image.resizeBilinear(imgObject.imageTensor, [256, 256]);
        console.log(imgObject.resizedImage.shape)
        imgObject.scaledImage = imgObject.resizedImage.div(tf.scalar(255)).reshape([1, 256, 256, 3]);
        // imgObject.scaledImage =  tf.tensor4d(Array.from(imgObject.resizedImage.dataSync()),[1,256,256,3])
        console.log(imgObject.scaledImage);
        console.log(imgObject.scaledImage.dataSync());

        tfmodel.predicting = true;
        imgObject.prediction = tfmodel.model.predict(imgObject.scaledImage, true).dataSync()[0];
        console.log(imgObject.prediction);
        prediction = imgObject.prediction < 0.5 ? "happy" : "sad";
        console.log(imgObject.prediction, prediction);
        tfmodel.predicting = false;
        tfmodel.predicted = true;

        
    }


    onMount(async() => {
        console.log("Hello")
        tfmodel.model = await tf.loadLayersModel(MODEL_ENDPOINT);
        tfmodel.status = true;
        console.log(typeof tfmodel.model);
    })

</script>

<h1>Happy-or-Sad Classifier</h1>
<div class="margin">
    {#if tfmodel.status}
        <strong><label for="image">Pick an image to classify : </label></strong>
        <input bind:this={imagePicker} class="margin" type="file" id="file" on:change={getImage}>
        <img src={imgObject.src} class="classification-img {imgObject.status ? 'block' : 'none'}" alt="classification-img"/>
        <strong class="block error">{imgObject.error ? imgObject.errorMessage : ''}</strong>
        <button type="button" class="block" on:click={classify}>Classify</button>
        {#if tfmodel.predicting || tfmodel.predicted}
            <h3 class="margin"> Prediction : {prediction !== '' ? prediction : "Predicting..."}</h3>

            {#if prediction !== ''}
                <h3>Happy-Scale</h3>
                <section id="bar-wrap">
                    <div style="width : {(1-imgObject.prediction)*100}%" id="bar"></div>
                </section>
                <strong class="special-text">Estimated Happiness Level : {parseFloat(((1-imgObject.prediction)*100).toString()).toFixed(2)}%</strong>
            {/if}
        {/if}
    {:else}
        <h2>Loading the model ...</h2>
    {/if}
    
</div>

<style>
    .none{
        display: none;
    }

    .block{
        display: block;
        margin: auto
    }

    .classification-img{
        margin: 2em
    }

    .margin{
        margin: 2em;
    }

    .error{
        color : red
    }

    #bar-wrap{
        width: 500px;
        height: 20px;

        background: linear-gradient(#171a21, #171a21) padding-box,
            linear-gradient(to right,#ff009e,#8700ff) border-box;
        border-radius: 10px;
        border: 2px solid transparent;

        margin: 3em auto;
        
    }
    
    #bar{
        border-radius: 8px;
        background: linear-gradient(to right,#ff009e,#8700ff);
        height: 100%;
        transition: all 0.5s ease-in-out;
    }


</style>
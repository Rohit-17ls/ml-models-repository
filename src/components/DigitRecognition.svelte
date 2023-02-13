<script>
    import {onMount} from 'svelte'
    import * as tf from '@tensorflow/tfjs'
    // import * as nj from 'numjs'
    import Canvas from './Canvas.svelte';
  
    let modelState = "";
    let predictState = false;
    let prediction = 0;
    let model;

    const MODEL_ENDPOINT = 'https://bitbucket.org/ml-projects-17/ml-models/raw/f5bbc2e263e13b637d08ada154f8e1189c22ac36/digit-recognizer/model.json';
    
    const loadModel = async() => {
      model = await tf.loadGraphModel(MODEL_ENDPOINT)
      console.log("model loaded");
    }

    onMount(() => {
        loadModel();
    })

    const makePrediction = async(base64img, img) => {
        
        // const image = tf.browser.fromPixels(img);
        // const res = model.predict(img);

        
        let tensor = tf.browser.fromPixels(img)
                       .resizeNearestNeighbor([28, 28])
                       .toFloat()
                       .expandDims(0)
                       .reshape([-1, 28, 28, 1])
        
        const layerNormalization = tf.layers.layerNormalization();
        layerNormalization.apply(tensor)


        console.log(tensor);

        const prediction = await model.predict(tensor).data();
        console.log(prediction)
        console.log(tf.argMax(prediction))
    }


    
  </script>
  
  <h1 class="error">Warning : This page doesn't work as of now</h1>
  <div>
    <h1>Enter a number into the canvas</h1>
    <Canvas {makePrediction}/>
    {#if predictState}
        <div><strong>Prediction :</strong>{prediction}</div>
    {/if}
  </div>
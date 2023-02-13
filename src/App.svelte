<script>
    import {Router, Route} from 'svelte-navigator'
    import DigitRecognition from './components/DigitRecognition.svelte';
    import HappySadClassifier from './components/HappySadClassifier.svelte'
    import Home from './components/Home.svelte';
    import Test from './components/Test.svelte';


</script>

<main>
  <Router>
    <Route path="/"><Home/></Route>
    <Route path="/digit-recognizer"><DigitRecognition/></Route>
    <Route path="/happy-sad-classifier"><HappySadClassifier/></Route>
    <Route path="/test"><Test/></Route>
  </Router>
</main>

<style>
 
</style>

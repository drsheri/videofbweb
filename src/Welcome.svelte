<script>
  import { navigate } from "svelte-routing";

  export let location;

  function onYes() {
      navigate("/video");
  }

</script>

<style>
  .dialogue{
    /* margin: 25% auto;
    padding: 20px;
    right: 20px; */
  /* position: relative; */
  
    width: 100%;
    height: auto;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

    /* background: rgb(95, 95, 95);
    font-family: Arial, Helvetica, sans-serif;
    font-weight: 100;
    font-size: 25px; */
    
  }

  .dialogue_text_box{
    background: rgb(95, 95, 95);
    padding: 30px;
    margin-bottom: 10px;
  }

  .dialogue_text_box>p{
    font-family: Arial, Helvetica, sans-serif;
    font-weight: 100;
    font-size: 25px;
    
  }

  .xxdialogue>p{
    /* margin: 25% auto;
    padding: 20px;
    right: 20px; */
  /* position: relative; */
  
    /* width: 100%;
    height: auto;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%); */

    /* background: rgb(95, 95, 95);
    font-family: Arial, Helvetica, sans-serif;
    font-weight: 100;
    font-size: 25px; */
    
  }


  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  .button {
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    font-size: 20px;
    background: #434343;
    padding: 10px;
	  width: 100px;
    border-radius: 5px;
    display: inline-block;
    border: none;
    transition: all 0.4s ease 0s;
  }
  .button:hover {
    background: #434343;
    letter-spacing: 1px;
    -webkit-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
    -moz-box-shadow: 0px 5px 40px -10px rgba(0, 0, 0, 0.57);
    box-shadow: 5px 40px -10px rgba(0, 0, 0, 0.57);
    transition: all 0.4s ease 0s;
  }
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>

<main>
  <!-- <section id="Banner" class="content-section">
    <div class="container content-wrap text-center">
      <h1>Hello from your friends at Casper.</h1>
      <h3>
        <em>Take a minute to give us a video review!</em>
      </h3>
      <div class="example_b" align="center" on:click={onYes}>
          Yes
      </div>

    </div>
    
  </section> -->

    <div class="dialogue">

      <div class="dialogue_text_box">
        <p>
          Hello from your friends at Casper.
          <br><br>
          Take a minute to give us a video review!
        </p>
      </div>
      
      <div class="button" align="center" on:click={onYes}>
          &check;&#8287;&#8287;&#8287;Yes
      </div>
    </div>

    

  <!-- Header Ends -->
</main>

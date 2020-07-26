<script>
  import { navigate } from "svelte-routing";

  // export let location;


</script>

<style>
  .dialogue{
    width: 100%;
    height: auto;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);  
  }

  .dialogue_text_box{
    background: rgb(50, 50, 50, 0.5);
    color: rgb(0, 0, 0, 0.65);
    padding: 30px;
    margin-bottom: 10px;
  }

  .dialogue_text_box>p{
    font-family: Arial, Helvetica, sans-serif;
    font-weight: 500;
    font-size: 28px;
  }

  main {
    background: #fff;
    width: 100vw;
    height: 100vh;

    text-align: center;
    padding: 1em;
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
    <div class="dialogue">

      <div class="dialogue_text_box">
        <p>
          All done! Thankyou for your response.
        </p>
      </div>
    </div>
</main>

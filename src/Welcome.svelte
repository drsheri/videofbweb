<script>
  import { navigate } from "svelte-routing";

  export let location;


  function onYes() {
      navigate("/video");
  }

</script>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }
  .example_b {
    color: #fff !important;
    text-transform: uppercase;
    text-decoration: none;
    background: #70ff73;
    padding: 10px;
	width: 100px;
    border-radius: 5px;
    display: inline-block;
    border: none;
    transition: all 0.4s ease 0s;
  }
  .example_b:hover {
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
  <section id="Banner" class="content-section">
    <div class="container content-wrap text-center">
      <h1>Hello from Casper</h1>
      <h3>
        <em>Take a minute to give us a video review!</em>
      </h3>
      <div class="example_b" align="center" on:click={onYes}>
          Yes
      </div>

    </div>
  </section>
  <!-- Header Ends -->
</main>

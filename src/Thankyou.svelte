<script>
  import { navigate } from "svelte-routing";

  export let location;

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
  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }
</style>

<main>
  <section id="Banner" class="content-section">
    <div class="container content-wrap text-center">
      <h1>Thank You!</h1>
    </div>
  </section>
  <!-- Header Ends -->
</main>

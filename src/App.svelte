<script>
  import { Router, Link, Route } from "svelte-routing";
  import Welcome from "./Welcome.svelte";
  import Video from "./Video.svelte";
  import Thankyou from "./Thankyou.svelte";

  export let url = "";
</script>

<Router url="{url}">
  <div>
    <Route path="/" component="{Welcome}" />
    <Route path="video"><Video /></Route>
    <Route path="thankyou"><Thankyou /></Route>
  </div>
</Router>
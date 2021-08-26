<script>
  import { Link } from 'svelte-routing';
  import { onMount } from 'svelte';

  let userInfo = undefined;
  const provider = 'aad';
  const redirect = window.location.pathname;

  onMount(async () => (userInfo = await getUserInfo()));

  async function getUserInfo() {
    try {
      const response = await fetch('/.auth/me');
      const payload = await response.json();
      const { clientPrincipal } = payload;
      return clientPrincipal;
    } catch (error) {
      console.error('No profile!');
      return undefined;
    }
  }

  function getProps({ href, isPartiallyCurrent, isCurrent }) {
    const isActive = href === '/' ? isCurrent : isPartiallyCurrent || isCurrent;

    // The object returned here is spread on the anchor element's attributes
    if (isActive) {
      return { class: 'router-link-active' };
    }
    return {};
  }
</script>

<div class="column is-2">
  <nav class="menu">
    <p class="menu-label">Menu</p>
    <ul class="menu-list">
      <Link to="/products" {getProps}>Products</Link>
      <Link to="/about" {getProps}>About</Link>
    </ul>
  </nav>

  <!-- Login and logout buttons -->
  <nav class="menu auth">
    <p class="menu-label">Auth</p>
    <div class="menu-list auth">
      {#if !userInfo}
        <a
          href={`/.auth/login/${provider}?post_login_redirect_uri=${redirect}`}
        >
          {provider}
        </a>
      {/if}
      {#if userInfo}
        <div class="user">
          <p>Welcome</p>
          <p>{userInfo && userInfo.userDetails}</p>
          {#each userInfo.userRoles as role}
            <p>Role: {role}</p>
          {/each}
          <p>{userInfo && userInfo.identityProvider}</p>
        </div>
        <a href={`/.auth/logout?post_logout_redirect_uri=${redirect}`}>
          Logout
        </a>
      {/if}
    </div>
  </nav>
  <!-- end of login and logout buttons -->
</div>

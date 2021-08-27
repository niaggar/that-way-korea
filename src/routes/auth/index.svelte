<script context="module">
	export async function preload(page, session) {
		let { user } = session
		if (user)
		    return this.redirect(302, '/')
	}
</script>


<svelte:head>
	<title>That way</title>
</svelte:head>

<div class="home-container">
    <img src="/img/map-icon.png" alt="Just a simple map">
    <h1>Welcome to That Way</h1>
    <a class="button important" href="/auth/login">Login into your account</a>
    <a class="button" href="/auth/signin">Create a new account</a>
</div>


<style>
    .home-container {
        width: 100%;
        min-height: 100vh;
        background-color: var(--dark);
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        padding: 0 1rem;
    }

    .home-container * {
        text-align: center;
    }

    h1 {
        color: #ffffff;
        margin-top: 2rem;
    }

    a {
        width: 100%;
        text-align: center;
        color: var(--blue-light);
        opacity: 0.7;
        font-weight: 700;
        margin-top: 2rem;
    }

    img {
        width: 40%;
        height: 40%;
        border-radius: 100%;
    }
</style>

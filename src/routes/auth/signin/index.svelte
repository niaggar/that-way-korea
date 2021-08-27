<script context="module">
	export async function preload(page, session) {
		let { user } = session
		if (user)
		    return this.redirect(302, '/')
	}
</script>


<div class="container">
    <a class="button important" href="auth/signin/passenger">Passenger account</a>
    <a class="button" href="auth/signin/driver">Driver account</a>
    <a class="button" href="auth">Back</a>
</div>


<style>
    .container * {
        display: block;
        width: 100%;
    }

    .container {
        padding: 0 1rem;
        width: 100%;
        min-height: 100vh;
        background-color: var(--dark);
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }

    a {
        width: 100%;
        text-align: center;
        color: var(--blue-light);
        opacity: 0.7;
        font-weight: 700;
        margin-top: 2rem;
    }
</style>